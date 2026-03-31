# Borrow Semantics

## Overview

Aria has borrow operators for safe reference passing:

| Operator | Meaning |
|----------|---------|
| `$$i` | Immutable borrow — read-only reference |
| `$$m` | Mutable borrow — read-write reference |

## Usage

```aria
$$i int32:ref = value;        // immutable borrow
$$m int32:mut_ref = value;    // mutable borrow
```

Multiple immutable borrows are allowed simultaneously. Only one mutable borrow at a time.
The compiler enforces the 1-mut XOR N-immut rule.

## Pin

The `#` operator pins a value, preventing it from being moved:

```aria
#value;                 // pin — cannot be moved after this
```

## Related

- [overview.md](overview.md) — memory model overview
- [types/pointer.md](../types/pointer.md) — pointer types
