# pick — Switch/Case

## Syntax

Aria's equivalent of switch/case:

```aria
pick (value) {
    (1i32) { println("one"); },
    (2i32) { println("two"); },
    (3i32) { println("three"); },
    (*) { println("other"); }
}
```

## Rules

1. **`(*)` wildcard is required** for types with infinite domains (int32, string, etc.)
2. Arms use `(value) { body }` syntax — value in parentheses, then block
3. Arms are separated by commas
4. No implicit fallthrough (unlike C switch)

## Fallthrough — `fall`

Explicit fallthrough to a labeled case:

```aria
pick (value) {
    (1i32) { println("one"); fall two; },
    two: (2i32) { println("two or fell from one"); },
    (*) { println("other"); }
}
```

Labels are placed before the match pattern with a colon: `label: (val) { }`.
Use `fall label;` to jump to a named case.

## With Enums

Enums support exhaustiveness checking — if all variants are covered, `(*)` is optional:

```aria
enum:Color = { RED, GREEN, BLUE };
Color:c = Color.RED;

pick (c) {
    (Color.RED)   { println("Red"); },
    (Color.GREEN) { println("Green"); },
    (Color.BLUE)  { println("Blue"); }
}
```

## Related

- [if_else.md](if_else.md) — simple branching
- [types/enum.md](../types/enum.md) — enums with exhaustiveness
