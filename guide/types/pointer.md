# Pointers — `wild`, `wildx`, `->`, `@`

## Overview

Aria has three memory modes. Pointers are only directly used in `wild` (unmanaged) mode
and `extern` blocks.

## Pointer Operators

| Operator | Meaning |
|----------|---------|
| `@` | Address-of (get pointer to variable) |
| `->` | In type: pointer to T. In expression: member access through pointer |
| `<-` | Dereference (get value from pointer) |
| `?.` | Safe navigation (NIL-safe member access) |

## Wild Pointers

```aria
wild int32:x = 42;          // unmanaged allocation
int32->:ptr = @x;           // pointer to x
int32:val = <-ptr;           // dereference
```

`wild` memory is **not garbage collected**. You must manage it manually or use `defer`
for cleanup.

## Executable Memory — `wildx`

`wildx` allocates executable memory for JIT compilation:

```aria
wildx uint8->:code = wildx_alloc(4096);  // 4KB executable page
```

## Safe Alternative: Handle<T>

For arena-allocated data, prefer `Handle<T>` over raw pointers. See
[memory_model/handle.md](../memory_model/handle.md).

## Related

- [memory_model/wild.md](../memory_model/wild.md) — wild allocation mode
- [memory_model/handle.md](../memory_model/handle.md) — generational handles
- [functions/extern.md](../functions/extern.md) — FFI pointer patterns
