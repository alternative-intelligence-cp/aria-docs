# Exotic Number Types

## Balanced Ternary

| Type | Description | Notes |
|------|-------------|-------|
| `trit` | Ternary digit: -1, 0, +1 | ~2 bits packed |
| `tryte` | 9 trits | Ternary byte equivalent |

Trit values: `T` (−1), `0` (0), `1` (+1).

## Balanced Nonary

| Type | Description | Notes |
|------|-------------|-------|
| `nit` | Nonary digit: -4 to +4 | Base-9 unit |
| `nyte` | Collection of nits | Nonary byte equivalent |

## Status

Basic initialization and comparison operations compile and pass tests:

```aria
trit:a = 1;
trit:b = 0;
tryte:ta = 1;
nit:n1 = 3;
nyte:ny1 = 7;
```

These types are functional for basic operations. Advanced arithmetic is limited.

## Related

- [tbb.md](tbb.md) — twisted balanced binary (production error-propagating type)
- [int.md](int.md) — standard binary integers
