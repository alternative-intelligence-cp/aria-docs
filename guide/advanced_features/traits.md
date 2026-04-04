# Traits

## Overview

Traits define shared behavior that types can implement. They serve as constraints
for generic type parameters.

```aria
trait:Describable = {
    func:describe = int32(int32:self);
};
```

## Implementation

```aria
impl:Describable:for:int32 = {
    func:describe = int32(int32:self) {
        pass self + 100;
    };
};
```

For struct types:

```aria
trait:Measurable = {
    func:area = int32(Rect2D:self);
};

impl:Measurable:for:Rect2D = {
    func:area = int32(Rect2D:self) {
        pass self.w * self.h;
    };
};
```

## As Generic Constraints

```aria
func<T: Addable>:apply_trait = int32(T:val) {
    pass val;
};
```

## Dynamic Dispatch — `dyn`

```aria
func:process = int32(dyn Describable:item) {
    pass item.describe();
};
```

## Borrow Receivers — `$$i` and `$$m`

Trait method parameters can use borrow qualifiers to express ownership semantics:

| Qualifier | Meaning | Aliasing Rule |
|-----------|---------|---------------|
| `$$i` | Immutable borrow (shared) | Multiple `$$i` borrows allowed simultaneously |
| `$$m` | Mutable borrow (exclusive) | Only one `$$m` borrow at a time, no concurrent `$$i` |

### Syntax

Borrow qualifiers appear before the type in trait method parameter lists:

```aria
trait:Transformable = {
    func:scale = int32($$i Transformable:self, int32:factor);
    func:mutate = int32($$m Transformable:self, int32:value);
};
```

- `$$i Transformable:self` — borrows `self` immutably (read-only access)
- `$$m Transformable:self` — borrows `self` mutably (read-write access)

### Implementation

```aria
impl:Transformable:for:Rect2D = {
    func:scale = int32($$i Rect2D:self, int32:factor) {
        // self is immutable — cannot modify fields
        pass self.w * factor;
    };

    func:mutate = int32($$m Rect2D:self, int32:value) {
        // self is mutable — can modify fields
        pass value;
    };
};
```

### When To Use Each

| Use `$$i` when | Use `$$m` when |
|----------------|----------------|
| Method only reads data | Method modifies the receiver |
| Want to allow shared access | Need exclusive access |
| Pure computation / getters | Setters / state mutation |

### Interaction with the Borrow Checker

The compiler enforces borrow rules at compile time:

```aria
// OK: multiple immutable borrows
$$i Rect2D:ref1 = ...;
$$i Rect2D:ref2 = ...;  // fine, both are $$i

// ERROR: mutable + immutable at same time
$$m Rect2D:ref1 = ...;
$$i Rect2D:ref2 = ...;  // compile error: cannot borrow while $$m exists
```

### The `#` Pin Operator

The pin operator `#` guarantees a variable's memory address stays stable. This
prevents the GC from relocating pinned data — essential for zero-copy FFI:

```aria
#my_struct;  // pin: address will not move
```

Pinning is orthogonal to borrowing: a pinned value can still be borrowed with
`$$i` or `$$m`, but its address is guaranteed stable for the pin's lifetime.

## Status

Traits are specified in the language design. Implementation status varies —
see the RFC at `META/ARIA/TRAITS_AND_BORROW_SEMANTICS_RFC.md`.

## Related

- [rules.md](rules.md) — refinement types (`Rules<T>`, `limit<>`)
- [functions/generics.md](../functions/generics.md) — generic functions
