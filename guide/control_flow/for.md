# for Loop

## C-Style For

```aria
for (int32:i = 0; i < 10; i++) {
    println(`&{i}`);
}
```

Standard three-part syntax: init; condition; update.

## Notes

- No trailing semicolon after closing brace
- For simple iteration with a counter, prefer `loop()` or `till()`
- `for` is best for complex iteration patterns

## Related

- [loop_till.md](loop_till.md) — loop/till with automatic `$` iterator
- [while.md](while.md) — condition-only loops
