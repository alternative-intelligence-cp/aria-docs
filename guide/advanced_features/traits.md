# Traits

## Overview

Traits define shared behavior that types can implement. They serve as constraints
for generic type parameters.

```aria
trait Printable {
    func:to_string = string(Self->:self);
}
```

## Implementation

```aria
impl Printable for Point {
    func:to_string = string(Point->:self) {
        pass `(&{self.x}, &{self.y})`;
    }
}
```

## As Generic Constraints

```aria
func:display = NIL(T:value) requires Printable<T> {
    println(raw value.to_string());
    pass(NIL);
}
```

## Status

Traits are specified in the language design. Implementation status varies —
see the RFC at `META/ARIA/TRAITS_AND_BORROW_SEMANTICS_RFC.md`.

## Related

- [rules.md](rules.md) — refinement types (`Rules<T>`, `limit<>`)
- [functions/generics.md](../functions/generics.md) — generic functions
