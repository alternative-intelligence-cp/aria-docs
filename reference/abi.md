# Nitpick C ABI Reference

This document describes the calling conventions and data layout rules that apply
when Nitpick code calls C functions (via `extern` blocks) or is called from C.

---

## Overview

Nitpick compiles to LLVM IR and emits code with standard C ABI linkage on all
supported targets. Any language that can call C functions can call Nitpick
exports. The rules below document the places where Nitpick's type system does
**not** map one-to-one to C types.

---

## Known ABI Edge Cases

### `flt32` extern parameters pass as `double`

**Rule:** `flt32` parameters in `extern` function declarations are passed to C
as `double` (IEEE 754 binary64), not as `float` (binary32).

```nitpick
extern "C" func:my_c_func = int32(flt32:x);
// x is passed to C as a double, not a float
```

**Rationale:** LLVM promotes `float` arguments to `double` at the C ABI
boundary by default. Nitpick inherits this behavior.

**Workaround:** Declare the C-side parameter as `double` and convert internally,
or use `flt64` in the Nitpick extern signature.

**Severity:** Low — only affects extern functions receiving `flt32` params from
Nitpick. Return values and local variables are unaffected.

---

### `extern` functions returning `string` must use `AriaString` by value

**Rule:** When a C function is declared to return `string` in an `extern` block,
the C implementation must return an `AriaString` struct **by value**:

```c
// C side — correct
typedef struct { const char* data; int64_t length; } AriaString;

AriaString my_function(void) {
    static char buf[] = "hello";
    return (AriaString){ .data = buf, .length = 5 };
}
```

The `AriaString` struct is returned with `data` in `RAX` and `length` in `RDX`
(System V AMD64 ABI for structs that fit in two registers).

**Common mistake:** Returning `AriaString*` (a pointer) instead of `AriaString`
by value. This sets only `RAX` to the struct pointer; Nitpick reads `RAX` as
the `data` field (a pointer to a struct, not a pointer to chars), which produces
corrupt output or a crash.

```c
// WRONG — do not do this
AriaString* my_function(void) {
    static AriaString result = { "hello", 5 };
    return &result;  // only RAX is set; RDX is undefined
}
```

**Severity:** High — silent data corruption or crash.

---

### `extern` pointer return wrapped in `{ i1, ptr }` result struct

**Rule:** Nitpick wraps pointer returns from `extern` functions in an internal
`{ i1, ptr }` result struct to carry a null-or-valid flag alongside the
pointer. When writing C helpers that return pointers consumed by Nitpick,
follow the standard C return convention; the wrapping is performed by the
Nitpick compiler, not the C function.

**Implication:** A C function returning `void*` or `T*` should simply return
the pointer. Do not attempt to return a `{ bool, ptr }` struct from C — the
compiler inserts the wrapping layer automatically.

**Severity:** Low — only affects interop where the caller inspects the LLVM IR
directly. Well-formed C functions are unaffected.

---

## Summary Table

| Nitpick type | C type at ABI boundary | Notes |
|---|---|---|
| `int8` | `int8_t` | Direct mapping |
| `int16` | `int16_t` | Direct mapping |
| `int32` | `int32_t` | Direct mapping |
| `int64` | `int64_t` | Direct mapping |
| `uint8`–`uint64` | `uint8_t`–`uint64_t` | Direct mapping |
| `flt32` | **`double`** | Promoted to 64-bit at call site |
| `flt64` | `double` | Direct mapping |
| `bool` | `bool` / `_Bool` | Direct mapping |
| `string` (param) | `const char*` | Compiler extracts `.data` field |
| `string` (return) | `AriaString` by value | `{data, length}` in RAX/RDX |
| `T->` (pointer) | `T*` | Direct mapping |
| `void->` | `void*` | Direct mapping |

---

## See Also

- [CROSS_LANGUAGE_BINDINGS.md](CROSS_LANGUAGE_BINDINGS.md) — Full interop guide
- [FUNCTIONS.md](FUNCTIONS.md) — Extern function syntax
- `REPOS/aria/KNOWN_ISSUES.md` — Open ABI limitations
