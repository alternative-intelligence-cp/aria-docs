# Standard Library — Math

## Functions

Standard math functions available via extern libm:

| Function | Description |
|----------|-------------|
| `sin(x)` | Sine |
| `cos(x)` | Cosine |
| `sqrt(x)` | Square root |
| `abs(x)` | Absolute value |
| `pow(x, y)` | Power |
| `log(x)` | Natural logarithm |
| `floor(x)` | Floor |
| `ceil(x)` | Ceiling |

## Usage

```aria
extern "libm" {
    func:sin = double(double:x);
    func:cos = double(double:x);
    func:sqrt = double(double:x);
}

flt64:result = sin(3.14159);
```

## Related

- [types/flt.md](../types/flt.md) — floating point types
- [types/fix256.md](../types/fix256.md) — deterministic math alternative
