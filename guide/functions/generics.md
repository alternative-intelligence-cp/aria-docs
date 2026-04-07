# Generics

## Generic Functions

```aria
func:identity = T(T:value) {
    pass value;
};

func:max = T(T:a, T:b) {
    if (a > b) {
        pass a;
    }
    pass b;
};
```

Generic type parameters are monomorphized at compile time — one specialized version
per concrete type used.

## Turbofish Syntax

Explicitly specify type arguments with `::<T>`:

```aria
int32:val = identity::<int32>(42);
flt64:pi = identity::<flt64>(3.14);
```

## Generic Constraints

Constraints limit which types can be used via trait bounds:

```aria
func<T: Addable>:sum = int32(T:a, T:b) {
    pass (a + b);
};
```

See [advanced_features/traits.md](../advanced_features/traits.md) for trait definitions.

## Related

- [advanced_features/traits.md](../advanced_features/traits.md) — trait constraints
- [advanced_features/rules.md](../advanced_features/rules.md) — refinement types
