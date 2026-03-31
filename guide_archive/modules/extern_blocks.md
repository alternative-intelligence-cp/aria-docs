# Extern Blocks

**Category**: Modules → FFI  
**Purpose**: Organize foreign function declarations

---

## Overview

Extern blocks group **foreign function declarations** with a specific ABI.

---

## Basic Extern Block

```aria
extern "C" {
    func:function1 = i32;()
    func:function2 = i32;(int32:x)
    func:function3 = i32;(int32:x, int32:y)
}
```

---

## Multiple Blocks

```aria
// C standard library
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
}

// Math library
#[link(name = "m")]
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:sin = f64;(flt64:x)
}

// Custom library
#[link(name = "mylib")]
extern "C" {
    func:custom_function = i32;()
}
```

---

## Block Organization

### By Library

```aria
// libc
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:strlen = usize;(*u8:s)
}

// libm (math)
#[link(name = "m")]
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:pow = f64;(flt64:base, flt64:exp)
}

// libcrypto
#[link(name = "crypto")]
extern "C" {
    func:encrypt = i32;(*void:data)
    func:decrypt = i32;(*void:data)
}
```

---

### By Functionality

```aria
// Memory management
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:realloc = *void;(*void:ptr, uint64:size)
}

// String operations
extern "C" {
    func:strlen = usize;(*u8:s)
    func:strcmp = i32;(*u8:s1, *u8:s2)
    func:strcpy = *u8;(*u8:dest, *u8:src)
}

// File I/O
extern "C" {
    func:fopen = *void;(*u8:path, *u8:mode)
    func:fclose = i32;(*void:file)
    func:fread = usize;(*void:ptr, uint64:size, uint64:count, *void:file)
}
```

---

## Static Variables in Blocks

```aria
extern "C" {
    static errno: i32;
    static stdin: *void;
    static stdout: *void;
    static stderr: *void;
}
```

---

## Attributes on Blocks

```aria
#[link(name = "mylib")]
#[link(kind = "static")]
extern "C" {
    func:library_function = NIL();
}
```

---

## Platform-Specific Blocks

```aria
#[cfg(target_os = "linux")]
extern "C" {
    func:linux_function = NIL();
}

#[cfg(target_os = "windows")]
extern "system" {
    func:windows_function = NIL();
}

#[cfg(target_os = "macos")]
extern "C" {
    func:macos_function = NIL();
}
```

---

## Common Patterns

### C Standard Library Block

```aria
extern "C" {
    // Memory
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:calloc = *void;(uint64:num, uint64:size)
    func:realloc = *void;(*void:ptr, uint64:size)
    
    // Strings
    func:strlen = usize;(*u8:s)
    func:strcmp = i32;(*u8:s1, *u8:s2)
    func:strcpy = *u8;(*u8:dest, *u8:src)
    func:strcat = *u8;(*u8:dest, *u8:src)
    
    // I/O
    func:printf = i32;(*u8:format, ...)
    func:scanf = i32;(*u8:format, ...)
    func:puts = i32;(*u8:s)
}
```

---

### Math Library Block

```aria
#[link(name = "m")]
extern "C" {
    // Trigonometry
    func:sin = f64;(flt64:x)
    func:cos = f64;(flt64:x)
    func:tan = f64;(flt64:x)
    
    // Powers and roots
    func:sqrt = f64;(flt64:x)
    func:pow = f64;(flt64:base, flt64:exp)
    
    // Logarithms
    func:log = f64;(flt64:x)
    func:log10 = f64;(flt64:x)
    
    // Rounding
    func:floor = f64;(flt64:x)
    func:ceil = f64;(flt64:x)
    func:round = f64;(flt64:x)
}
```

---

### POSIX Block

```aria
#[link(name = "c")]
extern "C" {
    // File operations
    func:open = i32;(*u8:path, int32:flags)
    func:close = i32;(int32:fd)
    func:read = isize;(int32:fd, *void:buf, uint64:count)
    func:write = isize;(int32:fd, *void:buf, uint64:count)
    
    // Process operations
    func:fork = i32;()
    func:exec = i32;(*u8:path, **u8:argv)
    func:wait = i32;(*i32:status)
}
```

---

## Best Practices

### ✅ DO: Group Related Functions

```aria
// Group memory functions together
extern "C" {
    func:malloc = *void;(uint64:size)
    fn free(ptr: *void;
    func:realloc = *void;(*void:ptr, uint64:size)
}
```

### ✅ DO: Document Block Purpose

```aria
// SQLite3 database functions
#[link(name = "sqlite3")]
extern "C" {
    func:sqlite3_open = i32;(*u8:filename, **void:db)
    func:sqlite3_close = i32;(*void:db)
    func:sqlite3_exec = i32;(*void:db, *u8:sql)
}
```

### ✅ DO: Use Link Attributes

```aria
#[link(name = "ssl")]
#[link(name = "crypto")]
extern "C" {
    func:SSL_library_init = i32;()
    func:SSL_load_error_strings = NIL();
}
```

### ❌ DON'T: Mix ABIs in Same Block

```aria
// Bad - different ABIs
extern "C" {
    func:c_function = NIL();
}
extern "system" {  // ✅ Separate block
    func:system_function = NIL();
}
```

---

## Modular Organization

```aria
// In ffi/libc.aria
pub mod libc {
    extern "C" {
        pub func:malloc = *void;(uint64:size)
        pub func:free = NIL(*void:ptr);
    }
}

// In ffi/math.aria
pub mod math {
    #[link(name = "m")]
    extern "C" {
        pub func:sqrt = f64;(flt64:x)
        pub func:sin = f64;(flt64:x)
    }
}

// Use them
use ffi.libc;
use ffi.math;

ptr: *void = libc.malloc(100);
Result: f64 = math.sqrt(16.0);
```

---

## Related

- [extern](extern.md) - Extern keyword
- [extern_syntax](extern_syntax.md) - Extern syntax
- [extern_functions](extern_functions.md) - External functions
- [ffi](ffi.md) - FFI overview

---

**Remember**: **Organize** extern blocks by library or functionality!
