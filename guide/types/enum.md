# Enum

## Overview

Enumerations define a set of named integer constants. Backed by `int64` at the ABI level.
Since v0.2.39.

## Declaration

```aria
enum:Color = { RED, GREEN, BLUE };                        // auto-numbered: 0, 1, 2
enum:HttpStatus = { OK = 200, NOT_FOUND = 404, ERROR = 500 };  // explicit values
enum:Mixed = { A, B = 10, C };                            // C = 11 (continues from last)
```

## Usage

```aria
Color:my_color = Color.RED;
int64:val = Color.GREEN;     // enums convert to int64

pick (my_color) {
    (Color.RED)   { drop println("Red"); },
    (Color.GREEN) { drop println("Green"); },
    (Color.BLUE)  { drop println("Blue"); },
    (*) {}
}
```

Enum comparison:

```aria
Color:a = Color.RED;
Color:b = Color.BLUE;
if (a != b) {
    drop println("Different colors");
}
```

## Rules

- Variant names must be unique within the enum
- Values must be valid `int64`
- Convention: variant names in UPPER_CASE
- Declared at top level (not inside functions)
- Supports `==` and `!=` comparison
- Enum-typed variables enforce same-enum assignment
- Integrates with `pick` statement exhaustiveness checking

## Related

- [struct.md](struct.md) — composite types
- [control_flow/pick.md](../control_flow/pick.md) — switch/case with exhaustiveness
