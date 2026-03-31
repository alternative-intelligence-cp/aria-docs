# Fractions — `frac`

**Widths:** `frac8`, `frac16`, `frac32`, `frac64`

## Overview

Fraction types store exact rational numbers as mixed fractions: whole part + numerator +
denominator. Built on TBB internally, inheriting sticky ERR propagation.

## Width Table

| Type | Component Type | Storage | Notes |
|------|---------------|---------|-------|
| frac8 | tbb8 | 3 bytes | `{tbb8:whole, tbb8:num, tbb8:denom}` |
| frac16 | tbb16 | 6 bytes | |
| frac32 | tbb32 | 12 bytes | |
| frac64 | tbb64 | 24 bytes | |

## Representation

A frac value `{whole, num, denom}` represents `whole + num/denom`:

- `{1, 1, 3}` = 1⅓ (exactly)
- `{0, 1, 3}` = ⅓ (exactly — not 0.333...)
- `{0, 0, 1}` = 0

## Canonical Form Invariants

After every operation, fractions auto-normalize to canonical form:
- Denominator > 0
- Numerator ≥ 0 when whole ≠ 0
- Numerator < denominator
- Reduced to lowest terms (GCD applied)

## Why Fractions?

Floating point cannot represent ⅓ exactly. Fractions can:

```
flt64:  1/3 = 0.333333333333... (truncated, drift accumulates)
frac32: 1/3 = {0, 1, 3}        (exact, zero drift)
```

## Status

**⚠️ Specified but not yet implemented in the compiler.** The type system design and
LLVM IR layout are defined. Implementation is planned.

## Related

- [tbb.md](tbb.md) — underlying component type
- [fix256.md](fix256.md) — deterministic fixed-point alternative
- [flt.md](flt.md) — IEEE floating point
