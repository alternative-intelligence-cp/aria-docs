# Result Operators

These operators control how `Result<T>` values are handled. Every function call in Aria
returns `Result<T>` — these operators let you unwrap, propagate, or discard results.

## Operator Table

| Operator | Keyword | Meaning | Example |
|----------|---------|---------|---------|
| `?` | — | Safe unwrap with fallback | `val = func() ? 0;` |
| `??` | — | Null coalesce | `val = func() ?? default;` |
| `?!` | — | Emphatic unwrap (failsafe on error) | `val = func()?!;` |
| `?\|` | `defaults` | Scoped fallback for chains | `val = a() + b() ?| 0;` |
| `_?` | `drop` | Discard Result entirely | `_? func();` |
| `_!` | `raw` | Extract value, bypass error | `val = _! func();` |

## Safe Unwrap — `?`

Returns the value on success, or the fallback on error:

```aria
int32:val = get_number() ? 0;    // 0 if get_number fails
```

## Null Coalesce — `??`

```aria
string:name = get_name() ?? "unknown";
```

## Emphatic Unwrap — `?!`

Calls `failsafe` if the Result is an error:

```aria
int32:val = must_succeed()?!;    // aborts via failsafe on error
```

## Defaults — `?|` / `defaults`

Provides a scoped fallback for an entire expression chain:

```aria
// Any error in a(), b(), or c() uses 0 as fallback
int32:val = a() + b() + c() defaults 0;

// Shorthand operator form
int32:val = a() + b() + c() ?| 0;

// Nested defaults
int32:val = func(a() + 2 + c(2 + d() defaults 3) defaults 2);
```

Defaults scopes to the nearest enclosing expression, not the entire statement.

## Drop — `_?` / `drop`

Silently discards the Result (no error check, no value extraction):

```aria
drop setup();         // keyword form
_? setup();           // shorthand form
```

Only works on expressions, NOT on named variables.

## Raw — `_!` / `raw`

Extracts the value field, ignoring any error:

```aria
int32:val = raw risky_func();   // keyword form
int32:val = _! risky_func();    // shorthand form
```

## Paren Syntax

All keywords support optional parentheses (v0.4.6+):

```aria
drop(func());    // same as: drop func();
raw(func());     // same as: raw func();
pass(value);     // same as: pass value;
fail(code);      // same as: fail code;
exit(0);         // same as: exit 0;
```

## Related

- [types/result.md](../types/result.md) — Result<T> type documentation
- [functions/result_system.md](../functions/result_system.md) — pass/fail patterns
