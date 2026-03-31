# Result System — pass and fail

## Overview

Inside regular functions (not `main` or `failsafe`), use `pass` and `fail` to return
Results:

- `pass value` — return success Result containing value
- `fail errcode` — return error Result with tbb8 error code

## Pass

```aria
func:square = int32(int32:x) {
    pass (x * x);
}

func:greet = NIL(string:name) {
    println(`Hello, &{name}!`);
    pass(NIL);           // NIL-returning functions must pass NIL
}
```

`pass` wraps the value in `Result<T>{value=val, error=NIL, is_error=false}`.

## Fail

```aria
func:divide = flt64(int32:a, int32:b) {
    if (b == 0) {
        fail 1;          // error code 1 — division by zero
    }
    pass (a / b);
}
```

`fail` wraps the error code in `Result<T>{value=NIL, error=code, is_error=true}`.

## Paren Syntax

Both support optional parentheses (v0.4.6+):

```aria
pass(42);        // same as: pass 42;
fail(1);         // same as: fail 1;
pass (x * x);   // parenthesized expression (always worked)
```

## return Keyword

`return` is also available and expects a full `Result<T>` instance:

```aria
return Result{ error: err, value: NIL, is_error: true };
```

In practice, `pass`/`fail` are preferred — they're more concise.

## Related

- [types/result.md](../types/result.md) — Result<T> type
- [operators/result_operators.md](../operators/result_operators.md) — ?, ?!, ?|, drop, raw
- [main_failsafe.md](main_failsafe.md) — exit instead of pass/fail
