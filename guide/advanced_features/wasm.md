# WebAssembly Target

## Overview

Aria can target WebAssembly (WASM) via LLVM's wasm32 backend.

## Limitations

Current WASM limitations:
- No direct syscall access (`sys()` not available)
- No `wildx` (executable memory)
- No POSIX threads (single-threaded only)
- Limited file I/O (browser sandbox)
- No FFI with native C libraries

## Building for WASM

```bash
ariac --target wasm32 source.aria -o output.wasm
```

## Related

- [functions/extern.md](../functions/extern.md) — FFI (not available in WASM)
- [io_system/sys.md](../io_system/sys.md) — syscalls (not available in WASM)
