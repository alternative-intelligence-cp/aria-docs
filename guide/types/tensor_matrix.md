# Tensor & Matrix Types

## tensor

Multi-dimensional array type for numerical computing and AI workloads.

```aria
tensor:t = tensor_new([3, 3]);  // 3×3 tensor
```

## matrix

2D matrix type. Semantically a rank-2 tensor with matrix-specific operations
(multiplication, transpose, determinant).

```aria
matrix:m = matrix_new(3, 3);
```

## Related

- [vec.md](vec.md) — fixed-size vector types
- [simd.md](simd.md) — SIMD vector types
