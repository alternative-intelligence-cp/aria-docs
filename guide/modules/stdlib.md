# Standard Library Overview

## Import Pattern

```aria
use "stdlib_file.aria".*;    // bare filename — compiler searches stdlib/
```

## Available Modules

The standard library lives in `REPOS/aria/stdlib/` and provides:

### String Utilities
- `string_length(s)` — character count (UTF-8 aware)
- `string_concat(a, b)` — concatenation
- `string_contains(s, substr)` — substring search

### I/O
- `print(s)` — output without newline
- `println(s)` — output with newline
- `readFile(path)` — read file contents
- `writeFile(path, data)` — write file contents

### Math
- Standard math functions via extern libm

### Threading
- Thread pool, channels, atomics, mutexes (via stdlib wrappers)

## Related

- [modules/use_import.md](../modules/use_import.md) — import syntax
- [io_system/print.md](../io_system/print.md) — print/println details
