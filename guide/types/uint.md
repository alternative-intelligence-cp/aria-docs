# Unsigned Integers — `uint`

**Widths:** `uint1`, `uint2`, `uint4`, `uint8`, `uint16`, `uint32`, `uint64`, `uint128`, `uint256`, `uint512`, `uint1024`, `uint2048`, `uint4096`

## Overview

Unsigned integers store non-negative values only. Required for bitwise operations
(`&`, `|`, `^`, `~`, `<<`, `>>`).

## Width Table

| Type | Bytes | Range | Alias | Notes |
|------|-------|-------|-------|-------|
| uint1 | 1 | 0 to 1 | u1 | Single bit |
| uint2 | 1 | 0 to 3 | u2 | |
| uint4 | 1 | 0 to 15 | u4 | |
| uint8 | 1 | 0 to 255 | u8 | Byte |
| uint16 | 2 | 0 to 65535 | u16 | |
| uint32 | 4 | 0 to 2^32-1 | u32 | |
| uint64 | 8 | 0 to 2^64-1 | u64 | |
| uint128 | 16 | 0 to 2^128-1 | u128 | LBIM |
| uint256 | 32 | 0 to 2^256-1 | u256 | LBIM |
| uint512 | 64 | 0 to 2^512-1 | u512 | LBIM |
| uint1024 | 128 | 0 to 2^1024-1 | u1024 | LBIM |
| uint2048 | 256 | 0 to 2^2048-1 | u2048 | LBIM |
| uint4096 | 512 | 0 to 2^4096-1 | u4096 | LBIM |

## Declaration

```aria
uint32:flags = 0xFF00u32;
uint64:addr = 0xDEADBEEFu64;
uint8:byte = 255u8;
```

## Bitwise Operations

Bitwise ops require unsigned types:

```aria
uint32:a = 0xFF00u32;
uint32:b = 0x0FF0u32;
uint32:and_result = (a & b);   // 0x0F00
uint32:or_result  = (a | b);   // 0xFFF0
uint32:xor_result = (a ^ b);   // 0xF0F0
uint32:not_result = (~a);      // 0xFFFF00FF
uint32:shl = (a << 4u32);     // 0xF0000
uint32:shr = (a >> 4u32);     // 0x0FF0
```

## LBIM Types (≥128 bit)

Same limb-based model as signed integers. See [int.md](int.md) for LBIM details.
Passed via sret/byval at ABI level.

## Related

- [int.md](int.md) — signed integers
- [tbb.md](tbb.md) — twisted balanced binary
