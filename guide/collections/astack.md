# User Stack — astack

## Overview

Per-scope implicit LIFO (last-in, first-out) scratch pad. Stores heterogeneous typed
values. Auto-cleanup on scope exit. Fatal error model (exit(1) on error, not Result<T>).

## Builtins

| Builtin | Signature | Description |
|---------|-----------|-------------|
| `astack(cap)` | `(int64?) → int64` | Initialize stack with byte budget (default 256). Returns handle. |
| `apush(h, val)` | `(int64, T) → void` | Push typed value. Fatal on overflow. |
| `apop(h)` | `(int64) → T` | Pop top value. Type inferred from assignment. Fatal on mismatch/underflow. |
| `apeek(h)` | `(int64) → T` | Non-destructive read of top value. Same typed semantics as apop. |
| `acap(h)` | `(int64) → int64` | Total capacity in bytes. |
| `asize(h)` | `(int64) → int64` | Bytes currently used. |
| `afits(h, val)` | `(int64, T) → bool` | True if val fits without overflow. |
| `atype(h)` | `(int64) → int32` | Type tag of top element. Useful to avoid mismatch. |

## Example

```aria
func:main = int32() {
    int64:stk = astack 512;            // 512-byte stack (keyword form)
    // int64:stk = astack(512);        // paren form also works

    apush stk, 42;                      // push int32
    apush stk, "hello";                 // push string
    apush(stk, 3.14);                   // paren form

    string:s = apop stk;                // pop string: "hello" (LIFO)
    // string:s = apop(stk);            // paren form

    int32:n = apop(stk);                // pop int32: 42
    println(n);

    exit 0;
}

func:failsafe = int32(tbb32:err) { exit 1; }
```

## Syntax Forms (v0.4.6+)

All astack builtins support both keyword and paren syntax:

```aria
// Statement-level (apush, astack)
apush stk, value;       // keyword
apush(stk, value);      // paren
astack 256;             // keyword
astack(256);            // paren

// Expression-level (apop, apeek, acap, asize, atype)
int32:v = apop stk;     // keyword
int32:v = apop(stk);    // paren

// Expression-level with argument (afits)
bool:ok = afits stk, value;   // keyword
bool:ok = afits(stk, value);  // paren
```

## Error Behavior

Errors are **fatal** (not Result<T>):
- Stack overflow → `exit(1)`
- Stack underflow → `exit(1)`
- Type mismatch on pop/peek → `exit(1)`

Check before popping:
```aria
if (asize(stk) > 0) {
    int32:val = apop(stk);
}
```

## Auto-Cleanup

The stack is automatically cleaned up when the scope exits (function return). No manual
deallocation needed.

## Related

- [ahash.md](ahash.md) — hash tables
