# Function Declaration

## Syntax

Nitpick functions use the `func:name = return_type(params)` syntax:

```aria
func:add = int32(int32:a, int32:b) {
    pass (a + b);
};

func:greet = NIL(string:name) {
    println(`Hello, &{name}!`);
    pass NIL;
};
```

## Key Rules

- Function name follows `func:` — e.g., `func:calculate`
- Return type comes before the parameter list
- Parameters use `type:name` syntax
- `NIL` return type for functions that return nothing
- NIL-returning functions must end with `pass NIL;`
- ALL functions return `Result<T>` implicitly (except extern functions)

## Calling Functions

```aria
int32:sum = raw add(10, 20);           // raw unwrap
int32:safe = add(10, 20) ? 0;          // safe unwrap with fallback
drop greet("Alice");                    // discard Result
```

## Visibility

```aria
pub func:public_func = int32() {      // visible to other modules
    pass 42;
};

func:private_func = int32() {          // file-private (default)
    pass 99;
};
```

## Type Inference for Literals

Bare integer literals default to `int32`. Use suffixes for other widths:

```aria
func:process = NIL(int64:val) { pass(NIL); };
// WRONG: process(42);       // 42 is int32, function wants int64
// RIGHT: process(42i64);    // explicit int64 suffix
```

## Non-Pub Helper Functions (v0.19.3)

Private helper functions (no `pub` keyword) are callable from any function in
the same module, including `pub` functions. Convention: prefix helper names
with `_` to distinguish them from public API.

```aria
func:_increment = int32(int32:n) {
    pass (n + 1i32);
};

pub func:compute = int32(int32:base) {
    pass raw _increment(base);
};

func:main = int32() {
    int32:r = raw compute(9i32);   // r == 10
    exit 0;
};
```

Helpers that are only used internally do not need `pub` and will not be visible
to callers importing the module.

## `@func_name` as a First-Class Value (v0.19.3)

A function can be referenced as a value using `@func_name` syntax and passed
directly as a call argument:

```aria
func:apply = int32((int32)(int32):f, int32:x) {
    int32:result = f(x) ? -1i32;
    pass result;
};

func:double_val = int32(int32:x) {
    pass (x + x);
};

func:main = int32() {
    // Assign to lambda variable first, then pass
    (int32)(int32):fn = @double_val;
    int32:r = apply(fn, 21i32) ? -1i32;
    exit r;
};
```
