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

## Status

Traits are specified in the language design. Implementation status varies —
see the RFC at `META/ARIA/TRAITS_AND_BORROW_SEMANTICS_RFC.md`.

## Related

- [rules.md](rules.md) — refinement types (`Rules<T>`, `limit<>`)
- [functions/generics.md](../functions/generics.md) — generic functions
