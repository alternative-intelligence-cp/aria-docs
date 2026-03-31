# Debugging

## Debug Output

Use the `stddbg` stream (FD 3) for debug output that doesn't mix with stdout/stderr.

## Print Debugging

```aria
println(`DEBUG: x = &{x}, y = &{y}`);
```

## Type Inspection

- `atype(handle)` — type tag of top astack element
- `ahtype(handle, key)` — type tag of ahash entry

## Common Issues

See `META/ARIA/KNOWN_ISSUES.md` for current known issues and workarounds.

### Constant Arithmetic Bug
`fixed` with arithmetic expressions evaluates to 0. Compute in a variable:

```aria
// WRONG: fixed int64:NEG1 = 0i64 - 1i64;   // → 0
// RIGHT: int64:neg1 = (0i64 - 1i64);
```

### String Literal in sys()
Store in variable first:

```aria
// WRONG: sys(SYS_OPEN, "file.txt", 0i64, 0i64);
// RIGHT: string:path = "file.txt";
//        sys(SYS_OPEN, path, 0i64, 0i64);
```
