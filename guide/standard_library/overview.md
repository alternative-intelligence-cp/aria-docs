# Standard Library Overview

## What's in stdlib/

The standard library provides common functionality as importable Aria modules.
Located at `REPOS/aria/stdlib/`.

## Import Pattern

```aria
use "module_name.aria".*;    // bare filename — compiler searches stdlib/
```

## Available Categories

| Category | Modules | See |
|----------|---------|-----|
| String | string_utils | [string_utils.md](string_utils.md) |
| Math | math functions via libm | [math.md](math.md) |
| I/O | print, readFile, writeFile | [io_system/](../io_system/) |
| Threading | thread pool, channels | [advanced_features/concurrency.md](../advanced_features/concurrency.md) |

## Built-in (No Import Needed)

These are available without `use`:
- `print()`, `println()`
- `astack`, `apush`, `apop`, `apeek`, `acap`, `asize`, `afits`, `atype`
- `ahash`, `ahset`, `ahget`, `ahcount`, `ahsize`, `ahfits`, `ahtype`
- `sys()`, `sys!!()`, `sys!!!()`

## Related

- [modules/use_import.md](../modules/use_import.md) — import syntax
- [modules/packages.md](../modules/packages.md) — third-party packages
