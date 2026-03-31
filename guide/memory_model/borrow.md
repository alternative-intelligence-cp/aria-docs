# Borrow Semantics

## Overview

Aria has borrow operators for safe reference passing:

| Operator | Meaning |
|----------|---------|
| `$$i` | Immutable borrow — read-only reference |
| `$$m` | Mutable borrow — read-write reference |

## Usage

```aria
$$i:ref = value;        // immutable borrow
$$m:mut_ref = value;    // mutable borrow
```

## Pin

The `#` operator pins a value, preventing it from being moved:

```aria
#value;                 // pin — cannot be moved after this
```

## Related

- [overview.md](overview.md) — memory model overview
- [types/pointer.md](../types/pointer.md) — pointer types
