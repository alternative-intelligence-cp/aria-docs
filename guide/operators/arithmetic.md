# Arithmetic Operators

## Binary Operators

| Operator | Operation | Example | Notes |
|----------|-----------|---------|-------|
| `+` | Addition | `(a + b)` | Also string concatenation |
| `-` | Subtraction | `(a - b)` | |
| `*` | Multiplication | `(a * b)` | Also pointer/deref in extern |
| `/` | Division | `(a / b)` | Integer division truncates toward zero |
| `%` | Modulo | `(a % b)` | Remainder after division |

## Unary Operators

| Operator | Operation | Example |
|----------|-----------|---------|
| `++` | Increment | `x++` |
| `--` | Decrement | `x--` |

## Compound Assignment

| Operator | Equivalent | Example |
|----------|------------|---------|
| `+=` | `x = (x + y)` | `x += 5;` |
| `-=` | `x = (x - y)` | `x -= 5;` |
| `*=` | `x = (x * y)` | `x *= 2;` |
| `/=` | `x = (x / y)` | `x /= 2;` |
| `%=` | `x = (x % y)` | `x %= 3;` |

## Example

```aria
int32:a = 10;
int32:b = 3;
int32:sum = (a + b);     // 13
int32:quot = (a / b);    // 3 (truncates)
int32:rem = (a % b);     // 1
a++;                      // 11
a += 5;                   // 16
```

## Integer Division

Integer division truncates toward zero: `1000000 / 2000000 = 0`.
For float division, use `flt64` operands.
