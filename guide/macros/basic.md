# Basic Macros

## Declaration

Use `macro:name = (params) { body };` to declare a macro:

```aria
// zero-argument macro
macro:greet = () {
    println("hello");
};

// one-argument macro
macro:double = (x) {
    x + x;
};

// two-argument macro
macro:add_and_print = (a, b) {
    println(`sum: &{a + b}`);
};
```

A macro body is a block `{ ... }` containing statements and expressions.
The last expression in the body (without a trailing `;`) becomes the
expansion result in expression position.

## Invocation

Invoke a macro with `name!(args)`:

```aria
// expression position — result used as a value
int32:n = double!(5i32);        // expands to: 5 + 5

// statement position — result discarded
greet!();                       // expands to: println("hello");
add_and_print!(10i32, 20i32);   // expands to: println(...)
```

## Rules

- Macro names follow `macro:` — e.g., `macro:my_macro`
- Parameters are names only (no type annotations) — types are inferred from the call site
- The `!` suffix marks an invocation as a macro call (not a function call)
- Macros must be declared before they are used
- Macro bodies must end with `};` (the `}` closes the block, `;` closes the declaration)

## Scope

Macros are file-scoped. They are not exported from modules and cannot be
imported with `use`. To share a macro across files, define it in a shared
include or duplicate it.

## Example: min macro

```aria
macro:min = (a, b) {
    if (a < b) { a } else { b }
};

func:main = int32() {
    int32:x = min!(10i32, 20i32);   // expands to: if (10 < 20) { 10 } else { 20 }
    println(`min = &{x}`);
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```
