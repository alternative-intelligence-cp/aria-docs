# when / then / end — State-Tracked Loop

## Syntax

```aria
when (condition) then {
    // body — executes while condition is true
} end
```

## Example

```aria
int32:retries = 3;
when (retries > 0) then {
    bool:success = raw try_connection();
    if (success) {
        retries = 0;    // exit loop
    } else {
        retries--;
    }
} end
```

## vs while

`when/then/end` is semantically similar to `while` but intended for state-machine
patterns where the loop tracks a changing condition. Use `while` for simple loops,
`when` for state-driven loops.

## Notes

- No trailing semicolon after `end`
- `end` keyword terminates the block (not just `}`)

## Related

- [while.md](while.md) — simple condition loops
- [loop_till.md](loop_till.md) — counted iteration
