# Compile-Time Evaluation (`comptime`)

**Category**: Advanced Features → Compile-Time  
**Added**: v0.2.12  
**Purpose**: Force evaluation of expressions, blocks, or functions at compile time

---

## Overview

`comptime` instructs the compiler to evaluate code during compilation rather than at runtime. The result is embedded as a constant in the generated binary. Aria provides three forms of compile-time evaluation.

---

## Form 1: Comptime Expression

Evaluate a single expression at compile time. The result must be a constant (integer, float, bool, or string).

```aria
func:main = int32() {
    int32:size = comptime(4 * 1024);
    flt64:pi_squared = comptime(3.14159265358979 * 3.14159265358979);
    bool:flag = comptime(8 > 4);

    pass(size);
};

failsafe {
    pass(1);
};
```

The inner expression is fully evaluated by the type checker's `ConstEvaluator`. The IR generator emits the pre-computed result as an LLVM constant.

---

## Form 2: Comptime Block

Evaluate an entire block at compile time. Useful for complex computations with multiple steps.

```aria
comptime {
    int32:table_size = 256;
    int32:mask = table_size - 1;
};

func:main = int32() {
    pass(0);
};

failsafe {
    pass(1);
};
```

The block executes during type checking. At IR generation time, comptime blocks emit **no code** — all effects are resolved at compile time.

**Note**: The closing `};` (semicolon after brace) is required for comptime blocks.

---

## Form 3: Comptime Function

Mark a function for compile-time function evaluation (CTFE). The function is registered in the `ConstEvaluator` and can be called inside `comptime(...)` expressions.

```aria
comptime func:factorial = int64(int64:n) {
    if(n <= 1i64) {
        pass(1i64);
    }
    pass(n * factorial(n - 1i64));
};

func:main = int32() {
    int64:result = comptime(factorial(10i64));
    // result == 3628800 — computed at compile time
    pass(0);
};

failsafe {
    pass(1);
};
```

Comptime functions:
- Are registered in the symbol table for CTFE calls
- Are **also** emitted as regular IR (they can be called at runtime too)
- Can be combined with `pub`: `pub comptime func:name = ...`
- Must be deterministic — no I/O, no runtime-only operations

---

## ConstEvaluator

The compile-time interpreter supports:

| Value Kind | Description |
|---|---|
| Integer | All Aria integer types (int8 through int4096) |
| Unsigned | All unsigned types (uint8 through uint64) |
| TBB | Ternary Balanced Binary with sticky error propagation |
| Float | flt32, flt64, flt128 through flt512 |
| Bool | `true` / `false` |
| String | String constants |
| Pointer, Struct, Array | Complex types via virtual heap |

### Resource Limits

The evaluator enforces sandbox limits:
- **1,000,000 instructions** maximum per evaluation
- **512 stack frames** maximum depth
- **1 GB virtual heap** maximum allocation

---

## Supported Operations in Comptime

- Arithmetic: `+`, `-`, `*`, `/`, `%`
- Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
- Logical: `&&`, `||`, `!`
- Bitwise: `&`, `|`, `^`, `<<`, `>>`
- Function calls (to comptime-registered functions)
- Conditional expressions
- TBB arithmetic with error propagation

---

## What Cannot Be Used in Comptime

- I/O operations (`print()`, file operations)
- Memory allocation (`alloc`)
- Thread operations
- Extern/FFI calls
- Anything that requires runtime state

---

## Error Handling

If a comptime expression cannot be evaluated, the compiler reports:

```
comptime evaluation failed: <reason>
```

Common causes:
- Referencing a runtime variable inside `comptime(...)`
- Calling a non-comptime function
- Exceeding resource limits (instruction count, stack depth)
- Division by zero or other undefined operations

---

## Related

- [Preprocessor Macros](macros.md) — Text-level code generation (`%define`, `%macro`)
- [Inline Functions](inline.md) — `inline func:` / `noinline func:` hints