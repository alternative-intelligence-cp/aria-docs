# Rules & Limit — Refinement Types

## Rules<T>

Rules define compile-time and runtime constraints on types:

```aria
Rules<int32>:r_positive = {
    $ > 0
};

Rules<int32>:r_small_positive = {
    limit<r_positive>,
    $ < 100
};
```

Multi-type rules (value validated against multiple widths):

```aria
Rules<int32,int64>:r_positive_wide = {
    $ > 0
};
```

## limit<Rules>

Apply Rules as a type annotation:

```aria
func:process = NIL(limit<r_positive> int32:count) {
    // 'count' is guaranteed > 0
    pass NIL;
}
```

## Member Access in Rules — $.field

```aria
Rules<Person>:r_adult = {
    $.age >= 18
};
```

## Array Rules

```aria
Rules<int32[]>:arr_first_positive = { $[0] > 0 };
Rules<int32[]>:arr_min_length = { $.length >= 4 };
```

## Notes

- `$` is the value being checked
- `$.field` accesses struct members
- `$[n]` accesses array elements
- Rules are file-scoped — not importable via `use`

## Related

- [traits.md](traits.md) — behavioral constraints
- [functions/design_by_contract.md](../functions/design_by_contract.md) — requires/ensures
