# Signed Integers — `int`

**Widths:** `int1`, `int2`, `int4`, `int8`, `int16`, `int32`, `int64`, `int128`, `int256`, `int512`, `int1024`, `int2048`, `int4096`

## Overview

Aria provides two's complement signed integers in powers of 2 from 1-bit to 4096-bit.
`int32` (alias `i32`) is the **default integer type** — bare literals like `100` infer as `int32`.

## Width Table

| Type | Bytes | Range | Alias | Notes |
|------|-------|-------|-------|-------|
| int1 | 1 | -1 to 0 | i1 | Sign bit only |
| int2 | 1 | -2 to 1 | i2 | |
| int4 | 1 | -8 to 7 | i4 | |
| int8 | 1 | -128 to 127 | i8 | `abs(-128)` is undefined |
| int16 | 2 | -32768 to 32767 | i16 | |
| int32 | 4 | -2^31 to 2^31-1 | i32 | **Default**. Wraps on overflow |
| int64 | 8 | -2^63 to 2^63-1 | i64 | |
| int128 | 16 | -2^127 to 2^127-1 | i128 | LBIM — see below |
| int256 | 32 | -2^255 to 2^255-1 | i256 | LBIM |
| int512 | 64 | -2^511 to 2^511-1 | i512 | LBIM |
| int1024 | 128 | -2^1023 to 2^1023-1 | i1024 | LBIM |
| int2048 | 256 | -2^2047 to 2^2047-1 | i2048 | LBIM |
| int4096 | 512 | -2^4095 to 2^4095-1 | i4096 | LBIM |

## Declaration

```aria
int32:x = 42;           // explicit type
x := 100;               // inferred as int32
int64:big = 9999999i64;  // suffix selects width
int128:huge = 99999999999999999999i128;  // suffix required for large literals
```

## Arithmetic

```aria
int32:a = 10;
int32:b = 3;
int32:sum  = (a + b);    // 13
int32:quot = (a / b);    // 3 (truncates toward zero)
int32:rem  = (a % b);    // 1
a++;                      // 11
b--;                      // 2
```

Integer division truncates: `1000000 / 2000000 = 0`.
Overflow wraps (two's complement).

## Conversions

```aria
int64:wide = a;            // widening: always safe
int32:narrow = big => i32; // narrowing: truncates high bits
int32:from_flt = f => i32; // float→int: truncates (no rounding)
```

Widening from any intN to intM (M > N) is always safe. Narrowing truncates.

## LBIM Types (≥128 bit)

Widths `int128` through `int4096` use the **Limb-Based Integral Model**:
- Stored as arrays of 64-bit limbs
- **Slower than int64** — use only when necessary
- Passed via `sret`/`byval` at the ABI level (not in registers)
- Type suffix required for literals: `42i128`, `99i1024`
- Use cases: cryptography, post-quantum key material, UUID storage, high-precision financial

## ABI Notes

| Width | ABI Passing |
|-------|-------------|
| int1–int64 | Register (zero/sign-extended) |
| int128+ | Pointer (sret/byval) |

## Related

- [uint.md](uint.md) — unsigned integers
- [tbb.md](tbb.md) — twisted balanced binary (error-propagating integers)
- [fix256.md](fix256.md) — fixed-point using LBIM
