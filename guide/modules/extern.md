# `extern` Keyword

**Category**: Modules → FFI  
**Syntax**: `extern "<abi>" { ... }`  
**Purpose**: Interface with foreign code (C, system libraries)

---

## Overview

`extern` declares **foreign functions** and **external interfaces**.

---

## Basic Extern

```aria
extern "C" {
    func:printf = i32;(*u8:format, ...)
}

// Use it
extern.printf("Hello from C!\n");
```

---

## Extern Block

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:strlen = usize;(*u8:s)
}
```

---

## ABI (Application Binary Interface)

```aria
extern "C" { }      // C calling convention
extern "system" { } // System calling convention
extern "stdcall" { } // Windows stdcall
```

Most common: `"C"`

---

## Calling C Functions

```aria
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:abs = i32;(int32:x)
}

Result: f64 = extern.sqrt(16.0);  // 4.0
value: i32 = extern.abs(-5);      // 5
```

---

## Linking Libraries

```aria
#[link(name = "m")]  // Link against libm (math library)
extern "C" {
    func:sin = f64;(flt64:x)
    func:cos = f64;(flt64:x)
}
```

---

## Common C Functions

### Standard C Library

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:memcpy = *void;(*void:dest, *void:src, uint64:n)
    func:strlen = usize;(*u8:s)
    func:strcmp = i32;(*u8:s1, *u8:s2)
}
```

---

## Extern Functions from Aria

Make Aria functions callable from C:

```aria
#[no_mangle]  // Prevent name mangling
pub extern "C" fn aria_add(a: i32, b: i32) -> i32 {
    pass(a + b);
}
```

---

## Best Practices

### ✅ DO: Wrap Unsafe Extern Calls

```aria
extern "C" {
    func:unsafe_c_function = i32;(*void:ptr)
}

// Safe wrapper
pub func:safe_function = Result<int32>(*Data:data) {
    if data == NULL {
        fail("Null pointer");
    }
    
    Result: i32 = extern.unsafe_c_function(data);
    if result < 0 {
        fail("C function failed");
    }
    
    pass(result);
}
```

### ✅ DO: Document ABI

```aria
// Calls C standard library malloc
extern "C" {
    func:malloc = *void;(uint64:size)
}
```

### ⚠️ WARNING: Extern is Unsafe

```aria
// No safety checks - you must validate!
extern "C" {
    func:dangerous_c_function = NIL(*void:ptr);
}

// Can crash, corrupt memory, etc.
extern.dangerous_c_function(invalid_ptr);  // ⚠️ DANGER
```

---

## Related

- [extern_syntax](extern_syntax.md) - Extern syntax details
- [extern_blocks](extern_blocks.md) - Extern blocks
- [extern_functions](extern_functions.md) - External functions
- [ffi](ffi.md) - Foreign Function Interface
- [c_interop](c_interop.md) - C interoperability

---

**Remember**: `extern` is **powerful** but **unsafe** - wrap it carefully!
