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

## Related

- [functions/extern.md](../functions/extern.md) — basic extern usage
- [types/string.md](../types/string.md) — string ABI
- [types/int.md](../types/int.md) — LBIM types
