# Twisted Balanced Binary — `tbb`

**Widths:** `tbb8`, `tbb16`, `tbb32`, `tbb64`

## Overview

TBB types are Aria's **error-propagating integers**. They use a symmetric range with a
dedicated ERR sentinel value. Once a TBB value becomes ERR, it stays ERR through all
subsequent operations — "sticky error propagation." This is the foundation of Aria's
safety model.

Think of ERR as **NaN for integers**, but with a single well-defined value and deterministic
comparison behavior.

## Width Table

| Type | Bytes | Valid Range | ERR Sentinel | Notes |
|------|-------|-------------|--------------|-------|
| tbb8 | 1 | -127 to +127 | -128 (0x80) | `abs(-127) = +127` always safe |
| tbb16 | 2 | -32767 to +32767 | -32768 (0x8000) | |
| tbb32 | 4 | -2^31+1 to +2^31-1 | -2^31 (0x80000000) | |
| tbb64 | 8 | -2^63+1 to +2^63-1 | -2^63 (0x8000000000000000) | |

Note the **symmetric range** — unlike standard two's complement, the positive and negative
maximums are equal in magnitude. The minimum value is reserved as the ERR sentinel.

## Declaration

```aria
tbb32:value = 42;
tbb8:small = 100tbb8;     // optional type suffix
tbb32:error = ERR;         // ERR literal
```

Default initial value is `0`.

## Sticky Error Propagation

Any arithmetic involving ERR produces ERR:

```aria
tbb32:a = 10;
tbb32:b = ERR;
tbb32:c = (a + b);   // ERR — propagated from b
tbb32:d = (c * 100); // ERR — propagated from c
```

This lets you chain computations and check once at the end:

```aria
tbb32:result = step3(step2(step1(input)));
if (result == ERR) {
    // handle error — one check covers entire pipeline
}
```

## What Triggers ERR

- Overflow (result exceeds valid range)
- Underflow (result below valid range)
- Division by zero
- Explicit assignment: `val = ERR;`
- Any operation with an ERR operand (sticky propagation)

## ERR Comparison

Unlike IEEE NaN, ERR has well-defined comparison behavior:
- `ERR == ERR` is **true**
- ERR compares as **less than all valid values**

## TBB vs Standard Integers

| Feature | int32 | tbb32 |
|---------|-------|-------|
| Range | -2^31 to 2^31-1 | -2^31+1 to 2^31-1 |
| Overflow | Wraps | ERR |
| Div by zero | Undefined | ERR |
| Error propagation | None | Sticky |
| abs(min) | Undefined | Valid (+max) |
| Performance | Faster | Slight overhead |

Use `tbb` when error propagation matters. Use `int` when you need maximum performance
or the full range.

## Related

- [result.md](result.md) — Result<T> error handling (function-level)
- [int.md](int.md) — standard signed integers
- [frac.md](frac.md) — fraction types built on TBB
- [fix256.md](fix256.md) — fixed-point built on limbs
