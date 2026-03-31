# Wild — Unmanaged Memory

## Declaration

```aria
wild int32:raw = 42;
wild int8->:buffer = alloc(1024);
wild int64->:ptr = alloc<int64>();
```

Wild memory is **not tracked** by stack cleanup or GC. You are responsible for cleanup.

## Defer — Guaranteed Cleanup

Use `defer` to ensure cleanup runs when scope exits:

```aria
wild int8->:buffer = alloc(1024);
defer {
    free(buffer);
}

// ... use buffer ...
// free(buffer) runs automatically at scope exit
```

`defer` executes in LIFO (last-in, first-out) order for multiple defers.

## Executable Memory — wildx

`wildx` allocates memory with execute permission for JIT compilation:

```aria
wildx uint8->:code = wildx_alloc(4096);   // 4KB executable page
defer {
    wildx_free(code);
}

// Write machine code into 'code' buffer
// Execute via function pointer cast
```

**Note:** `wildx` is specified but has no passing tests yet.

## When to Use Wild

- FFI buffers passed to C functions
- OS-level operations (mmap, syscalls)
- JIT compilation (`wildx`)
- Performance-critical paths where GC pauses are unacceptable

## Related

- [overview.md](overview.md) — all allocation modes
- [handle.md](handle.md) — safe alternative to raw pointers
- [types/pointer.md](../types/pointer.md) — pointer syntax
