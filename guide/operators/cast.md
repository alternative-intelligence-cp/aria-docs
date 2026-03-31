# Cast & Type Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=>` | Cast / type conversion | `val => int32` |
| `@cast<T>` | Explicit checked cast | `@cast<int32>(val)` |
| `@cast_unchecked<T>` | Unchecked cast | `@cast_unchecked<int32>(val)` |
| `::<T>` | Turbofish type annotation | `func::<int32>()` |

## Cast Operator

```aria
flt64:f = 3.14;
int32:i = f => int32;          // truncates: 3
int64:wide = i => int64;       // widening: safe
```

## Built-in Cast Functions

```aria
int32:val = @cast<int32>(f);              // checked
int32:val = @cast_unchecked<int32>(f);    // unchecked (no validation)
```

## Optional Types — `<T>?`

```aria
int64?:maybe = get_value();    // may be NIL
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
$$i int32:ref = value;         // immutable borrow
$$m int32:mut_ref = value;     // mutable borrow
```
