# User Stack Reference

**Feature**: Per-Scope Implicit User Stack  
**Version**: v0.4.3 (March 2026)  
**Status**: Implemented, 22/22 tests passing

---

## Summary

The user stack is a compiler-managed typed LIFO scratch pad for intra-function value passing. It provides type-safe push/pop operations with automatic type tagging, context-typed pop, and scope-based auto-cleanup.

---

## API

### `astack(capacity?)`

| Property | Value |
|----------|-------|
| Arguments | 0 or 1 (`int64` capacity, default 256) |
| Returns | Nothing meaningful (implicit handle) |
| Side Effects | Allocates mmap-backed storage |
| Errors | None (allocation failure is fatal) |
| Scope | One per function scope |

### `apush(value)`

| Property | Value |
|----------|-------|
| Arguments | 1 (any supported type) |
| Returns | Nothing |
| Side Effects | Stores value + type tag at stack top |
| Errors | Fatal `exit(1)` on overflow |
| Supported Types | int8, int16, int32, int64, flt32, flt64, bool, string, pointer |

### `apop()`

| Property | Value |
|----------|-------|
| Arguments | 0 |
| Returns | Value of destination type (context-inferred) |
| Side Effects | Removes top element |
| Errors | Fatal `exit(1)` on underflow or type mismatch |
| Type Inference | From assignment target: `flt32:f = apop()` → pop as flt32 |

### `apeek()`

| Property | Value |
|----------|-------|
| Arguments | 0 |
| Returns | Value of destination type (context-inferred) |
| Side Effects | None (non-destructive) |
| Errors | Fatal `exit(1)` on underflow or type mismatch |
| Type Inference | Same as `apop()` |

---

## Type Tag Scheme

| Tag | Type    | Storage Method |
|-----|---------|----------------|
| 0   | int8    | Zero-extended to int64 |
| 1   | int16   | Zero-extended to int64 |
| 2   | int32   | Zero-extended to int64 |
| 3   | int64   | Direct |
| 4   | flt32   | Bitcast float → int32, zero-extended to int64 |
| 5   | flt64   | Bitcast double → int64 |
| 6   | bool    | Zero-extended to int64 |
| 7   | string  | Pointer stored as int64 |
| 8   | pointer | Pointer stored as int64 |

---

## Compiler Implementation

### Files Modified (from v0.4.2 → v0.4.3)

| File | Changes |
|------|---------|
| `include/runtime/ustack.h` | Push changed to void return, removed error codes, added default capacity |
| `src/runtime/collections/ustack.cpp` | Push returns void with fatal exit on overflow |
| `include/backend/ir/codegen_expr.h` | Added `ustack_pop_dest_type` context variable |
| `include/backend/ir/ir_generator.h` | Added `ustack_pop_dest_type` for propagation |
| `src/frontend/sema/type_checker.cpp` | New signatures: astack(0-1 args), apush(1), apop/apeek(0 args, UnknownType return) |
| `src/backend/ir/codegen_expr.cpp` | Per-scope implicit stack via hidden `__aria_ustack_handle` alloca; type-aware pop/peek with bitcast chain |
| `src/backend/ir/codegen_stmt.cpp` | VarDecl sets dest type before initializer; auto-cleanup on return |
| `src/backend/ir/ir_generator.cpp` | VarDecl sets dest type; CALL dispatch propagates to fresh ExprCodegen |

### Key Implementation Detail

The `ustack_pop_dest_type` context variable is the critical mechanism. When the compiler processes `int64:x = apop()`, the VarDecl handler sets `ustack_pop_dest_type = "int64"` before generating the initializer expression. The `apop()` codegen reads this context to know which type tag to validate and how to bitcast the returned value.

This required a fix in v0.4.3: `IRGenerator` creates a **fresh** `ExprCodegen` for each CALL node, so the context must be set on the `IRGenerator` and propagated to `ExprCodegen` in the CALL dispatch path.

---

## Error Diagnostics

All errors print to stderr with the format:
```
[ustack] <error description>
```

Then call `exit(1)`.

Example messages:
- `[ustack] overflow: stack full (256/256)`
- `[ustack] underflow: stack empty`
- `[ustack] type mismatch: expected tag 3 (int64), got tag 5 (flt64)`

---

## Tests

| Test File | Count | Coverage |
|-----------|-------|----------|
| `tests/test_ustack.aria` | 14 | LIFO ordering, peek, multi-push, typed pop (int8/int32/int64/flt32/flt64), mixed types |
| `tests/test_ustack_strings.aria` | 8 | Scope independence across function calls, mixed int/float types |

---

## Breaking Changes from v0.4.2

| v0.4.2 (Handle-Based) | v0.4.3 (Implicit) |
|------------------------|-------------------|
| `int64:h = astack(256i64)` | `astack(256i64)` or `astack()` |
| `apush(h, value)` | `apush(value)` |
| `apop(h)` | `apop()` — type from context |
| `apeek(h)` | `apeek()` — type from context |
| Push returns error code | Push is void (fatal on overflow) |

---

## Design Rationale

The user stack fills a specific niche in Aria's "safe by default, opt-in to danger" philosophy:

1. **Why not just use variables?** Variables are faster but require knowing the shape at compile time. The user stack supports dynamic depth and mixed types.

2. **Why not Result<T>?** Stack misuse (overflow, underflow, type mismatch) is a logic error, not a runtime condition. Fatal errors prevent silent corruption.

3. **Why implicit handles?** Explicit handles add ceremony without benefit when there's only one stack per scope. The compiler manages the handle internally.

4. **Why context-typed pop?** Requiring `apop::<int64>()` turbofish syntax would work but is verbose. Context typing from the assignment target is natural and catches mismatches at runtime.

---

## See Also

- [Spec](../specs/aria_specs.txt) — Line 6109: USER STACK section
- [Guide](../guide/advanced_features/user_stack.md) — Tutorial with examples
- [Memory Model](../guide/memory_model/user_stack.md) — Memory perspective
