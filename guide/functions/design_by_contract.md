# Design by Contract

## Overview

Aria supports formal contracts on functions: `requires` (preconditions), `ensures`
(postconditions), and `invariant` (loop invariants). These are checked at compile time
where possible and at runtime otherwise.

## Requires — Preconditions

```aria
func:divide = flt64(int32:a, int32:b)
    requires b != 0
{
    pass (a / b);
}
```

Multiple conditions are comma-separated:

```aria
func:clamp_to_byte = int32(int32:val)
    requires val >= 0, val <= 255
{
    pass val;
}
```

## Ensures — Postconditions

```aria
func:check_order = int32(int32:lo, int32:hi)
    requires lo >= 0, hi > lo
    ensures hi > 0
{
    pass hi - lo;
}
```

Contract clauses go between the closing `)` of the signature and the opening `{` of
the body. Compiled with `--verify-contracts` flag (uses Z3 SMT solver).

## Invariant

Loop invariants are specified in the language design but have limited compiler test
coverage.

## Related

- [advanced_features/rules.md](../advanced_features/rules.md) — refinement types with `limit<>`
- [functions/declaration.md](declaration.md) — function syntax
