# Built-in Macros

Nitpick provides four built-in macros. They are always available — you do
not need to import or declare them. They **cannot be redefined**.

## `assert!(condition)`

Checks a boolean expression at compile time (static assertion).

```aria
assert!(1 == 1);         // always passes
assert!(true);           // always passes
assert!(some_const > 0); // passes if some_const > 0 is known at compile time
```

If the condition evaluates to `false`, the compiler emits an error:

```
error: assertion failed: 1 == 2
```

`assert!` works on constant expressions. For runtime checks, use the
`assert_rt!` pattern (write an `if` with `unreachable!()`).

## `todo!()`

Marks code that has not been implemented yet. The program compiles but the
`todo!` site will panic at runtime.

```aria
func:load_config = string() {
    todo!();    // placeholder — implement later
};
```

At runtime, executing a `todo!()` site triggers a panic with the message:
```
not yet implemented
```

## `unreachable!()`

Marks a code path that should never be reached. If the path is reached at
runtime, the program panics.

```aria
func:classify = string(int32:n) {
    if (n > 0i32) { pass "positive"; }
    if (n < 0i32) { pass "negative"; }
    if (n == 0i32) { pass "zero"; }
    unreachable!();   // compiler appeased; runtime panics if somehow reached
};
```

Use `unreachable!()` at the end of exhaustive match-like chains to signal
that you have covered all cases.

## `cfg!(platform)`

Compiles a block conditionally based on the target platform.

```aria
cfg!(linux) {
    println("running on Linux");
};

cfg!(windows) {
    println("running on Windows");
};
```

Supported platform tokens: `linux`, `windows`, `macos`.

The block for the **current** compile target is included; all others are
discarded during compilation.

### Platform Detection Example

```aria
func:main = int32() {
    cfg!(linux) {
        exit(0);   // Linux: success
    };
    cfg!(windows) {
        exit(2);   // Windows: different code
    };
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

## Cannot Redefine Built-ins

Attempting to declare a macro with a built-in name is a compile error:

```aria
macro:assert = () { exit(0); };
// error: Cannot redefine built-in macro 'assert'
```
