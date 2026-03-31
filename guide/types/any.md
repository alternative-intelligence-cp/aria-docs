# Dynamic Type — `any`

## Overview

`any` is Aria's dynamically-typed value container. It can hold any type, with the actual
type tracked at runtime. Internally implemented as a fat pointer `{ptr, i64}` with a type tag.

> **Note:** `dyn` is NOT an alias for `any`. `dyn` is used exclusively for trait object
> dispatch (`dyn TraitName` in function parameters). See [traits](../advanced_features/traits.md).

## Declaration

```aria
any:box = 42i64;
```

## Access Methods

Use turbofish syntax to read, write, and resolve `any` values:

```aria
// Read with get::<T>()
int64:val = box.get::<int64>();

// Write with set::<T>(value)
box.set::<int64>(100i64);

// Resolve — consuming transform to concrete pointer
int64->:ptr = box.resolve::<int64>();
```

## Casting

`any` does not use `as` for casting. Aria has three cast forms:

```aria
// Infix arrow cast
int32:num = some_value => int32;

// Checked builtin
int32:num = @cast<int32>(some_value);

// Unchecked builtin (wraps/truncates on overflow)
int8:truncated = @cast_unchecked<int8>(large_value);
```

For `any`, use `.get::<T>()` or `.resolve::<T>()` to extract the concrete value.

## Heterogeneous Collections

```aria
any[5]:items;
items[0] = 42i64;
items[1] = "hello";
```

## Performance

Dynamic type checking has runtime overhead. **Prefer static types** wherever possible.
Use `any` only when truly needed (e.g., heterogeneous containers, plugin interfaces).

## Related

- [struct.md](struct.md) — static composite types
- [result.md](result.md) — type-safe error handling
