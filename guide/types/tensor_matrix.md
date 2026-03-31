# Tensor & Matrix Types

## tensor

Multi-dimensional array type for numerical computing and AI workloads.

## matrix

2D matrix type. Semantically a rank-2 tensor with matrix-specific operations.

## Status

**⚠️ Not yet implemented.** The types are registered in the parser and can be declared,
but have no constructors or operations. Also registered as `tmatrix` and `ttensor`.

```aria
matrix:mat;   // compiles (declaration only)
tensor:tens;  // compiles (declaration only)
```

## Related

- [vec.md](vec.md) — fixed-size vector types
- [simd.md](simd.md) — SIMD vector types
