# `extern` Syntax

**Category**: Modules → FFI  
**Purpose**: Detailed extern declaration syntax

---

## Overview

`extern` has several syntax forms for interfacing with foreign code.

---

## Basic Syntax

```aria
extern "<abi>" {
    fn <function_name>(<params>) -> <return_type>;
}
```

---

## ABI Specifications

### C ABI

```aria
extern "C" {
    func:function_name = NIL();
}
```

### System ABI

```aria
extern "system" {
    func:system_function = NIL();
}
```

### Platform-Specific

```aria
extern "stdcall" {  // Windows
    func:windows_function = NIL();
}

extern "fastcall" {  // Windows
    func:fast_function = NIL();
}
```

---

## Function Declarations

### Simple Function

```aria
extern "C" {
    func:simple_func = i32;()
}
```

### With Parameters

```aria
extern "C" {
    func:add = i32;(int32:a, int32:b)
    func:multiply = f64;(flt64:x, flt64:y)
}
```

### With Pointers

```aria
extern "C" {
    func:process_data = i32;(*void:data, uint64:size)
    func:get_string = *u8;()
}
```

---

## Variadic Functions

```aria
extern "C" {
    func:printf = i32;(*u8:format, ...)
    func:sprintf = i32;(*u8:buffer, *u8:format, ...)
}
```

---

## Multiple Declarations

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:realloc = *void;(*void:ptr, uint64:new_size)
    func:calloc = *void;(uint64:num, uint64:size)
}
```

---

## Linking Attributes

```aria
#[link(name = "m")]
extern "C" {
    func:sin = f64;(flt64:x)
    func:cos = f64;(flt64:x)
}

#[link(name = "crypto")]
extern "C" {
    func:encrypt = i32;(*void:data)
    func:decrypt = i32;(*void:data)
}
```

---

## Static Variables

```aria
extern "C" {
    static errno: i32;
    static stdout: *void;
}
```

---

## Exporting Aria Functions

### Basic Export

```aria
#[no_mangle]
pub extern "C" fn aria_function(x: i32) -> i32 {
    pass(x * 2);
}
```

### With Attributes

```aria
#[no_mangle]
#[export_name = "custom_name"]
pub extern "C" fn aria_func() -> i32 {
    pass(42);
}
```

---

## Type Mapping

### Integers

```aria
extern "C" {
    func:int8_func = i8;(int8:x)
    func:int16_func = i16;(int16:x)
    func:int32_func = i32;(int32:x)
    func:int64_func = i64;(int64:x)
}
```

### Floats

```aria
extern "C" {
    func:float_func = f32;(flt32:x)
    func:double_func = f64;(flt64:x)
}
```

### Pointers

```aria
extern "C" {
    func:void_ptr_func = *void;(*void:ptr)
    func:u8_ptr_func = *u8;(*u8:ptr)
    func:const_ptr_func = *const(*const void:ptr)void;
    func:mut_ptr_func = *mut(*mut void:ptr)void;
}
```

---

## Platform-Specific Syntax

### Conditional Extern

```aria
#[cfg(target_os = "linux")]
extern "C" {
    func:linux_specific = NIL();
}

#[cfg(target_os = "windows")]
extern "system" {
    func:windows_specific = NIL();
}

#[cfg(target_os = "macos")]
extern "C" {
    func:macos_specific = NIL();
}
```

---

## Common Patterns

### C Standard Library

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:memcpy = *void;(*void:dest, *void:src, uint64:n)
    func:strlen = usize;(*u8:s)
    func:strcmp = i32;(*u8:s1, *u8:s2)
    func:printf = i32;(*u8:format, ...)
}
```

### POSIX Functions

```aria
#[link(name = "c")]
extern "C" {
    func:open = i32;(*u8:path, int32:flags)
    func:close = i32;(int32:fd)
    func:read = isize;(int32:fd, *void:buf, uint64:count)
    func:write = isize;(int32:fd, *void:buf, uint64:count)
}
```

### Math Library

```aria
#[link(name = "m")]
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:pow = f64;(flt64:base, flt64:exp)
    func:sin = f64;(flt64:x)
    func:cos = f64;(flt64:x)
    func:log = f64;(flt64:x)
}
```

---

## Best Practices

### ✅ DO: Specify ABI Explicitly

```aria
extern "C" {  // ✅ Clear ABI
    func:c_function = NIL();
}
```

### ✅ DO: Document Foreign Functions

```aria
// Allocates memory using C malloc
// Caller must free using free()
extern "C" {
    func:malloc = *void;(uint64:size)
}
```

### ✅ DO: Group Related Functions

```aria
// String functions
extern "C" {
    func:strlen = usize;(*u8:s)
    func:strcmp = i32;(*u8:s1, *u8:s2)
    func:strcpy = *u8;(*u8:dest, *u8:src)
}

// Memory functions
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:memcpy = *void;(*void:dest, *void:src, uint64:n)
}
```

### ⚠️ WARNING: No Safety Checks

```aria
extern "C" {
    func:dangerous = NIL(*void:ptr);  // No null checks!
}
```

---

## Syntax Reference

| Syntax | Purpose |
|--------|---------|
| `extern "C" { }` | C calling convention |
| `extern "system" { }` | System calling convention |
| `#[link(name = "lib")]` | Link external library |
| `#[no_mangle]` | Prevent name mangling |
| `pub extern "C" fn` | Export function to C |
| `static var: type;` | Extern static variable |

---

## Related

- [extern](extern.md) - Extern keyword overview
- [extern_blocks](extern_blocks.md) - Extern blocks
- [extern_functions](extern_functions.md) - External functions
- [ffi](ffi.md) - FFI overview

---

**Remember**: Get the **ABI** and **signatures** exactly right!
