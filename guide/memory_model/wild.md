# Wild — Unmanaged Memory

## Declaration

```aria
wild int32:raw = 42;
wild uint8->:buffer = wild_alloc(4096);
```

Wild memory is **not tracked** by stack cleanup or GC. You are responsible for cleanup.

## Defer — Guaranteed Cleanup

Use `defer` to ensure cleanup runs when scope exits:

```aria
wild int32->:ptr = wild_alloc(sizeof(int32));
defer wild_free(ptr);    // guaranteed to run on scope exit

// ... use ptr ...
// wild_free(ptr) runs automatically here
```

`defer` executes in LIFO (last-in, first-out) order for multiple defers.

## Executable Memory — wildx

`wildx` allocates memory with execute permission for JIT compilation:

```aria
wildx uint8->:code = wildx_alloc(4096);   // 4KB executable page
defer wildx_free(code);

// Write machine code into 'code' buffer
// Execute via function pointer cast
```

## When to Use Wild

- FFI buffers passed to C functions
- OS-level operations (mmap, syscalls)
- JIT compilation (`wildx`)
- Performance-critical paths where GC pauses are unacceptable

## Related

- [overview.md](overview.md) — all allocation modes
- [handle.md](handle.md) — safe alternative to raw pointers
- [types/pointer.md](../types/pointer.md) — pointer syntax
