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
vec2:pos = { x = 1.0, y = 2.0 };
vec3:color = { x = 0.5, y = 0.8, z = 1.0 };
```

## Special Constructors

```aria
vec2:zero  = vec2::ZERO;   // {0, 0}
vec2:one   = vec2::ONE;    // {1, 1}
vec2:right = vec2::RIGHT;  // {1, 0}
vec2:up    = vec2::UP;     // {0, 1}
```

## Operations

```aria
vec2:a = { x = 1.0, y = 2.0 };
vec2:b = { x = 3.0, y = 4.0 };

vec2:sum  = (a + b);       // component-wise add: {4, 6}
vec2:diff = (a - b);       // component-wise sub: {-2, -2}
vec2:prod = (a * b);       // component-wise mul (Hadamard): {3, 8}
```

**`*` between two vectors is component-wise (Hadamard product), NOT dot product.**

Use method calls for vector-specific operations:
- `dot(a, b)` — dot product (scalar result)
- `cross(a, b)` — cross product (vec3→vec3, vec2→scalar)

## Status

**⚠️ Limited compiler support.** Consider the `aria-vec` package for production use.

## Related

- [simd.md](simd.md) — generic SIMD types
- [tensor_matrix.md](tensor_matrix.md) — tensor and matrix types
