# Enums

**Category**: Types → Composite  
**Syntax**: `enum:Name = { VARIANT1, VARIANT2, ... };`  
**Purpose**: Named sets of integer constants with type safety  
**Since**: v0.2.39

---

## Overview

Enums define a **named set of variants**, each backed by an `int64` value. Unlike bare integer constants, enum values carry type identity — the type checker knows that `Color.RED` is a `Color`, not just an integer.

---

## Declaration

### Auto-Numbered (from 0)

```aria
enum:Color = { RED, GREEN, BLUE };
```

| Variant | Value |
|---------|-------|
| `RED`   | 0     |
| `GREEN` | 1     |
| `BLUE`  | 2     |

### Explicit Values

```aria
enum:HttpStatus = {
    OK = 200,
    NOT_FOUND = 404,
    SERVER_ERROR = 500
};
```

### Mixed Auto/Explicit

```aria
enum:Priority = {
    LOW,           // 0
    MEDIUM,        // 1
    HIGH = 10,     // 10
    CRITICAL       // 11 (last + 1)
};
```

Auto-numbering always continues from **last assigned value + 1**.

---

## Accessing Variants

Use dot syntax: `EnumName.VARIANT`

```aria
int64:status = HttpStatus.OK;           // 200
int64:color = Color.GREEN;              // 1
```

---

## Enum-Typed Variables

Declare variables with the enum as their type:

```aria
Color:my_color = Color.RED;
HttpStatus:code = HttpStatus.NOT_FOUND;
```

Enum-typed variables enforce that only values from the same enum are assigned.

---

## Comparison

Enum values support `==` and `!=`:

```aria
Color:a = Color.RED;
Color:b = Color.BLUE;

if (a != b) {
    drop(println("Different colors"));
};
```

---

## Interop with int64

Enum values are represented as `int64` at the ABI level. Assignment to `int64` variables is permitted:

```aria
int64:raw_value = Color.GREEN;  // raw_value = 1
```

---

## Exhaustiveness

Enums integrate with `pick` statement exhaustiveness checking. The compiler can verify that all variants are covered:

```aria
Color:c = Color.RED;
pick (c) {
    when (Color.RED)   { drop(println("red")); };
    when (Color.GREEN) { drop(println("green")); };
    when (Color.BLUE)  { drop(println("blue")); };
};
```

---

## Rules

1. Variant names must be unique within an enum
2. Values must be valid `int64` integers
3. Auto-numbering starts at 0, or last explicit value + 1
4. Enum names must start with an uppercase letter (convention)
5. Enums are declared at top level (not inside functions)

---

## See Also

- [pick](../control_flow/pick.md) — Pattern matching with exhaustiveness
- [int64](int64.md) — Underlying representation
