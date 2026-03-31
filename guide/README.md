# Aria Programming Guide

**Version:** 0.4.7  
**Last Updated:** 2026-03-31

## Sections

| Section | Description |
|---------|-------------|
| [types/](types/) | All Aria types — integers, floats, TBB, strings, structs, etc. |
| [operators/](operators/) | Arithmetic, comparison, logical, bitwise, result, and special operators |
| [functions/](functions/) | Function declaration, main/failsafe, pass/fail, extern, generics |
| [control_flow/](control_flow/) | if/else, for, while, loop/till, when, pick |
| [memory_model/](memory_model/) | Stack, GC, wild allocation modes, Handle<T>, borrow semantics |
| [modules/](modules/) | use/mod/pub/extern, package structure |
| [collections/](collections/) | User stack (astack) and hash tables (ahash) |
| [io_system/](io_system/) | Print, file I/O, stdin, sys() syscalls, streams |
| [standard_library/](standard_library/) | Standard library overview and builtins |
| [advanced_features/](advanced_features/) | Generics, traits, design by contract, closures, concurrency, JIT |
| [debugging/](debugging/) | Debugging tools and strategies |

## Building HTML

```bash
./md2html.py --all    # Convert all .md files to html/
./md2man.py           # Generate man pages
```

## Source of Truth

- **Language inventory:** `specs/list.txt`
- **Authoritative spec:** `specs/aria_specs.txt`
- **Archived old guide:** `guide_archive/` (372 files, pre-v0.4.7)
