# aria-make — Aria Build System

`aria_make` is the build system for Aria projects. It handles incremental
compilation, dependency tracking, and FFI (C library) builds.

For full documentation, see the
[aria-make README](https://github.com/alternative-intelligence-cp/aria-make).

## Quick Reference

### Build a project

```bash
aria_make              # build default target
aria_make my-target    # build specific target
aria_make --force      # rebuild everything
aria_make --clean      # remove build artifacts
aria_make -v           # verbose output
```

### build.abc format

Every project needs a `build.abc` file:

```
[project]
name = "my-app"
version = "0.1.0"

[compiler]
path = "ariac"
flags = "-O2"

[target.my-app]
type = "binary"
entry = "src/main.aria"
sources = ["src/*.aria"]
```

### FFI builds (C interop)

```
[target.my_c_lib]
type = "c_library"
sources = ["src/ffi/*.c"]
flags = "-O2 -fPIC"

[target.my-app]
type = "binary"
entry = "src/main.aria"
sources = ["src/*.aria"]
link = ["my_c_lib"]
```

### Features

- Incremental builds via SHA-256 content hashing
- Parallel compilation
- Glob patterns for source files
- C library compilation and linking
- Dependency graph tracking
