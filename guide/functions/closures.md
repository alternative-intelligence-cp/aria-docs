# Closures & Lambdas

## Anonymous Functions

```aria
func:apply = int32(func:f, int32:x) {
    pass raw f(x);
}

// Lambda passed as argument
int32:result = raw apply(func(int32:x) { pass (x * 2); }, 21);
```

## Closures

Closures capture variables from the enclosing scope:

```aria
int32:multiplier = 3;
func:scale = int32(int32:x) {
    pass (x * multiplier);    // captures 'multiplier'
}
```

## Related

- [declaration.md](declaration.md) — named function syntax
- [generics.md](generics.md) — generic function syntax
