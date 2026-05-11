# Variadic Macros

Variadic macros accept a variable number of arguments using a **rest
parameter** (`..?name`).

## Syntax

```aria
macro:log_all = (..?args) {
    // args is bound to the list of all passed arguments
};
```

The `..?` prefix before the parameter name indicates a rest parameter.
It must be the last (or only) parameter.

## Example: printing multiple values

```aria
macro:print_all = (..?vals) {
    // each element of vals is substituted in sequence
    // the body is replicated for each argument
};
```

> **Note:** Variadic macro bodies use each argument in the order provided.
> The compiler replicates the body once per argument, substituting the
> rest parameter at each occurrence.

## Practical Example

```aria
macro:assert_all = (..?conds) {
    assert!(conds);   // replicated for each argument
};

func:main = int32() {
    assert_all!(true, 1 == 1, 2 > 0);
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

This expands to:
```aria
assert!(true);
assert!(1 == 1);
assert!(2 > 0);
```

## Mixing Fixed and Rest Parameters

```aria
macro:labeled_print = (label, ..?vals) {
    // label is fixed; vals captures the remaining args
    println(`&{label}: &{vals}`);
};
```

## Restrictions

- Only one rest parameter is allowed per macro
- The rest parameter must be last
- A variadic macro called with zero rest arguments expands to an empty block
