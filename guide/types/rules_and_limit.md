# Rules and limit

**Category**: Types → Constraints  
**Syntax**: `Rules<T>:Name = { conditions };` / `limit<RulesName> type:var = value;`  
**Purpose**: Refinement types — constrain variable values at compile time and runtime  
**Since**: v0.2.41 (type parameters since v0.2.42)

---

## Overview

**Rules** define named sets of conditions that constrain what values a variable can hold. The **limit** keyword applies a Rules set to a variable declaration, creating a **refinement type** — a type with additional value constraints enforced by the compiler.

- **Compile-time**: Literal initializers are checked during compilation; violations are errors.
- **Runtime**: Dynamic values are checked at assignment; violations trigger `failsafe`.

The `$` placeholder represents the value being tested, following the same convention used in `till` loops.

---

## Declaring Rules

```aria
Rules:r_lt_100 = {
    $ < 100
};
```

Each condition is a boolean expression using `$` as the value placeholder. Multiple conditions are separated by commas — all must be satisfied (AND semantics):

```aria
Rules:r_special = {
    $ >= -2,
    $ <= 100,
    $ % 2 == 0,
    $ != 0,
    $ != 50
};
```

---

## Type Parameters (v0.2.42)

Rules can specify which types they accept using type parameters in angle brackets:

```aria
Rules<int32>:r_positive = {
    $ > 0
};

Rules<uint8>:r_byte_range = {
    $ > 0,
    $ < 100
};

Rules<flt64>:r_unit = {
    $ >= 0.0,
    $ <= 1.0
};
```

### Multiple Type Parameters

A single Rules set can accept multiple types:

```aria
Rules<int8, uint8>:r_small = {
    $ < 100
};

limit<r_small> int8:a = 50;    // OK — int8 is in the type list
limit<r_small> uint8:b = 50u8; // OK — uint8 is in the type list
limit<r_small> int32:c = 50;   // Compile error! int32 is not int8 or uint8
```

### Type Safety

When type parameters are specified, the compiler enforces that `limit<>` is only applied to variables matching those types:

```aria
Rules<uint8>:r_byte = { $ > 0 };
limit<r_byte> int32:x = 50;    // Error: limit<r_byte> can only be applied to uint8, got 'int32'
```

### Untyped Rules (Backward Compatible)

Rules without type parameters accept any numeric type, preserving v0.2.41 behavior:

```aria
Rules:r_any_pos = {
    $ > 0
};

limit<r_any_pos> int32:a = 10;   // OK
limit<r_any_pos> flt64:b = 3.14; // OK
limit<r_any_pos> uint8:c = 50u8; // OK
```

---

## Applying Rules with limit

Use `limit<RulesName>` before the type in a variable declaration:

```aria
limit<r_lt_100> int32:x = 50;     // OK — 50 < 100
limit<r_lt_100> int32:y = 100;    // Compile error! 100 is not < 100
```

The limit constraint follows the variable through reassignment:

```aria
limit<r_lt_100> int32:x = 50;
x = 99;     // OK — 99 < 100
x = 200;    // Runtime violation → failsafe called
```

---

## Cascading Rules

Rules can include other Rules using `limit<other>` inside the block:

```aria
Rules:r_even = {
    $ % 2 == 0
};

Rules:r_positive_even = {
    limit<r_even>,
    $ > 0
};

// r_positive_even requires: even AND positive
limit<r_positive_even> int32:val = 42;   // OK
```

Cascaded rules are evaluated recursively — all conditions from all included rules must pass.

---

## Supported Types

Rules can be applied to any numeric type:

- **Integers**: `int8`, `int16`, `int32`, `int64`, `uint8`, `uint16`, `uint32`, `uint64`
- **Floats**: `flt32`, `flt64`

```aria
Rules:r_unit_range = {
    $ >= 0.0,
    $ <= 1.0
};

limit<r_unit_range> flt64:alpha = 0.75;   // OK
```

---

## Compile-Time vs Runtime Checking

| Initializer | Check Time | On Violation |
|-------------|-----------|--------------|
| Literal (`= 50`) | Compile time | Compiler error |
| Negative literal (`= -2`) | Compile time | Compiler error |
| Expression / variable | Runtime | `failsafe` called, program exits |

### Compile-Time Error Example

```aria
Rules:r_positive = {
    $ > 0
};

limit<r_positive> int32:x = -5;
// Error: limit<r_positive> violation: value -5 fails rule condition
```

### Runtime Violation Example

```aria
Rules:r_lt_100 = {
    $ < 100
};

func:failsafe = NIL() {
    drop(puts("Limit violation!"));
};

func:main = int32() {
    limit<r_lt_100> int32:x = 50;   // OK at compile time
    int32:val = 200;
    x = val;                          // Runtime: failsafe called, exit(1)
    pass(0);
};
```

---

## Condition Operators

Any comparison or arithmetic expression using `$` is valid:

| Pattern | Meaning |
|---------|---------|
| `$ < N` | Less than N |
| `$ <= N` | Less than or equal to N |
| `$ > N` | Greater than N |
| `$ >= N` | Greater than or equal to N |
| `$ == N` | Equal to N |
| `$ != N` | Not equal to N |
| `$ % N == M` | Modulo check (e.g., divisibility) |

Arithmetic sub-expressions are fully supported:

```aria
Rules:r_divisible_by_3 = {
    $ % 3 == 0
};

Rules:r_in_range = {
    $ >= 10,
    $ <= 100,
    $ % 5 == 0
};
```

---

## Complete Example

```aria
extern func:puts = int32(string:s);

Rules:r_percentage = {
    $ >= 0,
    $ <= 100
};

Rules:r_even = {
    $ % 2 == 0
};

Rules:r_even_percentage = {
    limit<r_percentage>,
    limit<r_even>
};

func:failsafe = NIL() {
    drop(puts("Invalid value!"));
};

func:main = int32() {
    limit<r_even_percentage> int32:score = 80;
    drop(puts("Score accepted"));
    
    score = 42;    // OK — even and 0-100
    score = 50;    // OK
    // score = 101;  // Would trigger failsafe (> 100)
    // score = 51;   // Would trigger failsafe (odd)
    
    pass(0);
};
```

---

## Notes

- Rules names follow the same naming conventions as types (PascalCase recommended, but any identifier works).
- `$` is the only placeholder — it represents the current value being checked.
- All conditions in a Rules block use AND semantics (all must pass).
- `failsafe` must be defined in any program using `limit` with dynamic values.
- Rules are a zero-cost abstraction for literals (checked at compile time only).
