# SMT-Guided Optimizations

## How Proofs Become Performance

When `--smt-opt` is passed, the compiler uses Z3 proofs to eliminate runtime safety
checks. Every eliminated check means fewer branches, less code, and faster execution.

```bash
ariac program.aria -o program --smt-opt
```

`--smt-opt` implies `--verify` — it enables verification AND uses the results for
optimization.

## Elimination Phases

The compiler runs these optimization phases in order:

| Phase | What It Eliminates | When Safe |
|-------|-------------------|-----------|
| 4.0 | User stack type checks | All `apush()` types identical |
| 4.25 | Dead branches | Branch condition provably true/false |
| 4.5 | Bounds checks | Index provably in `[0, N)` |
| 4.75 | Overflow checks | Arithmetic provably in type range |
| 4.85 | Division-by-zero checks | Divisor provably non-zero |
| 5.0 | Null checks | Optional provably non-nil |
| 5.25 | Loop invariants | Expression constant across iterations |
| 5.5 | Rules propagation | Constraints flow interprocedurally |
| 5.75 | Defaults fallback | Function provably infallible |

## Overflow Check Elimination

The most impactful optimization. With Rules-constrained operands, the solver proves
arithmetic stays in bounds:

```aria
Rules<int32>:Small = { $ >= 0, $ <= 1000 };

func:compute = int32() {
    limit<Small> int32:a = 500;
    limit<Small> int32:b = 300;
    int32:sum = a + b;     // Max: 1000+1000 = 2000 < INT32_MAX → check eliminated
    pass sum;
};
```

Without `--smt-opt`, every `+`/`-`/`*` emits a branch to check for overflow.
With `--smt-opt`, proven-safe operations emit plain arithmetic — typically **~16%
runtime improvement** for arithmetic-heavy code.

### Range Inference Cascade (v0.14.1+)

The range analyzer tracks ranges through chains of operations:

```aria
Rules<int32>:Input = { $ >= 1, $ <= 100 };

func:pipeline = int32() {
    limit<Input> int32:a = 10;
    limit<Input> int32:b = 20;
    int32:sum = a + b;             // Range: [2, 200]
    int32:doubled = sum * 2i32;    // Range: [4, 400]
    int32:final = doubled + 1i32;  // Range: [5, 401]
    pass final;                     // All 3 ops: overflow checks eliminated
};
```

## Division-by-Zero Check Elimination (v0.14.4)

When the divisor is provably non-zero (via Rules or range inference), the compiler
emits plain `sdiv`/`srem` instead of checked division:

```aria
Rules<int32>:NonZero = { $ >= 1, $ <= 1000 };

func:safe_div = int32() {
    limit<NonZero> int32:a = 100;
    limit<NonZero> int32:b = 7;
    int32:result = a / b;     // Divisor b ∈ [1,1000] → never zero → check eliminated
    int32:rem = a % b;        // Same for modulo
    pass result + rem;
};
```

## Null Check Elimination

For `??` (nil-coalesce) and `?.` (optional-chain) expressions where the optional
is provably non-nil:

```aria
// When solver proves the optional is always populated,
// the null check branch is removed entirely
```

## User Stack / User Hash Fast Path

When all `apush()` calls in a function use the same type, the solver proves type
homogeneity and the codegen emits `_fast` variants without type tag arguments:

- `aria_ustack_push_fast` / `pop_fast` — no runtime type check (~25% faster)
- `aria_uhash_set_fast` / `get_fast` — no runtime type check

## Result<T> Elision

Functions that can never fail (no `fail`, no `sys`, all callees infallible) return
raw `T` instead of `{T, ptr, i8}` Result wrapper. No Z3 needed — pure static analysis.

## --verify-level (v0.14.3+)

Control how thorough verification is. Higher levels prove more but take longer:

| Level | Phases Enabled | Use Case |
|-------|---------------|----------|
| 0 | None | Skip verification entirely |
| 1 | Rules, limits | Fast feedback during development |
| 2 | + contracts, overflow, ranges | Standard development |
| 3 | + null, bounds, div-zero, concurrency, all proofs | Release builds |

```bash
ariac program.aria --verify --verify-level=1    # ~9% overhead
ariac program.aria --verify --verify-level=3    # Full verification
```

## Incremental Solving (v0.14.3+)

The Z3 solver uses `push`/`pop` to maintain context between related queries in the
same function, avoiding solver creation/destruction overhead. This makes verification
of large files practical.

## Performance Impact

| Metric | Value |
|--------|-------|
| Compile overhead (`--verify-level=1`) | ~9% |
| Compile overhead (`--verify-level=2`) | ~15% |
| Compile overhead (`--verify-level=3`) | ~19% |
| Runtime speedup (overflow elimination) | ~16% |
| Runtime speedup (user stack fast path) | ~25% |
| Runtime speedup (Result elision) | ~3% |

## Writing Optimization-Friendly Code

1. **Constrain inputs with Rules** — the solver needs bounds to prove safety
2. **Use narrow ranges** — `[0, 1000]` is more provable than `[0, INT32_MAX]`
3. **Keep functions focused** — fewer branches = simpler proofs
4. **Constrain divisors explicitly** — `Rules:NonZero = { $ >= 1 }` enables div-zero elimination
5. **Build with `--smt-opt` for releases** — compile-time cost, runtime benefit

## Next

- [Concurrency Verification](05_concurrency_verification.md) — race detection
- [Troubleshooting](06_troubleshooting.md) — when verification doesn't work
