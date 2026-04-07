# loop and till — Counted Iteration

## loop(start, limit, step)

Counted loop with automatic iteration variable `$`:

```aria
loop(0, 10, 1) {
    println(`&{$}`);       // prints 0, 1, 2, ..., 9
}

loop(0, 100, 2) {
    println(`&{$}`);       // prints 0, 2, 4, ..., 98
}
```

- `$` is the automatic iteration variable — no manual declaration needed
- `start` is the initial value
- `limit` is the exclusive upper bound
- `step` is the increment per iteration

## till(limit, step)

Shorthand when starting from 0:

```aria
till(10, 1) {
    println(`&{$}`);       // prints 0, 1, 2, ..., 9
}
```

Equivalent to `loop(0, limit, step)`.

## Using $ in Expressions

```aria
int32:sum = 0;
loop(1, 101, 1) {
    sum = (sum + $);   // sum of 1 to 100
}
println(`&{sum}`);          // 5050
```

## Notes

- No trailing semicolon after closing brace
- `$` is a **reserved symbol** — cannot use for other purposes inside loop body
- For complex iteration, use `for` instead

## Related

- [for.md](for.md) — C-style for loops
- [while.md](while.md) — condition-only loops
