# Troubleshooting Verification

## Understanding Unknown Results

When the solver returns **Unknown**, it means it couldn't prove or disprove the
property within the timeout. The runtime check is preserved — your program is still
safe, just not optimized.

Common causes of Unknown results and how to address them.

## Cause 1: Missing Constraints

The solver can only reason about what it knows. If a variable has no Rules and no
contracts, the solver assumes it can be any value in its type range.

**Problem:**
```aria
func:maybe_overflow = int32(int32:a, int32:b) {
    int32:sum = a + b;    // Unknown: a and b could be anything
    pass sum;
};
```

**Fix — add Rules:**
```aria
Rules<int32>:Bounded = { $ >= 0, $ <= 1000 };

func:safe_add = int32() {
    limit<Bounded> int32:a = 500;
    limit<Bounded> int32:b = 300;
    int32:sum = a + b;    // Proven: max 2000 < INT32_MAX
    pass sum;
};
```

## Cause 2: Complex Expressions

The solver works best with simple arithmetic and comparisons. Complex expressions
(nested function calls, pointer arithmetic, string operations) may exceed its
reasoning ability.

**Problem:**
```aria
func:complex = int32(int32:x) {
    int32:a = raw helper1(x);
    int32:b = raw helper2(a);
    int32:c = raw helper3(b);
    prove(c > 0);     // Unknown: too many layers of indirection
};
```

**Fix — add intermediate contracts:**
```aria
func:helper1 = int32(int32:x) requires x > 0 ensures result > 0 { ... };
func:helper2 = int32(int32:x) requires x > 0 ensures result > 0 { ... };
func:helper3 = int32(int32:x) requires x > 0 ensures result > 0 { ... };

func:chained = int32(int32:x) requires x > 0 {
    int32:a = raw helper1(x);       // ensures a > 0
    int32:b = raw helper2(a);       // ensures b > 0
    int32:c = raw helper3(b);       // ensures c > 0
    prove(c > 0);                    // Proven via contract chain
};
```

## Cause 3: Solver Timeout

For very large functions or very complex proofs, the solver may run out of time.

**Check remaining budget:**
```bash
ariac program.aria --verify --prove-report --smt-timeout=10000   # 10s per query
```

**Fix — split large functions:**
```aria
// Instead of one giant function with 50 operations,
// split into focused helpers with contracts
```

## Cause 4: Non-Linear Arithmetic

Z3's bitvector theory handles linear arithmetic well but struggles with:
- Multiplication of two variables (`a * b` where neither is constant)
- Division with variable divisors
- Modular arithmetic chains

**Workaround:** Constrain operands with narrow Rules to help the solver.

## Using --prove-report

The `--prove-report` flag shows every verification verdict:

```bash
ariac program.aria --verify --prove-report
```

Output:
```
[limit]    Proven:  'x' satisfies Positive
[contract] Proven:  requires clause of safe_divide() satisfied
[overflow] Proven:  addition at line 12 cannot overflow
[overflow] Unknown: multiplication at line 15 — timeout
[race]     Disproven: 'counter' in unsafe_writer() — unprotected write
Z3: 14 proven, 1 disproven, 1 unknown
```

## Using --verify-level for Debugging

Start with a low level to isolate which phase produces Unknown:

```bash
ariac program.aria --verify --verify-level=1   # Rules/limits only
ariac program.aria --verify --verify-level=2   # + contracts, overflow
ariac program.aria --verify --verify-level=3   # Everything
```

If level 1 works but level 2 produces Unknown, the issue is in contract or overflow
verification — focus your annotations there.

## Interpreting Counterexamples

When a property is Disproven, the solver provides a concrete counterexample:

```
[contract] Disproven: requires clause of process() NOT satisfied
           Counterexample: val = -1
```

This tells you exactly which input violates the contract. Fix the caller to not pass
that value, or weaken the contract.

## Common Gotchas

| Symptom | Cause | Fix |
|---------|-------|-----|
| All proofs Unknown | `--verify` not passed | Add `--verify` or `--smt-opt` |
| Overflow Unknown | No Rules on operands | Add `limit<>` constraints |
| Contract Disproven | Caller doesn't satisfy `requires` | Add constraints to caller |
| Race false positive | Custom sync not recognized | Use `aria_shim_mutex_*` |
| Slow compilation | Too many proofs at level 3 | Use `--verify-level=2` for dev |

## Performance Budget

Rule of thumb for compilation time:
- **No verification:** baseline
- **--verify-level=1:** +9% (Rules/limits only)
- **--verify-level=2:** +15% (standard)
- **--verify-level=3:** +19% (thorough)
- **--smt-timeout=2000:** faster, more Unknowns
- **--smt-timeout=10000:** slower, fewer Unknowns

For most projects, `--verify-level=2` during development and `--verify-level=3 --smt-opt`
for release builds is the sweet spot.

## Getting Help

- `--prove-report` — see all verdicts
- `--verbose` — see SMT phase timing and per-function progress
- Check if the property is actually provable: can you convince *yourself* it holds?
  If you can't explain why, the solver probably can't either.
