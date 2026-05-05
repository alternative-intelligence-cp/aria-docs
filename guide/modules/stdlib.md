# Standard Library Overview

The standard library contains **72 modules** in `REPOS/aria/stdlib/`.

## Import Pattern

```aria
use "stdlib_file.npk".*;    // bare filename — compiler searches stdlib/
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
- Thread pool, channels, atomics, mutexes, condvars, rwlocks, barriers, actors (via stdlib wrappers on aria-libc)

### Concurrency Modules (v0.11.0)
- `thread.npk`, `mutex.npk`, `condvar.npk`, `rwlock.npk` — via aria_libc_process
- `channel.npk`, `actor.npk`, `thread_pool.npk` — built on above
- `shm.npk` — POSIX shared memory via aria_libc_posix

## Related

- [modules/use_import.md](../modules/use_import.md) — import syntax
- [io_system/print.md](../io_system/print.md) — print/println details
