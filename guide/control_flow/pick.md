# pick — Switch/Case

## Syntax

Aria's equivalent of switch/case:

```aria
pick (value) {
    1 => { println("one"); }
    2 => { println("two"); }
    3 => { println("three"); }
    (*) => { println("other"); }   // wildcard (required for infinite domains)
}
```

## Rules

1. **`(*)` wildcard is required** for types with infinite domains (int32, string, etc.)
2. No trailing semicolon after the closing brace
3. Each case uses `=>` followed by a block
4. No implicit fallthrough (unlike C switch)

## Fallthrough — `fall`

Explicit fallthrough to a labeled case:

```aria
pick (value) {
    1 => { println("one"); fall two; }
    two: 2 => { println("two or fell from one"); }
    (*) => { println("other"); }
}
```

Use `fall label` to jump to a named case.

## With Enums

Enums support exhaustiveness checking — if all variants are covered, `(*)` is optional:

```aria
enum:Color = { RED, GREEN, BLUE };
Color:c = Color.RED;

pick (c) {
    Color.RED   => { println("Red"); }
    Color.GREEN => { println("Green"); }
    Color.BLUE  => { println("Blue"); }
}
```

## Negation — `(!)`

```aria
pick (value) {
    (!) 0 => { println("not zero"); }    // matches anything except 0
    (*) => { println("zero"); }
}
```

## Related

- [if_else.md](if_else.md) — simple branching
- [types/enum.md](../types/enum.md) — enums with exhaustiveness
