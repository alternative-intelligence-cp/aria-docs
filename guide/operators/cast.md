# Cast & Type Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=>` | Cast / type conversion | `val => int32` |
| `@cast<T>` | Explicit cast | `@cast<int32>(val)` |
| `@sizeof` | Size of type in bytes | `@sizeof(int32)` |
| `@typeof` | Type of expression | `@typeof(val)` |
| `::<T>` | Turbofish type annotation | `func::<int32>()` |

## Cast Operator

```aria
flt64:f = 3.14;
int32:i = f => int32;          // truncates: 3
int64:wide = i => int64;       // widening: safe
```

## Optional Types — `<T>?`

```aria
int64?:maybe = get_value();    // may be NIL
```

## Type Annotations

```aria
int32:x = 42;           // explicit type annotation
x := 42;                 // inferred type
```

## Pin — `#`

```aria
#value;                  // pin value (prevent move)
```

## Borrow Operators

| Operator | Meaning |
|----------|---------|
| `$$i` | Immutable borrow |
| `$$m` | Mutable borrow |

```aria
$$i:ref = value;         // immutable borrow
$$m:mut_ref = value;     // mutable borrow
```
