# Vector Types — `vec`

**Widths:** `vec2`, `vec3`, `vec9`

## Overview

Fixed-size vector types for 2D, 3D, and 9-component math. SIMD-optimized where possible.
Defaults to float components.

## Width Table

| Type | Components | Size (float) | Size (double) | Use Case |
|------|-----------|--------------|---------------|----------|
| vec2 | x, y | 8 bytes | 16 bytes | 2D graphics, UV coords |
| vec3 | x, y, z | 12 bytes | 24 bytes | 3D graphics, physics |
| vec9 | 9 elements | 36 bytes | 72 bytes | 3×3 matrices, tensors |

## Declaration

```aria
vec2:pos = vec2(1.0, 2.0);
vec3:color = vec3(0.5, 0.8, 1.0);
vec9:v = 0;                     // zero-init only (vec9 has no constructor yet)
```

Components are `flt64` by default. Access via `.x`, `.y`, `.z`:

```aria
flt64:px = pos.x;
flt64:py = pos.y;
```

## Operations

```aria
vec2:a = vec2(1.0, 2.0);
vec2:b = vec2(3.0, 4.0);

vec2:sum  = (a + b);       // component-wise add: {4, 6}
vec2:diff = (a - b);       // component-wise sub: {-2, -2}
vec2:prod = (a * b);       // component-wise mul (Hadamard): {3, 8}
```

**`*` between two vectors is component-wise (Hadamard product), NOT dot product.**

## Status

**Partial compiler support.** `vec2` and `vec3` constructors and arithmetic work.
`vec9` supports zero-init only — no constructor or indexing yet.

## Related

- [simd.md](simd.md) — generic SIMD types
- [tensor_matrix.md](tensor_matrix.md) — tensor and matrix types
