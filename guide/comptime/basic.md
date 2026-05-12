# Basic Comptime

Three forms of comptime are supported:

## 1. `comptime(expr)`

Evaluate a single expression at compile time. The compiler folds the result
into the surrounding code as a literal constant.

```aria
fixed int32:KB = comptime(1024);
fixed int32:MB = comptime(KB * 1024);
fixed int32:GB = comptime(MB * 1024);

// integers, floats, bools, strings — all work
fixed string:greeting = comptime("Hello, " + "world!");
fixed bool:debug      = comptime(true);
```

The expression must be **pure** — no I/O, no GC allocation, no extern calls.
See [limitations.md](limitations.md) for the full rules.

## 2. `comptime { ... }` blocks

Wrap multiple statements in a `comptime { ... }` block. Local bindings,
loops, and conditionals all work, and the final value (if any) is
elaborated into the program at the block's position.

```aria
comptime {
    fixed int32:check = 2 + 2;
};
```

Use blocks when you need to compute several intermediate values, or to run
side-effect-free setup at compile time.

## 3. `comptime func:` declarations

Declare a function whose body runs at compile time. The function body is
ordinary Aria, but it is callable only from `comptime(...)` contexts (or
from other `comptime func:` bodies).

```aria
comptime func:double = int32(int32:n) { pass n + n; };

fixed int32:eight = comptime(double(4));
```

`comptime func:` is the recommended way to factor reusable CTFE logic.
The compiler caches results across call sites with the same arguments
(see [ctfe.md](ctfe.md) for the memoization rules).

## Type Inference

The result of any comptime form has a fully-known type at the point of use.
You can use it anywhere a literal would appear:

```aria
fixed int32[comptime(8)]:ring = [0, 0, 0, 0, 0, 0, 0, 0];
```

## Errors

If the comptime expression hits an error (overflow, division by zero,
unbounded recursion), the compiler reports a diagnostic at the source line
of the `comptime(...)` call. See [debug.md](debug.md).
