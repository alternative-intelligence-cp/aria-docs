# Comptime and Macros

Nitpick has two compile-time mechanisms:

| | Macros | Comptime |
|---|---|---|
| Operates on | AST fragments | Values & types |
| Hygienic | Yes | N/A (no capture) |
| Type-checked? | After expansion | Yes, before evaluation |
| Pure? | Yes (rewriting) | Yes (no I/O) |
| Best for | Patterns, syntactic sugar | Math, reflection, tables |

They cooperate: a macro can **expand to** a `comptime(...)` expression, and
a `comptime func:` can **call** a macro-expanded helper.

## Macro Expanding to Comptime

```aria
macro:square_const = (n) {
    comptime(n * n);
};

fixed int32:nine = square_const!(3);   // expands to comptime(3 * 3) = 9
```

This pattern gives you a name that *looks* like a function but always
folds at the call site.

## Comptime Calling Macros

A macro is expanded before the type checker runs, so by the time the
const evaluator inspects a `comptime func:` body, all macro invocations
inside it are already resolved.

```aria
macro:dbl = (x) { x + x; };

comptime func:quad = int32(int32:n) {
    pass dbl!(dbl!(n));
};

fixed int32:f = comptime(quad(3));   // 12
```

## When to Choose Which

- Choose **macros** when you need to manipulate code shape (statement
  generation, repeated patterns, conditional compilation `cfg!`).
- Choose **comptime** when you need to compute values or inspect types.
- Use **both** when you want a syntactic shorthand that resolves to a
  constant.

See `guide/macros/README.md` for the macro reference.
