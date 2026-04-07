# Extern — FFI with C

## Overview

`extern` blocks declare C functions callable from Aria. Extern functions do **not**
return `Result<T>` — they return raw values.

## Syntax

```aria
extern "libm" {
    func:sin = double(double:x);
    func:cos = double(double:x);
    func:sqrt = double(double:x);
}
```

Or flat (no library block):

```aria
extern func:custom_func = int32(int32:a, int32:b);
```

## Rules

1. **Extern blocks must be at file scope** — not inside functions
2. `extern "lib" { }` blocks have a limit of **≤7** declarations
3. Flat `extern func:` declarations have no per-file limit
4. Use `void` for no-return (not NIL): `func:exit = void(int32:code);`
5. Use `const` (not `fixed`) inside extern blocks

## String ABI

| Direction | Aria Type | C Type |
|-----------|-----------|--------|
| Parameter | `string` | `const char*` |
| Return | `string` | `AriaString {char* data, int64_t length}` |

C shims for string-returning functions must return the AriaString struct.

## Float ABI

Aria's `flt32` passes as `double` at the C ABI level. C shims must use `double` params:

```c
// C side
double my_func(double x) {    // NOT float!
    return (float)x * 2.0f;
}
```

## Pointer Types in Extern

Pointer types in extern blocks may create `{i1, ptr}` optional wrappers that corrupt
struct fields. Workaround: use `int64` for handle/pointer types (ABI-compatible on x86-64).

## Example

```aria
extern "libc" {
    func:printf = int32(string:fmt);
    func:malloc = int64(int64:size);    // use int64 instead of ptr
}

func:main = int32() {
    drop printf("Hello from C!\n");
    exit 0;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

## Related

- [modules/use_import.md](../modules/use_import.md) — importing Aria modules
- [types/string.md](../types/string.md) — string ABI details
