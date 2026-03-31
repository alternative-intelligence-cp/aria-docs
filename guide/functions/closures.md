# Closures & Lambdas

## Function Pointer Variables

Lambda expressions are assigned to typed variables using function pointer syntax:

```aria
(int32)(int32):identity = int32(int32:x) { pass x; };

(int32)(int32, int32):add = int32(int32:a, int32:b) {
    int32:sum = a + b;
    pass sum;
};
```

The variable type is `(ReturnType)(ParamTypes)` — return type first, then parameter types.

## Passing as Arguments

Higher-order functions accept function pointers as parameters:

```aria
func:apply = int32((int32)(int32):f, int32:x) {
    int32:result = f(x) ? -1i32;
    pass result;
};

(int32)(int32):dbl = int32(int32:x) { int32:v = x + x; pass v; };
int32:r = apply(dbl, 21i32) ? -1i32;
```

**Note:** Lambdas must be assigned to a named variable first, then passed by name.
There are no anonymous inline lambdas at call sites.

## Related

- [declaration.md](declaration.md) — named function syntax
- [generics.md](generics.md) — generic function syntax
