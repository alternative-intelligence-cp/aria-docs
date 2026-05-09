# Advanced FFI Patterns

## Large Struct Passing (LBIM)

Types ≥128 bits are passed via `sret`/`byval` pointers, not registers:

```aria
// int128, int256, etc. are passed by pointer at ABI level
extern func:big_math = int128(int128:a, int128:b);
// C side receives: i128* sret, i128* byval, i128* byval
```

## String Return from Extern

C functions that return strings must return `AriaString`:

```c
// C side
typedef struct { char* data; int64_t length; } AriaString;

AriaString my_string_func(const char* input) {
    AriaString result;
    result.data = strdup(input);
    result.length = strlen(input);
    return result;
}
```

## Handle Pattern

Use `int64` instead of pointer types in extern blocks to avoid `{i1, ptr}` wrapper issues:

```aria
extern func:create_context = int64();        // returns handle as int64
extern func:destroy_context = NIL(int64:h);  // takes handle as int64
```

## Store Before Pass

When passing extern function results directly to `pass`, store in a variable first:

```aria
// WRONG: pass(extern_func());
// RIGHT:
int32:val = extern_func();
pass val;
```

## Stdio Buffering Mismatch (POLISH-013)

Nitpick's `print()` and `println()` call the `write()` syscall directly — they
are **unbuffered**. C's `printf`, `fputs`, and `puts` write to a **buffered**
stream (`FILE*` stdout). When mixing both in the same program (e.g., a C shim
alongside Nitpick I/O), the C output may be delayed or lost on program exit.

**Symptom:** C shim output appears out of order or missing when interleaved with
Nitpick `println()` calls.

**Fix:** Call `fflush(stdout)` at the end of any C function that writes to
`stdout`:

```c
// C shim
void my_shim_print(const char* msg) {
    puts(msg);
    fflush(stdout);  // flush C buffer to fd 1 before Nitpick read/write
}
```

Alternatively, use `write(1, buf, len)` in C shims for fully unbuffered output
matching Nitpick's model:

```c
#include <unistd.h>
void my_shim_print(const char* msg, int64_t len) {
    write(1, msg, (size_t)len);
}
```

## Related

- [functions/extern.md](../functions/extern.md) — basic extern usage
- [types/string.md](../types/string.md) — string ABI
- [types/int.md](../types/int.md) — LBIM types
