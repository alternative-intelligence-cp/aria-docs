# use — Importing Modules

## Syntax

```aria
use "path/to/module.aria".*;           // wildcard import (all pub symbols)
use "stdlib_file.aria".*;              // stdlib import (bare filename)
```

## Path Resolution

- **Relative paths**: resolved relative to the importing source file
- **Bare filenames**: compiler searches the stdlib directory
- Wildcard `.*` imports all `pub` declarations

## Example

```aria
// file: math_utils.aria
pub func:square = int32(int32:x) {
    pass (x * x);
};

// file: main.aria
use "math_utils.aria".*;

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

## Related

- [mod.md](mod.md) — defining modules
- [packages.md](packages.md) — package structure
