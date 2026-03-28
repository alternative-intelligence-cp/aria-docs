# Rules and limit

**Category**: Types → Constraints  
**Syntax**: `Rules<T>:Name = { conditions };` / `limit<RulesName> type:var = value;`  
**Purpose**: Refinement types — constrain variable values at compile time and runtime  
**Since**: v0.2.41 (type parameters v0.2.42, member access v0.2.43, arrays & null v0.2.44)

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

## String Member Access (v0.2.43)

Rules can access `.length` on string-typed `$`:

```aria
Rules<string>:r_nonempty = {
    $.length > 0
};

Rules<string>:r_short_str = {
    $.length >= 1,
    $.length <= 50
};

limit<r_nonempty> string:name = "Alice";    // OK — length 5 > 0
limit<r_short_str> string:tag = "hello";    // OK — length 5, within 1..50
```

### Struct Field Access

Rules on struct types can access fields using `$.field`:

```aria
struct:Point = {
    int32:x,
    int32:y
};

Rules<Point>:r_first_quadrant = {
    $.x > 0,
    $.y > 0
};

limit<r_first_quadrant> Point:p = Point(10, 20);   // OK
```

---

## Array Rules (v0.2.44)

Rules can constrain arrays using `T[]` type parameter syntax:

```aria
Rules<int32[]>:r_arr_check = {
    $[0] > 0,
    $.length >= 4
};
```

### Array Element Access

Use `$[idx]` to access array elements by index:

```aria
Rules<int32[]>:r_positive_first = {
    $[0] > 0
};

limit<r_positive_first> int32[4]:data = [5, 10, 15, 20];   // OK — data[0] is 5 > 0
```

### Array Length

Use `$.length` to check the compile-time array size:

```aria
Rules<int32[]>:r_min_size = {
    $.length >= 2
};

limit<r_min_size> int32[4]:arr = [1, 2, 3, 4];     // OK — length 4 >= 2
limit<r_min_size> int32[1]:tiny = [1];              // Error! length 1 < 2
```

### Combined Array Checks

```aria
Rules<int32[]>:r_valid_data = {
    $[0] > 0,
    $.length >= 4
};

limit<r_valid_data> int32[4]:readings = [100, 200, 300, 400];   // OK
```

Note: `T[]` in the Rules type parameter matches any fixed-size array `T[N]` regardless of N.

---

## Null/NIL Checks (v0.2.44)

Rules conditions can check for null and nil values:

```aria
Rules<int32>:r_not_nil = {
    $ != NIL
};

limit<r_not_nil> int32:val = 42;    // OK — 42 != NIL (0)
```

- **NIL**: For integer types, `$ != NIL` checks that the value is not zero.
- **NULL**: For pointer types, `$ != NULL` checks that the pointer is not null.

### Combined with Range Checks

```aria
Rules<int32>:r_safe_positive = {
    $ != NIL,
    $ > 0,
    $ < 1000
};

limit<r_safe_positive> int32:id = 42;   // OK — nonzero, positive, under 1000
```

---

## Notes

- Rules names follow the same naming conventions as types (PascalCase recommended, but any identifier works).
- `$` is the only placeholder — it represents the current value being checked.
- All conditions in a Rules block use AND semantics (all must pass).
- `failsafe` must be defined in any program using `limit` with dynamic values.
- Rules are a zero-cost abstraction for literals (checked at compile time only).

---

## Z3 Static Verification (v0.2.45)

The Aria compiler can use the **Z3 SMT solver** to mathematically *prove* that Rules constraints
are satisfied — moving beyond simple compile-time constant evaluation to formal verification.

### Enabling Verification

```bash
ariac source.aria --verify              # Enable Z3 proof checking
ariac source.aria --verify-report       # Detailed proof report (implies --verify)
```

### What Gets Verified

When `--verify` is enabled, the compiler runs a Z3 verification pass (Phase 3.25) that:

1. **Rules Consistency** — Checks that each Rules declaration's conditions are not contradictory.
   For example, `$ > 100` AND `$ < 50` is impossible — Z3 detects this and emits a compile error.

2. **Literal Initializer Proofs** — For every `limit<>` variable with a literal or negative literal
   initializer, Z3 formally proves each condition holds using exact-width bitvector arithmetic
   (matching `int8`=BV8, `int16`=BV16, `int32`=BV32, `int64`=BV64) or real arithmetic for floats.

3. **Cascading Rules** — Z3 follows `limit<parent>` cascades and verifies inherited conditions too.

### Verification Report

With `--verify-report`, the compiler prints a summary:

```
=== Z3 Verification Report ===
  Proven:    19
  Disproven: 0
  Unknown:   0
  Total:     19
==============================
```

- **Proven**: Z3 mathematically proved the constraint holds for all relevant inputs.
- **Disproven**: Z3 found a counterexample — the value violates the constraint. This is a compile error.
- **Unknown**: Z3 timed out or the formula was too complex. Runtime checks are preserved.

### How It Works

Z3 verification uses the "negation check" approach:
- To prove `42 > 0`, assert `NOT(42 > 0)` and ask Z3 if it's satisfiable.
- If **unsatisfiable** → the negation has no solution → the original property holds for ALL inputs → **PROVEN**.
- If **satisfiable** → the negation has a solution (counterexample) → **DISPROVEN**.

Integer types use bitvector sorts for exact width-matching (e.g., `int32` → 32-bit signed bitvector),
ensuring proofs account for overflow semantics. Float types use Z3's real sort.

### Example

```aria
Rules:r_percentage = {
    $ >= 0,
    $ <= 100
};

limit<r_percentage> int32:score = 75;  // Z3 proves: 75 >= 0 ✓, 75 <= 100 ✓
```

Compiling with `--verify-report` shows each condition proven.

### Contradictory Rules Detection

```aria
Rules:r_impossible = {
    $ > 100,
    $ < 50
};
// With --verify: error: [z3] Rules 'r_impossible' constraints are contradictory
```

### Requirements

Z3 verification requires `libz3-dev` (or `libz3`) to be installed at build time. The compiler
is built with `ARIA_HAS_Z3=1` when Z3 is found. Without Z3, `--verify` is accepted but has no effect.
