# if / else

## Syntax

`if` **requires parentheses** around the condition:

```aria
if (x > 0) {
    println("positive");
} else if (x == 0) {
    println("zero");
} else {
    println("negative");
}
```

## Rules

- Parentheses around condition are **mandatory**
- No trailing semicolon after the closing brace
- Braces are required (no single-statement if)

## Common Patterns

```aria
// Guard clause
if (input == NIL) {
    fail 1;
}

// With logical operators
if (age >= 18 && has_id) {
    println("Allowed");
}
```
