# JIT Compilation — wildx

## Overview

`wildx` allocates executable memory pages for runtime code generation (JIT):

```aria
wildx uint8->:code_page = wildx_alloc(4096);   // 4KB executable
defer {
    wildx_free(code_page);
}

// Write machine code bytes into code_page
// Cast to function pointer and call
```

**Note:** `wildx` is specified in the language but has limited test coverage.

## Safety

`wildx` is a Layer 3 (raw) operation — it bypasses all safety guarantees.
Executable memory is:
- Not bounds-checked
- Not type-checked
- A security risk if used with untrusted input

## Use Cases

- JIT compilers
- Dynamic code generation
- Runtime optimization
- Plugin systems with native code

## Related

- [memory_model/wild.md](../memory_model/wild.md) — unmanaged memory
- [types/pointer.md](../types/pointer.md) — pointer operations
