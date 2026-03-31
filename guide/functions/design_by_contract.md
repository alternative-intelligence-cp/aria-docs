# Design by Contract

## Overview

Aria supports formal contracts on functions: `requires` (preconditions), `ensures`
(postconditions), and `invariant` (loop invariants). These are checked at compile time
where possible and at runtime otherwise.

## Requires — Preconditions

```aria
func:divide = flt64(int32:a, int32:b)
    requires (b != 0)
{
    pass (a / b);
}
```

## Ensures — Postconditions

The special `result` variable is bound to the return value in `ensures` clauses:

```aria
func:abs = int32(int32:x)
    ensures (result >= 0)
{
    if (x < 0) {
        pass (0 - x);
    }
    pass x;
}
```

## Invariant — Loop Invariants

```aria
int32:sum = 0;
loop(0, 10, 1) {
    invariant (sum >= 0);
    sum = (sum + $);
}
```

## Related

- [advanced_features/rules.md](../advanced_features/rules.md) — refinement types with `limit<>`
- [functions/declaration.md](declaration.md) — function syntax
