# while Loop

## Syntax

```aria
while (condition) {
    // body
}
```

## Example

```aria
int32:count = 0;
while (count < 10) {
    println(`&{count}`);
    count++;
}
```

## Notes

- No trailing semicolon after closing brace
- For state-tracked loops, prefer `when/then/end`
- For counted iteration, prefer `loop()` or `till()`

## Related

- [when.md](when.md) — when/then/end (state-tracked while)
- [loop_till.md](loop_till.md) — counted iteration
