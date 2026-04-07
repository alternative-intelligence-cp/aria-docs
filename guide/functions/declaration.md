# Function Declaration

## Syntax

Aria functions use the `func:name = return_type(params)` syntax:

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
