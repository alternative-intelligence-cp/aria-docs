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

## The `!!!` Operator — Failsafe Shorthand

The `!!!` operator immediately invokes `failsafe` with an error code. It is syntactic sugar:

```
!!! errcode;    →    failsafe(errcode);
```

### Usage

```aria
func:validate = int32(int32:input) {
    if (input < 0) {
        !!! 42;     // calls failsafe(42) — program exits
    }
    pass input;
};

func:main = int32() {
    int32:val = raw validate(10);
    exit 0;
};

func:failsafe = int32(tbb32:err) {
    // err == 42 if triggered by !!! above
    exit err => int32;
};
```

### When To Use

| Mechanism | When | Recovery |
|-----------|------|----------|
| `fail errCode` | Recoverable error in a function | Caller handles Result |
| `?!` (emphatic unwrap) | Unwrap Result, failsafe if error | failsafe receives error |
| `!!! errCode` | Unrecoverable error, bail immediately | failsafe receives errCode |

### Interaction with `defer`

`defer` blocks still execute before `!!!` transfers control to failsafe:

```aria
func:cleanup_example = NIL() {
    defer {
        // This runs BEFORE failsafe is called
    };

    !!! 1;   // defer fires, then failsafe(1)
    pass NIL;
};
```

## Related

- [types/tbb.md](../types/tbb.md) — TBB sticky error propagation
- [types/result.md](../types/result.md) — Result<T> system
- [functions/main_failsafe.md](../functions/main_failsafe.md) — failsafe function
- [operators/result_operators.md](../operators/result_operators.md) — error handling operators
