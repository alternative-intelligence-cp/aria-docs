# break and continue

## break

Exit the innermost loop immediately:

```aria
loop(0, 100, 1) {
    if ($ == 42) {
        break;
    }
}
```

## continue

Skip to the next iteration:

```aria
loop(0, 10, 1) {
    if ($ == 5) {
        continue;    // skip 5
    }
    println(`&{$}`);
}
```

## Works With

- `for`
- `while`
- `loop` / `till`
- `when/then/end`
