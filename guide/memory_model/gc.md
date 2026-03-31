# Garbage Collected Allocation

## Declaration

GC allocation is **implicit** — variables that are not marked `wild` or `stack` are
garbage collected:

```aria
Point:p1;
p1.x = 10i32;
p1.y = 20i32;
```

The `gc` keyword exists in the lexer but is not used as a prefix in practice.
Absence of `wild` = GC-managed.

## When to Use

- Data that must outlive the creating function's scope
- Shared references across multiple parts of the program
- Complex object graphs with cycles

## Notes

`gc` is a contextual keyword — it can be used as a variable name outside allocation contexts.

## Related

- [overview.md](overview.md) — all allocation modes
- [stack.md](stack.md) — stack allocation (prefer when possible)
- [wild.md](wild.md) — unmanaged memory
