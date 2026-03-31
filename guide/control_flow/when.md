# when / then / end — State-Tracked Loop

## Syntax

```aria
when (condition) {
    // body — executes while condition is true
} then {
    // runs when loop completed normally (condition became false)
} end {
    // runs when condition was initially false, or break was used
}
```

## Example

```aria
int32:i = 0i32;
int32:result = 0i32;
when (i < 3i32) {
    i = i + 1i32;
} then {
    result = 1i32;     // loop completed normally
} end {
    result = 2i32;     // condition was initially false or break
}
```

## vs while

`when/then/end` is semantically similar to `while` but also tracks *how* the loop
terminated. `then` runs on normal completion (condition became false). `end` runs if
the condition was initially false or `break` was used. Use `while` for simple loops,
`when` for state-driven loops where termination reason matters.

## Notes

- The loop body goes in the `when (cond) { }` block
- `then { }` is the normal-completion handler
- `end { }` is the false-or-break handler

## Related

- [while.md](while.md) — simple condition loops
- [loop_till.md](loop_till.md) — counted iteration
