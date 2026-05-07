# pick — Switch/Case

## Syntax

Nitpick's equivalent of switch/case:

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

## Struct Field Patterns (v0.19.1)

When the selector is a struct value, use a field destructuring pattern to bind
struct fields as locals for the arm body:

```aria
struct:Point = {
    int32:x;
    int32:y;
};

Point:p = Point{ x: 10i32, y: 20i32 };

pick (p) {
    (Point{ x, y }) {
        // x and y are local copies of p.x and p.y
        println(`&{x + y}`);
    }
}
```

**Rules:**

- The type name in the pattern must match the selector's exact struct type.
- Only the fields you name are bound; use `_` for fields you want to ignore.
- Bound fields are immutable copies; mutations do not affect the original struct.
- A struct pattern always matches — no wildcard `(*)` is required for
  exhaustiveness when a struct pattern is present.
- An unknown field name in the pattern is a compile-time error.

**Partial destructure with `_`:**

```aria
Color:c = Color{ r: 10i32, g: 20i32, b: 30i32 };

pick (c) {
    (Color{ r, _, _ }) {
        // only r is bound; g and b are ignored
        println(`&{r}`);
    }
}
```

## Related

- [if_else.md](if_else.md) — simple branching
- [types/enum.md](../types/enum.md) — enums with exhaustiveness
- [types/struct.md](../types/struct.md) — struct update syntax
