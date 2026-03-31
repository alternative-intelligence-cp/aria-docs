# Comparison Operators

| Operator | Operation | Example | Result |
|----------|-----------|---------|--------|
| `==` | Equal | `(a == b)` | `bool` |
| `!=` | Not equal | `(a != b)` | `bool` |
| `<` | Less than | `(a < b)` | `bool` |
| `>` | Greater than | `(a > b)` | `bool` |
| `<=` | Less or equal | `(a <= b)` | `bool` |
| `>=` | Greater or equal | `(a >= b)` | `bool` |
| `<=>` | Spaceship | `(a <=> b)` | `-1`, `0`, or `1` |

## Spaceship Operator

Three-way comparison returning `-1` (less), `0` (equal), or `1` (greater):

```aria
int32:cmp = (a <=> b);
pick (cmp) {
    -1 => { println("a < b"); }
    0  => { println("a == b"); }
    1  => { println("a > b"); }
}
```

## Ternary Conditional — `is`

Aria uses `is` for ternary expressions:

```aria
int32:max = is a > b : a : b;
string:msg = is count == 0 : "empty" : "has items";
```

Syntax: `is condition : true_value : false_value`

## Float Comparison

**Never use `==` with floats.** Use epsilon comparison:

```aria
flt64:epsilon = 0.000001;
bool:equal = (abs(a - b) < epsilon);
```
