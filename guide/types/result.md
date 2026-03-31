# Result<T> — Mandatory Error Handling

## Overview

**Every function in Aria returns `Result<T>`** (except extern functions). The compiler
forces you to handle the result — unhandled Results are compile errors. This is the
core of Aria's safety model at the function level.

## Structure

```
Result<T> = { T|NULL:value, tbb8:error, bool:is_error }
```

- Success: `value` is set, `error` is NIL, `is_error` is false
- Error: `value` is NIL, `error` is set, `is_error` is true

## Creating Results

Inside functions (not `main` or `failsafe`):

```aria
func:divide = flt64(int32:a, int32:b) {
    if (b == 0) {
        fail 1;              // return error Result
    }
    pass (a / b);            // return success Result
}
```

- `pass value` — return success, wraps value in Result
- `fail errcode` — return error, wraps tbb8 error code in Result
- Both support paren syntax: `pass(value)`, `fail(errcode)`

## Handling Results

```aria
// Safe unwrap with fallback
flt64:val = divide(10, 3) ? 0.0;

// Null coalesce
flt64:val = divide(10, 0) ?? default_value;

// Emphatic unwrap — calls failsafe on error
flt64:val = divide(10, 3)?!;

// Defaults keyword — scoped fallback for chains
flt64:val = a() + b() + c() defaults 0.0;
// Shorthand: flt64:val = a() + b() + c() ?| 0.0;

// Drop (discard result entirely)
drop divide(10, 3);
// Shorthand: _? divide(10, 3);

// Raw (extract value, ignore error)
flt64:val = raw divide(10, 3);
// Shorthand: flt64:val = _! divide(10, 3);

// Discard assignment
_ = divide(10, 3);
```

## Operator Summary

| Operator | Keyword | Meaning |
|----------|---------|---------|
| `?` | — | Safe unwrap with fallback value |
| `??` | — | Null coalesce |
| `?!` | — | Emphatic unwrap (failsafe on error) |
| `?\|` | `defaults` | Scoped fallback for expression chains |
| `_?` | `drop` | Discard Result entirely |
| `_!` | `raw` | Extract value, bypass error checking |

## Auto-Unwrap in Argument Position

When passing a Result<T> as an argument to a function expecting T, it auto-unwraps:

```aria
func:add = int32(int32:a, int32:b) { pass (a + b); }
func:get_val = int32() { pass 42; }

// get_val() returns Result<int32>, but add() expects int32
// Result auto-unwraps in argument position
int32:sum = raw add(get_val(), get_val());
```

This does **not** apply to variable assignment — you must explicitly unwrap.

## The `ok` Keyword

`ok` allows passing a variable whose value may be `unknown`:

```aria
ok maybe_unknown_value;
```

## main and failsafe

`main` and `failsafe` are special — they use `exit` instead of `pass`/`fail`:

```aria
func:main = int32() {
    exit 0;   // exit program with code 0
}

func:failsafe = int32(tbb32:err) {
    exit 1;   // must exit with code > 0
}
```

## Related

- [tbb.md](tbb.md) — TBB error propagation (value-level)
- [functions/main_failsafe.md](../functions/main_failsafe.md) — mandatory functions
- [functions/result_system.md](../functions/result_system.md) — pass/fail patterns
