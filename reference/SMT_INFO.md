# SMT Solver Reference — Aria Compiler

Internal architecture reference. Updated for v0.14.5.
User-facing guide: `guide/verification/` (6 sections) and `guide/advanced_features/verification.md`.

---

## Architecture Overview

Z3 is statically linked into `ariac`. Users never interact with it directly.
The compiler uses it in two modes:

1. **Verification mode** (`--verify`) — prove constraints, contracts, overflow safety, data races
2. **Optimization mode** (`--smt-opt`) — prove properties → emit faster codegen

### Pipeline Location

Z3 work spans multiple phases between type checking and IR generation:

```
Parse → Type Check → Z3 Verification (Phase 3.25) → Dead Branch Elim (4.25)
  → Bounds Check Elim (4.5) → Overflow Elim (4.75) → Div-Zero Elim (4.85)
  → Null Check Elim (5.0) → Loop Hoist (5.25) → Rules Propagation (5.5)
  → Defaults Elim (5.75) → User Assertions (Phase 6) → Borrow Check
  → Result Elision (Phase 3.75) → IR Gen → LLVM → Link
```

### Files

| File | Role |
|------|------|
| `include/analysis/z3_verifier.h` | Class definition, verification API, persistent solver |
| `src/analysis/z3_verifier.cpp` | Core verification: contracts, ranges, races, proofs |
| `src/main.cpp` (Phases 3.25–6) | Orchestration, AST walks, optimization discovery |
| `include/backend/ir/ir_generator.h` | Safe-set members, fast-mode flags |
| `src/backend/ir/ir_generator.cpp` | Fast-path code emission for all elimination types |

---

## Compiler Flags

| Flag | Implies | Effect |
|------|---------|--------|
| `--verify` | — | Enable Z3 verification (Rules consistency, limit checks) |
| `--verify-contracts` | `--verify` | Verify requires/ensures function contracts |
| `--verify-overflow` | `--verify` | Verify integer arithmetic overflow safety |
| `--verify-concurrency` | `--verify` | Detect data races and deadlocks (v0.14.2) |
| `--verify-memory` | `--verify` | Detect use-after-free bugs |
| `--verify-report` | `--verify` | Emit detailed proof results (proven/disproven/unknown) |
| `--verify-level=N` | `--verify` | Control verification depth: 0=none, 1=fast, 2=standard, 3=thorough (v0.14.3) |
| `--smt-opt` | `--verify` | Enable SMT-guided optimizations (fast paths) |
| `--smt-timeout=N` | — | Per-query Z3 timeout in milliseconds (default: 5000) |
| `--prove-report` | `--verify` | Print Proven/Disproven/Unknown for every check |
| `--emit-llvm` | — | Emit LLVM IR (useful for verifying eliminations) |

### --verify-level Detail (v0.14.3)

| Level | Phases Enabled |
|-------|---------------|
| 0 | None (skip verification) |
| 1 | Rules consistency, literal limits |
| 2 | + contracts, overflow, range inference |
| 3 | + null, bounds, div-zero, concurrency, all proofs |

---

## Verification Passes (current as of v0.14.5)

### Phase 3.25: Core Z3 Verification
- Rules consistency (SAT check per Rules declaration)
- Literal limit verification (concrete values vs constraints)
- Function contract verification Phase 1 (satisfiability)
- Function contract verification Phase 2 (body proves ensures from requires) — v0.14.0

### Phase 4.0: User Stack/Hash Optimization
- Type tag homogeneity for `apush()`/`ahset()` → `_fast` variants

### Phase 4.25: Dead Branch Elimination
- Branch conditions provably true/false → remove dead code

### Phase 4.5: Bounds Check Elimination
- Array index provably in [0, N) → skip runtime bounds check
- Uses `bounds_safe_set` (external `std::set<ASTNode*>`)

### Phase 4.75: Overflow Check Elimination
- Arithmetic provably in type range → skip overflow guard
- Uses `overflow_safe_set` (external `std::set<ASTNode*>`)
- Range inference (v0.14.1) makes this much more powerful — ranges propagate through assignments

### Phase 4.85: Division-by-Zero Check Elimination (v0.14.4)
- Divisor provably non-zero → emit plain `sdiv`/`srem`
- Covers `/`, `%`, `/=`, `%=` operators
- Uses `div_safe_set` (external `std::set<ASTNode*>`)
- Reuses `proveNonNullFromRules()` — proving divisor ≠ 0

### Phase 5.0: Null Check Elimination
- Optional provably non-nil → skip null check for `??`/`?.`
- Uses `null_check_safe_set` (external `std::set<ASTNode*>`)

### Phase 5.25: Loop Invariant Hoisting
- Expressions proven constant across loop iterations → hoist

### Phase 5.5: Rules Propagation
- Interprocedural constraint flow — callee inherits caller's limits

### Phase 5.75: Defaults Fallback Elimination
- Infallible functions → skip `?|` fallback code

### Phase 6: User Assertion Verification
- `prove()` and `assert_static()` directives
- Per-function progress reporting with timing (v0.14.3)

### Phase 3.75: Result<T> Elision (post-borrow-check)
- Functions that can never fail → return raw T instead of `{T, ptr, i8}`
- Pure static analysis, no Z3 needed

### Data Race Analysis (v0.14.2)
- Shared variable detection (globals in thread entry functions)
- Lock region recognition: mutex, rwlock (read/write), atomic
- Channel-based ownership transfer detection
- Deadlock detection via lock ordering analysis

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

### Z3 Context & Solver
- Single `Z3_context` lives for entire compilation
- Per-query timeout: **5 seconds** (configurable via `--smt-timeout=N`)
- **Persistent solver** with push/pop scope (v0.14.3) — avoids create/destroy per query
- `Z3_solver_push()` before each query, `Z3_solver_pop()` after — reuses learned lemmas

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

The compiler uses external `std::set<ASTNode*>` keyed by raw pointer identity to
pass proof results from SMT phases to codegen. No flags on AST nodes.

### Sets Declared in main.cpp

| Set Name | Populated By | Codegen Effect |
|----------|-------------|----------------|
| `bounds_safe_set` | Phase 4.5 | Skip array bounds check |
| `overflow_safe_set` | Phase 4.75 | Skip overflow guard |
| `div_safe_set` | Phase 4.85 | Skip division-by-zero check |
| `null_check_safe_set` | Phase 5.0 | Skip null/nil check |
| `limit_check_safe_set` | Phase 5.5 | Skip redundant limit check |
| `defaults_safe_set` | Phase 5.75 | Skip defaults fallback |
| `ustack_opt_funcs` | Phase 4.0 | Emit `_fast` ustack calls |
| `uhash_opt_funcs` | Phase 4.0 | Emit `_fast` uhash calls |

### Flow

```
main.cpp Phase 3.25–6:
  AST walk → collect proof targets
  Z3 proves properties → insert into safe sets

main.cpp (post-verification):
  ir_gen.setBoundsCheckSafe(bounds_safe_set)
  ir_gen.setOverflowCheckSafe(overflow_safe_set)
  ir_gen.setDivCheckSafe(div_safe_set)         // v0.14.4
  ir_gen.setNullCheckSafe(null_check_safe_set)
  ...

ir_generator.cpp (codegen):
  if (overflow_check_safe.count(expr)) → emit plain add/sub/mul
  if (div_check_safe.count(expr))      → emit plain sdiv/srem
  else → emit safe variant with runtime check
```

### Range Inference (v0.14.1)

The RangeAnalyzer performs flow-sensitive range inference per function:
1. Collects all Rules constraints into VarRulesMap
2. Walks assignments to infer intermediate ranges
3. Merges inferred ranges back into VarRulesMap via `mergeInferred()`
4. Z3 proofs use merged ranges → more eliminations without explicit annotations

---

## Known Gaps & Limitations (v0.14.5)

### Contract Verification
- Phase 2 body analysis covers all pass/return paths
- Multi-return (conditional returns) fully supported
- Limitation: recursive functions not yet unrolled

### Overflow Detection
- Range inference through assignments and control flow (v0.14.1)
- Cascade: intermediate ranges propagate through variable chains
- Limitation: multiplication of two variables with wide ranges may timeout

### Type Tag Inference
- Binary expressions: walks leftmost chain heuristically
- Complex expressions (function calls, casts) → tagged as unknown (-1)
- Unknown tags prevent optimization for entire function

### Cross-Function Analysis
- Ensures propagation: callee's postconditions available in caller (v0.14.0)
- Rules propagation: interprocedural constraint flow (v0.14.1)
- Limitation: no module boundary crossing

### Data Race Analysis (v0.14.2)
- Recognizes: mutex lock/unlock, rwlock rd/wr, atomics, channels
- Limitation: custom synchronization primitives not recognized
- Limitation: indirect locking through wrapper functions not inlined

### Solver Performance (v0.14.3)
- Persistent solver with push/pop eliminates per-query overhead
- --verify-level controls which phases run
- Large files (100+ functions) benefit significantly from incremental solving

---

## v0.14.x Changelog

### v0.14.0 — Contract Proof Completion
- Phase 2 contract verification: proves `ensures` from `requires` + function body
- Symbolic execution of all code paths per function
- Multi-return support (conditional paths)
- Cross-function `ensures` propagation

### v0.14.1 — Range Inference & Flow-Sensitive Analysis
- RangeAnalyzer: flow-sensitive range inference through assignments
- Range cascade: intermediate variables inherit and propagate ranges
- `mergeInferred()`: folds inferred ranges into VarRulesMap for Z3 proofs
- Significantly more overflow eliminations without explicit annotations

### v0.14.2 — Data Race Analysis
- Shared variable detection for thread entry functions
- Lock region recognition: mutex, rwlock (read/write)
- Atomic operation exemption
- Channel-based ownership transfer detection
- Deadlock detection via lock ordering graph

### v0.14.3 — Performance & Incremental Solving
- Persistent Z3 solver with `push`/`pop` (no create/destroy per query)
- `--verify-level=N` (0=none, 1=fast, 2=standard, 3=thorough)
- Level-gating on all phases
- Per-function progress reporting with timing in Phase 6

### v0.14.4 — Extended SMT Optimization Fast-Paths
- Division-by-zero check elimination (Phase 4.85)
- `div_safe_set` + IRGenerator `div_check_safe`
- 4 codegen fast-paths: `/`, `%`, `/=`, `%=`
- Reuses `proveNonNullFromRules()` for divisor ≠ 0 proof

### v0.14.5 — Documentation & Final Audit
- 6-section verification guide (`guide/verification/`)
- SMT_INFO.md updated for all v0.14.x changes
- Final regression, fuzzer check, audit

---

## Future: Best Practices Guide Topics

Covered in `guide/verification/`:
- [01_why_verification.md](../guide/verification/01_why_verification.md) — motivation & comparison to testing
- [02_rules_and_limits.md](../guide/verification/02_rules_and_limits.md) — Rules, limits, range inference
- [03_contracts.md](../guide/verification/03_contracts.md) — requires/ensures, prove, assert_static
- [04_optimizations.md](../guide/verification/04_optimizations.md) — --smt-opt, elimination phases, performance
- [05_concurrency_verification.md](../guide/verification/05_concurrency_verification.md) — races, locks, channels
- [06_troubleshooting.md](../guide/verification/06_troubleshooting.md) — Unknown results, timeouts, workarounds
