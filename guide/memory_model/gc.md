# Garbage Collected Allocation

## Declaration

```aria
gc string:shared_data = "persistent";
gc []int32:heap_array = [1, 2, 3, 4, 5];
```

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
