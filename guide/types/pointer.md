# Pointers — `wild`, `wildx`, `->`, `<-`, `.`, `@`

## Overview

Aria has three memory modes. Pointers are only directly used in `wild` (unmanaged) mode
and `extern` blocks.

## Pointer Operators

| Operator | Meaning |
|----------|---------|
| `@` | Address-of (get pointer to variable) |
| `.` | Direct member access on struct values |
| `->` | In type: pointer to T. In expression: dereference pointer then access field |
| `<-` | Dereference (get entire value from pointer) |
| `?.` | Safe navigation (NIL-safe member access) |

## Member Access: `.` vs `->`

`.` accesses a field on a **struct value** directly. It does not auto-dereference pointers.

`->` dereferences a **pointer to a struct**, then accesses the field. Equivalent to
`(<-ptr).field`.

`<-` dereferences a pointer and returns the **entire value**.

```aria
struct:Point = { int32:x; int32:y; };

// Direct struct — use .
Point:p = Point { x: 10, y: 20 };
int32:px = p.x;                     // direct member access

// Pointer to struct — use -> for fields, <- for whole value
Point->:ptr = @p;
int32:px2 = ptr->x;                 // dereference + field access
Point:copy = <-ptr;                 // dereference entire struct

// Pointer indexing then . access
// [0] on a pointer dereferences at offset 0, yielding a struct value
// then . accesses the field on that value
HashMap->:map = get_map();
int64:len = map[0].length;          // index deref + direct field access
```

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
