# Error Flow

## Layered Error Model

Aria has a three-tier error system:

| Tier | Mechanism | Scope | Recovery |
|------|-----------|-------|----------|
| Value-level | TBB ERR sentinel | Arithmetic | Check `== ERR` |
| Function-level | Result<T> pass/fail | Function calls | `?`, `?!`, `?|`, `drop`, `raw` |
| Program-level | failsafe | Fatal errors | Must `exit(code > 0)` |

## Error Propagation Flow

```
TBB ERR (value) → Result<T> fail (function) → failsafe (program)
```

1. **TBB arithmetic** — overflow, div-by-zero → ERR sentinel, propagates through pipeline
2. **Function calls** — `fail errcode` → caller must handle Result
3. **Unrecoverable** — `?!` failure, `!!! err` → calls failsafe, program exits

## The `ok` Keyword

Allows passing a variable that may have value `unknown` down the call chain:

```aria
ok maybe_unknown_value;
```

## Related

- [types/tbb.md](../types/tbb.md) — TBB sticky error propagation
- [types/result.md](../types/result.md) — Result<T> system
- [functions/main_failsafe.md](../functions/main_failsafe.md) — failsafe function
- [operators/result_operators.md](../operators/result_operators.md) — error handling operators
