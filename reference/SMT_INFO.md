# SMT Solver Reference — Aria Compiler

Working document. Will become a best practices guide later.

---

## Architecture Overview

Z3 is statically linked into `ariac`. Users never interact with it directly.
The compiler uses it in two modes:

1. **Verification mode** (`--verify`) — prove constraints, contracts, overflow safety
2. **Optimization mode** (`--smt-opt`) — prove properties → emit faster codegen

### Pipeline Location

All Z3 work happens in **Phase 3.25** of compilation, between type checking
and borrow checking, before IR generation.

```
Parse → Type Check → Z3 Verification (Phase 3.25) → Borrow Check → Result Elision (Phase 3.75) → IR Gen → LLVM → Link
```

### Files

| File | LOC | Role |
|------|-----|------|
| `include/analysis/z3_verifier.h` | 208 | Class definition, verification API |
| `src/analysis/z3_verifier.cpp` | 1353 | Core verification logic, Z3 queries |
| `src/main.cpp` (Phase 3.25) | ~750 | Orchestration, AST walks, optimization discovery |
| `src/backend/ir/ir_generator.cpp` | ~20 | Fast-mode flag activation per function |
| `src/backend/ir/codegen_expr.cpp` | ~140 | Fast-path code emission for ustack/uhash |

---

## Compiler Flags

| Flag | Implies | Effect |
|------|---------|--------|
| `--verify` | — | Enable Z3 verification (Rules consistency, limit checks) |
| `--verify-contracts` | `--verify` | Verify requires/ensures function contracts |
| `--verify-overflow` | `--verify` | Verify integer arithmetic overflow safety |
| `--verify-report` | `--verify` | Emit detailed proof results (proven/disproven/unknown) |
| `--smt-opt` | `--verify` | Enable SMT-guided optimizations (fast paths) |

Defined in `CompilerOptions` struct at main.cpp:132-137.
Parsed at main.cpp:328-341.

---

## Verification Passes (current as of v0.4.7)

### 1. Rules Consistency Check
- Verifies each `Rules` declaration is not self-contradictory
- Creates Z3 bitvector/real variable, asserts all conditions, checks SAT
- If UNSAT → error: rules can never be satisfied

### 2. Literal Limit Verification
- Walks AST for `limit<RulesName>` on variable declarations
- For integer/float literal initializers, proves value satisfies all conditions
- Uses `verifyLimitInt()` / `verifyLimitFloat()` with concrete values

### 3. Function Contract Verification (`--verify-contracts`)
- Walks functions with `requires`/`ensures` clauses
- Currently Phase 1 only: checks requires and ensures are individually satisfiable
- **NOT YET IMPLEMENTED: Phase 2** — proving ensures from requires + body

### 4. Overflow Verification (`--verify-overflow`)
- Walks binary ops (+, -, *) looking for literal-literal arithmetic
- Uses Z3 bitvector overflow builtins (`Z3_mk_bvadd_no_overflow`, etc.)
- **Limitation:** Only checks when both operands are known constants

### 5. SMT Optimization Discovery (`--smt-opt`)
- **User Stack (astack):** Collects type tags from all `apush()` calls in a function.
  If all tags are identical, Z3 proves homogeneity → function added to `ustack_opt_funcs`.
  Codegen emits `aria_ustack_push_fast` / `pop_fast` (no runtime type check).
- **User Hash (ahash):** Same pattern for `ahset()` value tags → `uhash_opt_funcs`.
  Codegen emits `aria_uhash_set_fast` / `get_fast`.

---

## Z3 API Usage Patterns

### Core Pattern (Negation-Based Proof)
```
1. Create Z3 sort (bitvector for ints, real for floats)
2. Translate Aria AST condition → Z3_ast
3. Assert NEGATION of the property
4. solver.check()
5. L_FALSE (UNSAT) → property PROVEN for all inputs
   L_TRUE (SAT) → DISPROVEN, extract counterexample from model
   L_UNKNOWN → timeout or too complex
```

### Z3 Context
- Single `Z3_context` lives for entire compilation
- Per-query timeout: **5 seconds** (set in Z3Verifier constructor)
- Fresh solver created per query with push/pop scope

### Type Mapping
| Aria Type | Z3 Sort | Tag |
|-----------|---------|-----|
| int8 | BitVec(8) | 0 |
| int16 | BitVec(16) | 1 |
| int32 | BitVec(32) | 2 |
| int64/int | BitVec(64) | 3 |
| flt32 | Real | 4 |
| flt64/flt | Real | 5 |
| bool | — | 6 |
| string | — | 7 |

---

## Data Flow: Verifier → Codegen

```
main.cpp Phase 3.25:
  AST walk → collect push/set type tags per function
  Z3 proves homogeneity
  → std::set<std::string> ustack_opt_funcs
  → std::set<std::string> uhash_opt_funcs

main.cpp (post-verification):
  ir_gen.setUStackOptimizedFuncs(ustack_opt_funcs)
  ir_gen.setUHashOptimizedFuncs(uhash_opt_funcs)

ir_generator.cpp (FUNC_DECL entry):
  ustack_fast_mode = ustack_optimized_funcs.count(funcName) > 0
  uhash_fast_mode = uhash_optimized_funcs.count(funcName) > 0

codegen_expr.cpp (call emission):
  if (ustack_fast_mode) emit _fast variant (no type tag arg)
  else emit standard variant (with type tag arg)
```

---

## Known Gaps & Limitations (v0.4.7)

### Contract Verification
- Only Phase 1 (sanity check) — no body analysis
- Cannot prove ensures from requires + function body
- Needs symbolic execution or weakest precondition calculus

### Overflow Detection
- Only works with literal operands (both sides must be constants)
- No range inference through assignments or control flow
- Would need abstract interpretation or interval analysis

### Type Tag Inference
- Binary expressions: walks leftmost chain heuristically
- Complex expressions (function calls, casts) → tagged as unknown (-1)
- Unknown tags prevent optimization for entire function

### Cross-Function Analysis
- Each function analyzed in isolation
- No interprocedural flows (callee results don't propagate to callers)
- No module boundary crossing

### No Incremental Solving
- Fresh solver per query — no context reuse between queries
- Could cache Z3 context for related queries in same function

---

## Bugs, Workarounds, & Notes

(Log observations here as we work through v0.5.x)

### v0.5.0 Work Log

#### Result<T> Elision — IMPLEMENTED

**What:** Static analysis identifies functions that can never fail (no `fail`, no `sys`,
all callees also infallible). These return raw T instead of `{T, ptr, i8}`, eliminating
wrapping/unwrapping overhead at every call site.

**Activation:** `--smt-opt` flag (no Z3 needed — pure static analysis)

**Phase:** 3.75 (between Borrow Checker and IR Generation)

**Analysis algorithm:**
1. Collect all user functions with bodies (skip main, failsafe, generics, externs)
2. Walk each function's AST recursively, recording:
   - `has_fail`: contains any FAIL node
   - `has_sys_call`: calls sys/sys!!/sys!!!
   - `called_user_funcs`: set of user-function callees
3. Fixed-point iteration: a function is infallible iff `!has_fail && !has_sys_call &&
   all callees are infallible`. Propagate until stable.

**Files modified:**
- `ir_generator.h` — added `result_elide_funcs` set, `result_elide_mode` flag, `setResultElideFuncs()` setter
- `main.cpp` — Phase 3.75 analysis pass (~200 lines), passes set to IR gen
- `ir_generator.cpp` — Pre-pass forward decl checks elision, FUNC_DECL sets elide flag,
  Result wrapping decision uses elide flag, pass() returns raw T for elided funcs

**Key insight:** Call-site unwrap code needed NO modification — it detects Result by
structural shape `{T, ptr, i8}`. Elided functions return raw T, so the struct check
naturally short-circuits.

**Bug found/fixed:** Pre-pass forward declaration was creating functions with Result
return type before main codegen. Added elision check to pre-pass too.

**Also fixed:** Default return for non-struct types used `ConstantInt::get(type, 0)` which
only works for integers. Changed to `Constant::getNullValue(type)` which works for all types.

---

## Future: Best Practices Guide Topics

- When to use `--verify` vs `--smt-opt` vs `--verify-all`
- Writing solver-friendly code (how to make your code provable)  
- Understanding counterexamples
- Performance impact of verification (build time budgets)
- Dealing with UNKNOWN results
- Contract authoring patterns
- Rules design for verifiability
