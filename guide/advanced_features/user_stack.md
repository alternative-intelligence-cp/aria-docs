# User Stack

**Category**: Advanced Features → Data Structures  
**Syntax**: `astack()`, `apush()`, `apop()`, `apeek()`  
**Purpose**: Per-scope implicit typed LIFO scratch pad  
**Since**: v0.4.3

---

## Overview

The **user stack** is a compiler-managed typed LIFO (Last In, First Out) scratch pad for passing values between computation steps within a function. It provides type-safe value juggling with zero ceremony — the compiler manages handles, type tags, and cleanup automatically.

The user stack is **not** the hardware call stack and is **not** related to the `stack` memory allocation keyword.

---

## Philosophy

> "There are no solutions, only tradeoffs."

Aria provides multiple ways to move values around:

| Approach | Speed | Safety | Ergonomics |
|----------|-------|--------|------------|
| Plain variables | Fastest | Manual | Good for fixed shapes |
| User stack (builtin) | Moderate | Compiler-enforced types | Best — zero ceremony |
| Library stack (extern C) | Moderate | Manual tags | Verbose — handles, tags, cleanup |

The user stack exists for **ergonomics and safety**, not raw performance. When nanoseconds matter in a hot loop, use plain variables. When you need type-safe dynamic-depth value passing with no boilerplate, use the user stack.

---

## Getting Started

### Initialize

```aria
func:main = int32() {
    astack(64i64);     // 64-slot stack
    // or
    astack();          // 256-slot default

    // ... use apush/apop/apeek here ...

    return 0;
};
```

Call `astack()` once at the top of any function that needs a user stack. Only one user stack per function scope.

### Push Values

```aria
apush(42i64);          // push an int64
apush(3.14f64);        // push a flt64
apush(true);           // push a bool
apush(7i32);           // push an int32
```

Type tags are stored automatically — no manual tag management needed.

### Pop Values

```aria
int32:small = apop();  // compiler infers: pop as int32
bool:flag = apop();    // compiler infers: pop as bool
flt64:pi = apop();     // compiler infers: pop as flt64
int64:big = apop();    // compiler infers: pop as int64
```

The destination type in the assignment tells the compiler what type to expect. If the top-of-stack tag doesn't match, the program exits with a diagnostic.

### Peek (Non-Destructive)

```aria
apush(99i64);
int64:top = apeek();   // reads 99 without removing it
int64:also = apop();   // now removes it — also 99
```

---

## LIFO Ordering

Values come off in reverse order:

```aria
astack();

apush(1i64);    // bottom
apush(2i64);    // middle
apush(3i64);    // top

int64:c = apop();   // 3 (top)
int64:b = apop();   // 2
int64:a = apop();   // 1 (bottom)
```

---

## Mixed Types

The user stack supports pushing different types in the same stack:

```aria
astack(16i64);

apush(100i32);       // int32
apush(2.718f32);     // flt32
apush(999i64);       // int64

int64:big = apop();      // 999
flt32:e = apop();        // 2.718
int32:small = apop();    // 100
```

Each slot stores its own type tag. Pop/peek validate the tag against the assignment target.

---

## Error Handling

The user stack uses a **fatal error model** — misuse is a programming error, not a runtime condition:

| Error | Behavior |
|-------|----------|
| Push to full stack | Diagnostic to stderr + `exit(1)` |
| Pop from empty stack | Diagnostic to stderr + `exit(1)` |
| Pop type mismatch | Diagnostic to stderr + `exit(1)` |
| Peek from empty stack | Diagnostic to stderr + `exit(1)` |
| Peek type mismatch | Diagnostic to stderr + `exit(1)` |

User stack operations do **not** return `Result<T>`. They return values directly (pop/peek) or nothing (push).

---

## Auto-Cleanup

The user stack is automatically destroyed when the function returns — on both explicit `return` statements and fallthrough. No `defer` or manual cleanup needed.

```aria
func:example = NIL() {
    astack(32i64);
    apush(42i64);
    // stack is automatically cleaned up here
    pass(NIL);
};
```

---

## Practical Example

### Computation accumulator

```aria
func:compute_sum = int64() {
    astack();
    int64:i = 0i64;
    int64:sum = 0i64;

    while (i < 1000i64) {
        apush(i * 3i64 + 1i64);
        apush(i * 7i64 + 2i64);

        int64:b = apop();
        int64:a = apop();
        sum = sum + a + b;

        i = i + 1i64;
    }

    pass(sum);
};
```

---

## Supported Types

| Tag | Type    | Storage |
|-----|---------|---------|
| 0   | int8    | Widened to int64 |
| 1   | int16   | Widened to int64 |
| 2   | int32   | Widened to int64 |
| 3   | int64   | Direct |
| 4   | flt32   | Bitcast to int64 |
| 5   | flt64   | Bitcast to int64 |
| 6   | bool    | Widened to int64 |
| 7   | string  | Pointer as int64 |
| 8   | pointer | Pointer as int64 |

All values are stored in uniform 16-byte slots (8 bytes value + 8 bytes type tag). Float values use bitcast to preserve exact IEEE 754 representation.

---

## What the User Stack Is NOT

- **Not the call stack** — The hardware call stack manages function frames. The user stack is a separate data structure.
- **Not the `stack` keyword** — `stack int32:x;` is a memory allocation mode. Unrelated.
- **Not cross-function** — The user stack lives within one function scope. Use parameters and return values for cross-function data.
- **Not thread-safe** — Use channels or atomics for cross-thread communication.
- **Not a replacement for variables** — For known, fixed-shape data, plain variables are faster and simpler.

---

## Performance Notes

Benchmarked at 10M iterations (push two values, pop two values, accumulate):

- **Plain variables**: ~20ms baseline
- **Extern C library stack**: ~90ms (~4x baseline)
- **Builtin user stack**: ~185ms (~8-9x baseline)

The overhead comes from type tag storage/checking and mmap-backed memory. For hot inner loops, prefer plain variables. The user stack's value is safety and ergonomics, not speed.

---

## See Also

- [Memory Model → Stack](../memory_model/stack.md) — Hardware call stack
- [Memory Model → Allocation](../memory_model/allocation.md) — Memory allocation modes
- [Advanced Features → Channels](channels.md) — Cross-thread communication
