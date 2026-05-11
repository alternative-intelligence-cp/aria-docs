# Recursive Macros

Macros can invoke themselves recursively. The compiler enforces a **depth
limit** (default: 64 levels) to prevent infinite expansion loops.

## Basic Recursive Macro

```aria
// count down to zero, printing each step
macro:countdown = (n) {
    if (n > 0i32) {
        println(`&{n}`);
        countdown!(n - 1i32);
    }
};

func:main = int32() {
    countdown!(3i32);
    // expands to: if (3 > 0) { println("3"); if (2 > 0) { println("2"); ... } }
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

## Depth Limit

If a macro expands recursively more than 64 times, the compiler emits a
diagnostic error:

```
error: macro 'countdown' exceeded maximum recursion depth (64)
```

The limit exists to catch runaway macro expansions at compile time. There is
no runtime recursion for macros — everything is unrolled at compile time.

## Mutual Recursion

Two macros can invoke each other. The same depth limit applies to the total
combined expansion depth:

```aria
macro:is_even = (n) {
    if (n == 0i32) { true } else { is_odd!(n - 1i32) }
};

macro:is_odd = (n) {
    if (n == 0i32) { false } else { is_even!(n - 1i32) }
};
```

## Design Note

Recursive macros are best suited for small, bounded recursion (e.g., unrolling
a fixed-count loop, building a nested expression). For open-ended or
data-driven repetition, use `loop()` in regular code instead.
