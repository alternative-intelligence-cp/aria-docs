# use — Importing Modules

## Syntax

```aria
use "path/to/module.npk".*;           // wildcard import (all pub symbols)
use "stdlib_file.npk".*;              // stdlib import (bare filename)
```

## Path Resolution

- **Relative paths**: resolved relative to the importing source file
- **Bare filenames**: compiler searches the stdlib directory
- Wildcard `.*` imports all `pub` declarations

## Example

```aria
// file: math_utils.npk
pub func:square = int32(int32:x) {
    pass (x * x);
};

// file: main.npk
use "math_utils.npk".*;

func:main = int32() {
    int32:val = raw square(5);
    println(`&{val}`);    // 25
    exit 0;
};

func:failsafe = int32(tbb32:err) { exit 1; };
```

## Notes

- Only `pub func:` declarations are importable
- `Rules<T>` are file-scoped — not importable
- Negative constants imported via `use` may be zeroed at runtime (known codegen issue)
  — compute inline as `0i64 - value` instead

## File Extension Conventions

Nitpick source files use two accepted extensions:

| Extension | Status | When to use |
|-----------|--------|-------------|
| `.npk` | **Canonical** | All new Nitpick code |
| `.aria` | Alias (backward-compatible) | Legacy code or ports from the pre-rename era |

Both extensions are treated identically by the compiler and module resolver
(as of v0.22.2, which fixed `.aria` acceptance — POLISH-002). There is no
runtime or semantic difference.

**Convention:** prefer `.npk` for new projects. The `.aria` extension is
accepted to ease migration of older code and ports.

**`use` path:** include the extension explicitly in the import path. The
module resolver accepts both:

```nitpick
use "math_utils.npk".*;   // canonical
use "math_utils.aria".*;  // also accepted
```

## Related

- [mod.md](mod.md) — defining modules
- [packages.md](packages.md) — package structure
