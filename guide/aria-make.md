# nitpick-build — Nitpick Build System

`npkbld` is the build system for Nitpick projects. It handles incremental
compilation, dependency tracking, and FFI (C library) builds.

For full documentation, see the
[nitpick-build README](https://github.com/alternative-intelligence-cp/nitpick-build).

## Quick Reference

### Build a project

```bash
npkbld              # build default target
npkbld my-target    # build specific target
npkbld --force      # rebuild everything
npkbld --clean      # remove build artifacts
npkbld -v           # verbose output
```

### build.abc format

Every project needs a `build.abc` file:

```
[project]
name = "my-app"
version = "0.1.0"

[compiler]
path = "npkc"
flags = "-O2"

[target.my-app]
type = "binary"
entry = "src/main.npk"
sources = ["src/*.npk"]
```

### FFI builds (C interop)

```
[target.my_c_lib]
type = "c_library"
sources = ["src/ffi/*.c"]
flags = "-O2 -fPIC"

[target.my-app]
type = "binary"
entry = "src/main.npk"
sources = ["src/*.npk"]
link = ["my_c_lib"]
```

### Features

- Incremental builds via SHA-256 content hashing
- Parallel compilation
- Glob patterns for source files
- C library compilation and linking
- Dependency graph tracking
