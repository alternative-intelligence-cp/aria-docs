# String — `string`

## Overview

Aria strings are **UTF-8 encoded, immutable, reference-counted, and length-tracked**
(not null-terminated internally). String operations return new strings.

## Declaration

```aria
string:name = "Alice";
string:greeting = "Hello, world!";
string:empty = "";
```

## String Literals

| Syntax | Type | Escape Processing |
|--------|------|-------------------|
| `"..."` | Regular string | Yes — `\n`, `\t`, `\\`, etc. |
| `` `...` `` | Raw string | No — backslashes are literal |
| `` `Hello, &{name}!` `` | Template literal | Interpolation with `&{expr}` |

## Interpolation

Use backtick strings with `&{expression}` for interpolation:

```aria
string:name = "Aria";
int32:version = 4;
println(`&{name} version &{version}`);  // "Aria version 4"
```

## Operations

```aria
string:full = (first + " " + last);  // concatenation
int64:len = string_length(name);      // character count (UTF-8 aware)
string:sub = text[0..5];              // substring slice (inclusive range)
```

Built-in string functions:
- `string_length(s)` — character count
- `string_contains(s, substr)` — boolean search
- `string_concat(a, b)` — concatenation (also `+` operator)

## Performance

**Don't concatenate in loops.** String concatenation creates new strings each time.
For repeated building, collect parts in an array and join.

## ABI Notes

This is critical for FFI:
- **Parameters**: `string` passes as `const char*` (single register, null-terminated)
- **Returns**: `AriaString {char* data, int64_t length}` by value (%rax=data, %rdx=length)
- C shims must use `const char*` for input params, `AriaString` struct for returns

## Related

- [operators/string_ops.md](../operators/string_ops.md) — string operators
- [io_system/print.md](../io_system/print.md) — print/println
