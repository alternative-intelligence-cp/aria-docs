# Function Contracts

## requires / ensures

Contracts specify function preconditions and postconditions. The solver verifies
them across call sites.

```aria
func:safe_divide = flt64(int32:a, int32:b)
    requires b != 0
{
    pass (a / b);
};
```

- `requires` — what callers must guarantee (precondition)
- `ensures` — what the function guarantees to return (postcondition)

## Contract Verification (v0.14.0+)

The compiler performs two levels of contract verification:

### Phase 1: Satisfiability
Checks that `requires` and `ensures` are individually satisfiable (not self-contradictory).

### Phase 2: Proof (v0.14.0+)
Proves that the function body, given the `requires` clause, actually establishes the
`ensures` clause. This is the key advance in v0.14.x — the compiler can now prove
correctness, not just check consistency.

```aria
func:clamp = int32(int32:val, int32:lo, int32:hi)
    requires lo <= hi
    ensures result >= lo
{
    if (val < lo) { pass lo; };   // Proven: lo >= lo
    if (val > hi) { pass hi; };   // Proven: hi >= lo (given lo <= hi)
    pass val;                      // Proven: val >= lo (since val >= lo at this point)
};
```

The solver symbolically executes all code paths and verifies that every `pass`
statement satisfies the `ensures` clause under the `requires` assumptions.

## Cross-Function Propagation

When function A calls function B, the solver verifies that A's context satisfies
B's `requires`:

```aria
func:make_positive = int32(int32:x)
    requires x > 0
    ensures result > 0
{ pass x; };

func:double_positive = int32(int32:y)
    requires y > 0
{
    int32:pos = raw make_positive(y);   // Proven: y > 0 satisfies requires x > 0
    pass pos * 2;
};
```

## Ensures Propagation

When a called function has `ensures`, the solver uses that guarantee in the
calling context:

```aria
func:abs_val = int32(int32:x)
    ensures result >= 0
{
    if (x < 0) { pass (0i32 - x); };
    pass x;
};

func:use_abs = int32(int32:a) {
    int32:positive = raw abs_val(a);
    // Solver knows: positive >= 0 (from ensures)
    prove(positive >= 0);              // Proven at compile time
    pass positive;
};
```

## prove and assert_static

User-driven compile-time proof directives:

```aria
Rules<int32>:Percentage = { $ >= 0, $ <= 100 };

func:example = NIL() {
    limit<Percentage> int32:pct = 50;

    prove(pct >= 0);           // Proven → erased from binary
    prove(pct <= 100);         // Proven → erased from binary

    assert_static(pct > 10);   // Proven → erased
    assert_static(pct > 99);   // Unknown → warning, kept as runtime assert
};
```

| Directive | On Proven | On Disproven | On Unknown |
|-----------|-----------|--------------|------------|
| `prove(expr)` | Erased | Compile error | Compile error |
| `assert_static(expr)` | Erased | Warning + runtime assert | Warning + runtime assert |

Use `prove()` for invariants that **must** hold. Use `assert_static()` for properties
that **should** hold but you want a runtime fallback.

## Design-by-Contract Patterns

### Guard at the boundary
```aria
Rules<int32>:ValidAge = { $ >= 0, $ <= 150 };

func:create_user = int32(int32:age)
    requires age >= 0, age <= 150
{
    // All code here can assume age is valid
    pass age;
};
```

### Layer contracts
```aria
func:low_level = int32(int32:x)
    requires x > 0
    ensures result > 0
{ pass x * 2; };

func:mid_level = int32(int32:y)
    requires y > 0
    ensures result > 0
{
    pass raw low_level(y);    // requires satisfied, ensures propagates
};
```

### Pair with Rules
```aria
Rules<int32>:NonNeg = { $ >= 0 };

func:sqrt_approx = int32(int32:n)
    requires n >= 0
{
    // limit<NonNeg> on callers → requires automatically satisfied
    pass n;
};
```

## Next

- [Optimizations](04_optimizations.md) — how proofs become faster code
- [Concurrency Verification](05_concurrency_verification.md) — race detection
