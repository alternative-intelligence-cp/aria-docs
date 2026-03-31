# Inline and Noinline Function Hints

**Category**: Advanced Features → Optimization Hints  
**Added**: v0.2.12  
**Purpose**: Control function inlining behavior at the LLVM level

---

## Overview

Aria provides two function modifiers that control how LLVM handles inlining:

| Modifier | Effect | LLVM Attributes |
|---|---|---|
| `inline func:` | Force inlining at all call sites | `InlineHint` + `AlwaysInline` |
| `noinline func:` | Prevent inlining | `NoInline` |

Without either modifier, LLVM uses its default inlining heuristics.

---

## `inline func:`

Marks a function for aggressive inlining. The function body is substituted at every call site.

```aria
inline func:add = int32(int32:a, int32:b) {
    pass(a + b);
};

func:main = int32() {
    int32:result = add(3, 4);
    // The call to add() is replaced with the body: 3 + 4
    pass(result);
};

failsafe {
    pass(1);
};
```

### When to Use `inline`

- **Small, hot functions** called in tight loops
- **Wrapper functions** that add minimal logic around another call
- **Mathematical helpers** (abs, min, max, clamp)

### When NOT to Use `inline`

- **Large functions** — inlining them increases binary size (code bloat)
- **Recursive functions** — LLVM cannot fully inline recursion
- **Functions called from many sites** — each call site gets a copy

**Note**: Aria's `inline` is **not** a hint — it sets LLVM's `AlwaysInline` attribute, meaning the function **will** be inlined wherever possible.

---

## `noinline func:`

Prevents LLVM from inlining a function, even if its heuristics suggest inlining would be beneficial.

```aria
noinline func:heavy_computation = flt64(flt64:input) {
    // Complex computation that should stay as a call
    flt64:result = input * input * input;
    result = result + (input * 2.0);
    pass(result);
};

func:main = int32() {
    flt64:val = heavy_computation(3.14);
    pass(0);
};

failsafe {
    pass(1);
};
```

### When to Use `noinline`

- **Cold paths** — error handlers, logging, diagnostics
- **Large functions** that you want to keep as single copies
- **Debugging** — preventing inlining makes stack traces clearer
- **Binary size control** — keeping hot paths small in instruction cache

---

## Combining with `pub`

Both modifiers work with `pub` visibility:

```aria
pub inline func:fast_abs = int32(int32:x) {
    if(x < 0) {
        pass(0 - x);
    }
    pass(x);
};

pub noinline func:cold_error_log = NIL(string:msg) {
    stderr_write("ERROR: " + msg + "\n");
};
```

The modifier goes **before** `func:`:
- `inline func:name` — correct
- `pub inline func:name` — correct
- `pub noinline func:name` — correct

---

## Combining with `comptime`

`comptime` and `inline`/`noinline` serve different purposes and should not be combined on the same function:

- `comptime func:` — evaluated at compile time by the ConstEvaluator
- `inline func:` — affects runtime code generation in LLVM

---

## Related

- [Compile-Time Evaluation](comptime.md) — `comptime(expr)` and `comptime func:` for CTFE
- [Preprocessor Macros](macros.md) — `%define` / `%macro` for text-level code generation
