# Pipe & Range Operators

## Pipe Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `\|>` | Pipe forward | `data \|> transform \|> output` |
| `<\|` | Pipe backward | `output <\| transform <\| data` |

Pipe forward passes the left-hand value as the first argument to the right-hand function:

```aria
result = data |> filter |> transform |> output;
// Equivalent to: output(transform(filter(data)))
```

## Range Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `..` | Inclusive range | `1..10` (1 through 10) |
| `...` | Exclusive range | `1...10` (1 through 9) |

Used in:
- Array slicing: `arr[1..5]`
- Loop bounds
- Pattern matching

## Failsafe Shorthand — `!!!`

```aria
!!! err;     // equivalent to calling failsafe(err)
```

## Address-Of — `@`

```aria
int32->:ptr = @value;    // get address of value
```

## Iterator Variable — `$`

The `$` symbol is the automatic iteration variable in `loop` and `till`:

```aria
loop(0, 10, 1) {
    println($);       // prints 0 through 9
}
```
