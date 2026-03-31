# Memory Model Overview

## Three Allocation Modes

Aria provides explicit control over memory allocation:

| Keyword | Mode | Management | Use Case |
|---------|------|------------|----------|
| `stack` | Stack | Automatic (scope-based) | Default, fast, most variables |
| `gc` | Garbage Collected | Automatic (GC runtime) | Shared data, complex lifetimes |
| `wild` | Unmanaged | Manual | FFI, OS-level, performance-critical |

## Stack Allocation (Default)

```aria
int32:x = 42;           // stack-allocated (default)
stack int32:y = 99;      // explicit stack keyword
```

Stack variables are freed when their scope exits. This is the fastest allocation mode.

**Note:** `stack` and `gc` are **contextual keywords** — they can be used as variable
names outside allocation contexts.

## GC Allocation

```aria
string:data = "hello";     // no wild/stack prefix = GC-managed
```

GC-allocated values are tracked by the garbage collector and freed when no longer
referenced. This is implicit — absence of `wild` or `stack` means GC mode.

## Wild (Unmanaged)

```aria
wild int32:raw_val = 42;
wild int8->:buf = alloc(1024);
```

No automatic cleanup. Combine with `defer` for manual resource management.

## Choosing a Mode

- **Default to stack** — fast, safe, no overhead
- **Use GC** when data must outlive its creating scope or be shared
- **Use wild** only for FFI, OS interaction, or performance-critical paths
- **Never use wild without defer** unless you have a specific reason

## Related

- [stack.md](stack.md) — stack allocation details
- [gc.md](gc.md) — garbage collector
- [wild.md](wild.md) — unmanaged memory
- [handle.md](handle.md) — Handle<T> safe arena pointers
- [borrow.md](borrow.md) — borrow semantics
