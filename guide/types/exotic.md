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

**⚠️ These are specified but minimally implemented.** They exist in the type system
for research and exotic computing use cases (balanced ternary computing, nonary
architectures).

## Related

- [tbb.md](tbb.md) — twisted balanced binary (production error-propagating type)
- [int.md](int.md) — standard binary integers
