# Borrow Semantics

**Category**: Memory Model → Borrowing  
**Syntax**: `$$i` (immutable borrow), `$$m` (mutable borrow), `$` (legacy), `#` (pin), `@` (address-of)  
**Purpose**: Compile-time memory safety without garbage collection overhead  
**Since**: v0.2.35

---

## Overview

Aria's borrow checker enforces memory safety at compile time. It tracks how variables are **borrowed** (referenced) and prevents data races, use-after-free, and dangling pointers.

Key rules:
- **N immutable borrows** OR **1 mutable borrow** at a time (never both)
- Borrows must not outlive the borrowed value
- Pinned variables cannot be reassigned while pinned

---

## Immutable Borrows (`$$i`)

An immutable borrow provides read-only access. Multiple immutable borrows can coexist:

```aria
int32:x = 42i32;
int32 $$i:ref1 = x;    // Immutable borrow
int32 $$i:ref2 = x;    // Another immutable borrow — OK
// Both can read x, neither can modify it
```

Legacy syntax with `$` operator:
```aria
int32$:ref = !$x;       // Immutable borrow (legacy syntax)
```

---

## Mutable Borrows (`$$m`)

A mutable borrow provides exclusive read-write access. Only one can exist at a time:

```aria
int32:x = 42i32;
int32 $$m:ref = x;      // Mutable borrow — exclusive access
// No other borrows of x allowed while ref exists
```

---

## Borrow Rules

| Rule | Description |
|------|-------------|
| N immutable | Multiple `$$i` borrows of the same variable are allowed |
| 1 mutable | Only one `$$m` borrow at a time |
| No mixing | Cannot have `$$i` and `$$m` borrows simultaneously |
| Lifetime | Borrows must not outlive the borrowed value |

Violations are caught at **compile time**.

---

## Pinning (`#`)

The `#` operator pins a variable, preventing it from being moved or relocated by the GC:

```aria
int32:gc_var = 42i32;
wild int32->:pinned = #gc_var;   // Pin — stable pointer
// gc_var cannot be reassigned while pinned
```

Pins release when the pin reference goes out of scope:

```aria
int32:x = 10i32;
{
    wild int32->:pin = #x;
    // x is pinned here
}
// pin out of scope — x is free again
```

---

## Address-Of (`@`)

The `@` operator takes the raw address of a variable (primarily for FFI):

```aria
int32:x = 42i32;
wild int32->:ptr = @x;    // Raw pointer to x
```

**Warning**: Raw pointers bypass borrow checking. Use only at FFI boundaries.

---

## Wild Pointers

The `wild` qualifier marks unmanaged pointers (FFI interop):

```aria
wild int32->:raw_ptr = @some_var;     // Thin pointer (FFI)
```

Wild pointers are not tracked by the borrow checker — they represent trust boundaries.

---

## Rules Summary

1. `$$i` — immutable borrow, multiple allowed
2. `$$m` — mutable borrow, exclusive
3. `#` — pin, prevents GC relocation
4. `@` — address-of, raw pointer (FFI only)
5. `$` — legacy immutable borrow operator
6. `wild` — unmanaged pointer qualifier

---

## See Also

- [Memory Model](../memory_model/) — Arena, pool, and GC allocation
- [Pointers](../types/pointers.md) — Pointer types and FFI
