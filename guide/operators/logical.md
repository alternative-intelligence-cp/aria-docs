# Logical Operators

| Operator | Operation | Example |
|----------|-----------|---------|
| `&&` | Logical AND | `(a && b)` |
| `\|\|` | Logical OR | `(a \|\| b)` |
| `!` | Logical NOT | `(!a)` |

All logical operators use **short-circuit evaluation**:
- `&&` — skips right operand if left is false
- `||` — skips right operand if left is true

## Example

```aria
bool:valid = (age > 0 && age < 150);
bool:allowed = (is_admin || has_permission);
bool:denied = (!allowed);
```

## In Control Flow

```aria
if (is_ready && has_data) {
    process();
}

when (!done) then {
    work();
}
```
