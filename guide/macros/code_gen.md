# Code-Generating Macros

Macros can appear at **statement position** — i.e., as standalone statements
rather than as sub-expressions. This lets macros generate entire blocks of
code that are injected at the call site.

## Statement-Position Invocation

```aria
macro:init_buffers = () {
    int32:buf_a = 0i32;
    int32:buf_b = 0i32;
    println("buffers ready");
};

func:main = int32() {
    init_buffers!();   // statement position — injects all three statements
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

The macro body is inserted directly into the function body at the call site.
All statements in the body execute in sequence.

## Generating Control Flow

Macros can generate `if`, `loop`, and `pick` blocks:

```aria
macro:repeat3 = (body_expr) {
    loop(0i32, 3i32, 1i32) {
        body_expr;
    }
};

func:main = int32() {
    repeat3!(println("tick"));
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

## Expression vs Statement Position

The same macro can be used in either position, but the compiler determines
context from the call site:

```aria
macro:compute = (x) {
    x * 2i32
};

// expression position — result is assigned
int32:n = compute!(5i32);

// statement position — result discarded
compute!(5i32);
```

## Code Generation Pattern

A common pattern is using macros to reduce boilerplate for repetitive
initialization or teardown sequences:

```aria
macro:setup_logger = () {
    string:log_prefix = "[APP]";
    println(`&{log_prefix} starting`);
};

macro:teardown_logger = () {
    println("[APP] done");
};

func:main = int32() {
    setup_logger!();
    // ... app logic ...
    teardown_logger!();
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

## Limitations

- Macros cannot `pass` a value (they are not functions)
- Macro bodies cannot use `failsafe` or `exit` directly unless generating top-level code
- Statement-position macros that produce expressions must ensure the expression is valid in that context
