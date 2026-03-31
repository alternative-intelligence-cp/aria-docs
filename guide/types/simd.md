# SIMD Types — `simd<T,N>`

## Overview

Generic SIMD (Single Instruction, Multiple Data) vector type where T is the element type
and N is the lane count (must be a power of 2).

```aria
simd<flt32, 4>:v = simd_new(1.0f32, 2.0f32, 3.0f32, 4.0f32);
simd<int32, 8>:vi = simd_splat(42);  // all lanes = 42
```

## Lane Counts

N must be a power of 2: 2, 4, 8, 16, 32, etc.
Hardware support varies by platform:
- x86-64 SSE: 4×f32, 2×f64
- x86-64 AVX2: 8×f32, 4×f64
- x86-64 AVX-512: 16×f32, 8×f64

## Operations

Component-wise arithmetic: `+`, `-`, `*`, `/` apply to all lanes simultaneously.

## Related

- [vec.md](vec.md) — fixed-size vector types
- [tensor_matrix.md](tensor_matrix.md) — tensor and matrix types
