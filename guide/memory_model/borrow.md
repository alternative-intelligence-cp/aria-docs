# Borrow Semantics

## Overview

Nitpick has borrow operators for safe reference passing:

| Operator | Meaning |
|----------|---------|
| `$$i` | Immutable borrow — read-only reference |
| `$$m` | Mutable borrow — read-write reference |

## Usage

```aria
$$i int32:ref = value;        // immutable borrow
$$m int32:mut_ref = value;    // mutable borrow
```

Multiple immutable borrows are allowed simultaneously. Only one mutable borrow at a time.
The compiler enforces the 1-mut XOR N-immut rule.

## Array Element Borrows (v0.19.0)

Individual array elements can be borrowed by literal or variable index.

**Literal index** — the borrow path is statically known (`arr[k]`):

```aria
int32[4]:arr = [10, 20, 30, 40];
$$m int32:first = arr[0];     // borrows path arr[0]
$$m int32:third = arr[2];     // borrows path arr[2] — disjoint, OK
first = 99;                   // arr[0] = 99 after release
```

**Variable (dynamic) index** — the slot is not known at compile time;
the borrow checker maps it to `arr[*]` (the whole array):

```aria
int32:i = 1;
$$i int32:elem = arr[i];      // immutable borrow of arr[*]
// arr is locked while elem is live

$$m int32:slot = arr[i];      // mutable borrow — writes back on release
slot = 99;
```

Two borrows using variable indices on the same array always conflict (ARIA-023).
A literal-index borrow and a variable-index borrow on the same array also
conflict. See [types/array.md](../types/array.md) for the full rules.

## Lambda Function Pointer Borrows (v0.19.2)

Lambda variables (`(ReturnType)(ParamTypes):name = lambda`) are treated as
owned values by the borrow checker. Passing a lambda to a higher-order function
that takes a `$$i` function-pointer parameter works the same as any immutable
borrow:

```aria
(int32)(int32):dbl = int32(int32:x) { int32:v = x + x; pass v; };
// dbl is an owned lambda variable — no borrow restrictions on calls
int32:r = dbl(21i32) ? -1i32;
```

Lambda bodies do **not** capture variables from the enclosing scope. All values
a lambda uses must be passed as explicit parameters.

## Inter-Procedural Borrows (v0.19.2)

When calling a function that takes a `$$m` parameter, the borrow checker
verifies that the argument is not already borrowed at the call site:

```aria
func:modify = NIL($$m int32:x) { x = x + 1; pass NIL; };

int32:val = 10;
$$m int32:alias = val;        // val is borrowed

drop modify(val);             // ARIA-023 — val is already mutably borrowed
```

Release the borrow before passing:

```aria
// Let alias go out of scope first, then call modify(val)
```

## Pin

The `#` operator pins a value, preventing it from being moved:

```aria
#value;                 // pin — cannot be moved after this
```

## Borrow Checker Error Reference

| Code | Meaning |
|------|---------|
| ARIA-020 | Dollar-borrow syntax (`$x`) used — not valid Nitpick syntax |
| ARIA-023 | Borrow conflict: mutable borrow while already borrowed, or two conflicting array-element borrows |
| ARIA-024 | Borrow alias used as struct update base — must use a plain owned variable |
| ARIA-025 | Pin dereference violation |
| ARIA-026 | Assignment to a variable while it is borrowed |

## Related

- [overview.md](overview.md) — memory model overview
- [types/pointer.md](../types/pointer.md) — pointer types
- [types/array.md](../types/array.md) — array element borrows
