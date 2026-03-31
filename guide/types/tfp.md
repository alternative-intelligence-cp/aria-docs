# Twisted Floating Point — `tfp`

**Widths:** `tfp32`, `tfp64`

## Overview

Twisted floating point types combine IEEE-like floating-point representation with
TBB-style sticky error propagation. They are to floats what TBB is to integers.

| Type | Bytes | Notes |
|------|-------|-------|
| tfp32 | 4 | Single-precision with ERR sentinel |
| tfp64 | 8 | Double-precision with ERR sentinel |

## Status

`tfp32` and `tfp64` compile with `{exponent, mantissa}` initializer syntax:

```aria
tfp32:pi_approx = {1, 14159};
tfp64:e_approx = {2, 71828};
```

Basic initialization works. Full arithmetic operations are in progress.

## Related

- [flt.md](flt.md) — standard floating point
- [tbb.md](tbb.md) — twisted balanced binary (integer ERR propagation)
