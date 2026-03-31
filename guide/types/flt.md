# Floating Point — `flt`

**Widths:** `flt32`, `flt64`, `flt128`, `flt256`, `flt512`

## Overview

IEEE 754 floating-point types for real number arithmetic. `flt64` is recommended for
general use. For deterministic fixed-point arithmetic, see [fix256.md](fix256.md).

## Width Table

| Type | Bytes | Precision | Range (approx) | Alias | Notes |
|------|-------|-----------|-----------------|-------|-------|
| flt32 | 4 | ~7 decimal digits | ±3.4×10^38 | f32 | |
| flt64 | 8 | ~15 decimal digits | ±1.8×10^308 | f64 | Recommended |
| flt128 | 16 | ~33 decimal digits | ±1.2×10^4932 | f128 | LBIM |
| flt256 | 32 | ~70 decimal digits | Extended | f256 | LBIM |
| flt512 | 64 | ~150 decimal digits | Extended | f512 | LBIM |

## Declaration

```aria
flt64:pi = 3.14159265358979;
flt32:temp = 98.6f32;          // type suffix
flt64:big = 1.23e10;           // scientific notation
```

## Special Values

- `NaN` — Not a Number (result of `0.0 / 0.0`)
- `+∞` / `-∞` — overflow results
- `+0` / `-0` — signed zero (IEEE 754)

`NaN != NaN` is **true** (standard IEEE behavior). Always compare with epsilon:

```aria
flt64:a = 0.1 + 0.2;
flt64:epsilon = 0.000001;
// DON'T: if (a == 0.3) { ... }
// DO:    if (abs(a - 0.3) < epsilon) { ... }
```

## Conversions

```aria
flt64:from_int = x;            // int→float: safe widening
int32:from_flt = f => i32;     // float→int: truncates (no rounding)
flt64:from_f32 = small;        // flt32→flt64: safe widening
```

## ABI Note

**`flt32` passes as `double` at the C ABI level.** If writing C shims for extern functions,
the C side must declare parameters as `double`, then cast internally. This is an Aria
codegen convention, not a bug.

## When NOT to Use Floats

- **Money**: Use cents as `int64`, or `fix256`
- **Deterministic simulation**: Use `fix256` (bit-exact across platforms)
- **Error-propagating math**: Use `tbb` types

## Related

- [fix256.md](fix256.md) — deterministic fixed-point
- [tfp.md](tfp.md) — twisted floating point
- [int.md](int.md) — integer types
