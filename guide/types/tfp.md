# Twisted Floating Point — `tfp`

**Widths:** `tfp16`, `tfp32`

## Overview

Twisted floating point types combine IEEE-like floating-point representation with
TBB-style sticky error propagation. They are to floats what TBB is to integers.

| Type | Bytes | Notes |
|------|-------|-------|
| tfp16 | 2 | Half-precision with ERR sentinel |
| tfp32 | 4 | Single-precision with ERR sentinel |

## Status

**⚠️ Specified but not yet implemented in the compiler.** These types are part of the
language specification and reserved for future implementation.

## Related

- [flt.md](flt.md) — standard floating point
- [tbb.md](tbb.md) — twisted balanced binary (integer ERR propagation)
