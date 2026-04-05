# Verification in Aria — Formal Proofs with Z3

> **Expanded guide available:** See `guide/verification/` for the full 6-section
> verification guide covering Rules, contracts, optimizations, concurrency, and
> troubleshooting in depth.

## Overview

Aria integrates the Z3 SMT solver to verify program properties at compile time.
Verification is opt-in via `--verify` flags and covers six domains: Rules/limit
constraints, function contracts, integer overflow, concurrency safety, memory safety,
and user-driven proofs.

When a property is **Proven**, the compiler can eliminate redundant runtime checks.
When **Disproven**, a compile-time diagnostic is emitted with a counterexample.
When **Unknown** (solver timeout), the runtime check is preserved.

## Compiler Flags

| Flag | Purpose |
|------|---------|
| `--verify` | Enable all verification phases |
| `--verify-contracts` | Verify `requires`/`ensures` contracts |
| `--verify-overflow` | Prove arithmetic won't overflow, eliminate checks |
| `--verify-concurrency` | Detect data races and deadlocks |
| `--verify-memory` | Detect use-after-free bugs |
| `--smt-opt` | Enable SMT-guided optimizations (implies `--verify`) |
| `--smt-timeout=N` | Per-query Z3 timeout in milliseconds (default: 5000) |
| `--verify-level=N` | Verification depth: 0=none, 1=fast, 2=standard, 3=thorough (v0.14.3+) |
| `--prove-report` | Print Proven/Disproven/Unknown for every check (implies `--verify`) |

Use `--verify` alone to enable everything, or combine individual flags:

```bash
ariac program.aria -o program --verify-contracts --verify-overflow --prove-report
```

## 1. Rules & Limit Verification

Rules define value constraints. The compiler uses Z3 to verify that every `limit<>`
binding satisfies its Rules at compile time.

```aria
Rules<int32>:r_positive = { $ > 0 };
Rules<int32>:r_bounded  = { $ >= 0, $ <= 1000 };

func:main = int32() {
    limit<r_positive> int32:x = 42;      // Proven: 42 > 0
    limit<r_bounded>  int32:y = 500;     // Proven: 500 >= 0 AND 500 <= 1000
    limit<r_positive> int32:z = -1;      // Disproven: -1 > 0 is false
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

Compile with `--verify --prove-report`:

```
[limit] Proven:  'x' satisfies r_positive
[limit] Proven:  'y' satisfies r_bounded
[limit] Disproven: 'z' violates r_positive — counterexample: $ = -1
```

### Rules Propagation

When **all** callers of a function pass `limit<>`-constrained arguments, the compiler
propagates those constraints to the callee and eliminates redundant checks:

```aria
Rules<int32>:r_safe = { $ >= 0, $ <= 100 };

func:process = int32(int32:val) {
    pass val * 2;     // overflow check eliminated if all callers use limit<r_safe>
}

func:main = int32() {
    limit<r_safe> int32:a = 50;
    int32:result = raw process(a);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

### Rules Narrowing

If `r_child` constraints are a subset of `r_parent`, the compiler proves subsumption:

```aria
Rules<int32>:r_positive       = { $ > 0 };
Rules<int32>:r_small_positive = { limit<r_positive>, $ < 100 };

// Passing r_small_positive where r_positive is expected: Proven safe
```

## 2. Function Contracts

`requires` (preconditions) and `ensures` (postconditions) are verified across call
sites using Z3.

```aria
func:safe_divide = flt64(int32:a, int32:b)
    requires b != 0
{
    pass (a / b);
};

func:clamp = int32(int32:val, int32:lo, int32:hi)
    requires lo <= hi
    ensures result >= lo
{
    if (val < lo) { pass lo; };
    if (val > hi) { pass hi; };
    pass val;
};
```

### Cross-Function Contract Propagation

When function A calls function B, the compiler checks that A's context satisfies B's
`requires` clause:

```aria
func:make_positive = int32(int32:x)
    requires x > 0
    ensures result > 0
{ pass x; };

func:double_positive = int32(int32:y)
    requires y > 0
{
    int32:pos = raw make_positive(y);  // Proven: y > 0 satisfies requires x > 0
    pass pos * 2;
};
```

### Loop Invariants

```aria
func:sum_to_n = int32(int32:n) requires n >= 0 {
    int32:s = 0;
    int32:i = 0;
    while (i < n) invariant s >= 0, i >= 0 {
        s = s + i;
        i = i + 1;
    };
    pass s;
};
```

## 3. Overflow Elimination

With `--verify-overflow` or `--smt-opt`, the compiler uses Rules constraints to prove
arithmetic is within bounds and eliminates overflow checks:

```aria
Rules<int32>:SmallPos = { $ >= 0, $ <= 1000 };

func:safe_add = int32() {
    limit<SmallPos> int32:a = 500;
    limit<SmallPos> int32:b = 300;
    int32:result = a + b;    // Proven safe: max 1000+1000 = 2000 < INT32_MAX
    pass result;             // overflow check ELIMINATED
};

func:risky_mul = int32() {
    limit<SmallPos> int32:m = 999;
    limit<SmallPos> int32:n = 999;
    int32:big = m * n;       // Unknown: 999*999 = 998001 fits, but solver can't prove all cases
    pass big;                // overflow check KEPT
};
```

## 4. Concurrency Verification

With `--verify-concurrency`, the compiler detects data races on shared variables and
deadlocks from inconsistent lock ordering.

### Data Race Detection

```aria
int32:counter = 0i32;
extern func:aria_shim_mutex_lock = int32(int64:mtx);
extern func:aria_shim_mutex_unlock = int32(int64:mtx);

// UNSAFE — shared write without lock
func:unsafe_writer = int64(int64:arg) {
    counter = counter + 1i32;          // Disproven: data race
    pass 0i64;
};

// SAFE — mutex-guarded write
func:safe_writer = int64(int64:arg) {
    drop aria_shim_mutex_lock(global_mtx);
    counter = counter + 1i32;          // Proven: race-free
    drop aria_shim_mutex_unlock(global_mtx);
    pass 0i64;
};
```

### Deadlock Detection

```aria
// UNSAFE — cyclic lock ordering
func:lock_ab = NIL() { lock(mtx_a); lock(mtx_b); unlock(mtx_b); unlock(mtx_a); };
func:lock_ba = NIL() { lock(mtx_b); lock(mtx_a); unlock(mtx_a); unlock(mtx_b); };
// Disproven: potential deadlock — A→B then B→A

// SAFE — consistent ordering
func:lock_ab2 = NIL() { lock(mtx_a); lock(mtx_b); unlock(mtx_b); unlock(mtx_a); };
func:lock_ab3 = NIL() { lock(mtx_a); lock(mtx_b); unlock(mtx_b); unlock(mtx_a); };
// Proven: deadlock-free
```

## 5. Memory Safety

With `--verify-memory`, the compiler tracks allocation/free state of `wild` pointers
and detects use-after-free bugs.

```aria
// UNSAFE — use after free
func:test_uaf = NIL() {
    wild ?->:buf = malloc(64i64);
    free(buf);
    wild ?->:alias = buf;     // Disproven: buf is freed
};

// SAFE — alias before free
func:test_safe = NIL() {
    wild ?->:buf = malloc(64i64);
    wild ?->:alias = buf;     // Proven: buf is live
    free(buf);
};

// CONDITIONAL — path-dependent UAF
func:test_cond_uaf = NIL(int32:flag) {
    wild ?->:buf = malloc(64i64);
    if (flag > 0i32) { free(buf); };
    wild ?->:use = buf;       // Disproven: buf might be freed on flag > 0 path
};
```

## 6. prove and assert_static

User-driven compile-time proof directives:

```aria
Rules<int32>:Percentage = { $ >= 0, $ <= 100 };

func:example = NIL() {
    limit<Percentage> int32:pct = 50;

    prove(pct >= 0);           // Proven at compile time → erased from binary
    prove(pct <= 100);         // Proven at compile time → erased from binary

    assert_static(pct > 10);   // Proven → erased
    assert_static(pct > 99);   // Unknown → warning, kept as runtime assert
};
```

| Directive | On Proven | On Disproven | On Unknown |
|-----------|-----------|--------------|------------|
| `prove(expr)` | Erased | Compile error | Compile error |
| `assert_static(expr)` | Erased | Warning + runtime assert | Warning + runtime assert |

## SMT-Guided Optimizations (--smt-opt)

When `--smt-opt` is passed, the compiler uses Z3 proofs to perform additional
optimizations beyond safety checking:

- **Overflow check elimination** — proven-safe arithmetic skips runtime overflow guards
- **Division-by-zero check elimination** — proven non-zero divisor skips zero check (v0.14.4)
- **Null check elimination** — `Optional` from non-null sources skips nil checks
- **Loop-invariant hoisting** — expressions proven constant across iterations
- **Defaults fallback elimination** — infallible functions skip `?|` fallback code
- **Rules propagation** — inter-procedural constraint propagation
- **Range inference cascade** — intermediate ranges propagate through assignments (v0.14.1)

### Performance Impact

Typical overhead and speedup from the benchmark suite:

| Metric | Value |
|--------|-------|
| Compile-time overhead (per flag) | 9–19% |
| Compile-time overhead (--verify-memory) | ~114% (small files) |
| Runtime speedup (--smt-opt, Rules) | ~3% |
| Runtime speedup (--smt-opt, overflow) | ~16% |
| Runtime speedup (--smt-opt, user stack) | ~25% |

## Timeout Tuning

The default per-query timeout is 5000ms. Reduce for faster compilation at the cost
of more Unknown results; increase for complex proofs:

```bash
ariac program.aria -o program --verify --smt-timeout=2000    # faster, more unknowns
ariac program.aria -o program --verify --smt-timeout=10000   # slower, fewer unknowns
```

## Notes

- Verification requires the compiler built with `-DARIA_HAS_Z3=ON`
- The Z3 context is created once per compilation and shared across all queries
- Each verification query creates a lightweight solver from the shared context
- `--prove-report` prints a summary of all Proven/Disproven/Unknown verdicts
- Rules, contracts, and `prove`/`assert_static` compose — use them together
- Disproven results include counterexamples when available

## Best Practices

1. **Start with contracts** — Add `requires`/`ensures` to public-facing functions first.
   They document intent and catch bugs at call sites.

2. **Use `limit<Rules>` at boundaries** — Apply Rules constraints where data enters
   the system (user input, file reads, network data). The solver propagates them inward.

3. **Prefer `prove` for critical invariants** — If a property MUST hold, use `prove()`.
   A compile error on failure is better than a runtime surprise.

4. **Use `assert_static` for desirable invariants** — When a property should hold but
   isn't critical, `assert_static` gives a warning and keeps a runtime fallback.

5. **Build with `--smt-opt` in release** — Let proven-safe checks be eliminated.
   The ~3–25% runtime speedup is free after paying the compile-time cost.

6. **Tune timeouts per project** — Small projects: `--smt-timeout=2000`. Large projects
   with complex proofs: `--smt-timeout=10000` or higher.

7. **Use `--prove-report` during development** — See exactly what's Proven vs Unknown.
   Target Unknown results for optimization or stronger annotations.

8. **Compose constraints** — Rules, contracts, and prove/assert_static work together.
   A `limit<r>` binding inside a function with `requires` gives the solver more to work with.

## Related

- [rules.md](rules.md) — Rules & Limit syntax
- [../functions/design_by_contract.md](../functions/design_by_contract.md) — requires/ensures
- [safety_layers.md](safety_layers.md) — Aria's safety model
- [concurrency.md](concurrency.md) — threading and mutexes
- [../verification/01_why_verification.md](../verification/01_why_verification.md) — full verification guide
- [../../reference/SMT_INFO.md](../../reference/SMT_INFO.md) — internal SMT architecture reference
