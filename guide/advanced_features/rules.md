# Rules & Limit — Refinement Types

## Rules<T>

Rules define compile-time and runtime constraints on types:

```aria
Rules<int32> PositiveInt {
    $ > 0;          // $ is the checked value
}

Rules<string> NonEmpty {
    string_length($) > 0;
}
```

## limit<Rules>

Apply Rules as a type annotation:

```aria
func:process = NIL(limit<PositiveInt> int32:count) {
    // 'count' is guaranteed > 0
    pass(NIL);
}
```

## Member Access in Rules — $.field

```aria
Rules<Person> ValidPerson {
    $.age > 0;
    $.age < 200;
    string_length($.name) > 0;
}
```

## Array & Null Rules

```aria
Rules<[]int32> NonEmptyArray {
    $.length > 0;
}

Rules<int32?> NotNull {
    $ != NIL;
}
```

## Notes

- `$` is the value being checked
- `$.field` accesses struct members
- Rules are file-scoped — not importable via `use`

## Related

- [traits.md](traits.md) — behavioral constraints
- [functions/design_by_contract.md](../functions/design_by_contract.md) — requires/ensures
