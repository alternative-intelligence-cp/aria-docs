# Foreign Function Interface (FFI)

**Category**: Modules → FFI  
**Purpose**: Interfacing Aria with C and other languages

---

## Overview

FFI allows Aria to **call foreign code** (C, C++, system libraries) and be **called by** foreign code.

---

## Why FFI?

- ✅ Use existing C libraries
- ✅ Call system APIs
- ✅ Interop with legacy code
- ✅ Access platform-specific features
- ✅ Integrate with databases, crypto libraries, etc.

---

## Basic FFI Pattern

### 1. Declare Foreign Function

```aria
extern "C" {
    func:c_function = i32;(int32:x)
}
```

### 2. Link Library

```aria
#[link(name = "mylib")]
extern "C" {
    func:library_function = NIL();
}
```

### 3. Call It

```aria
Result: i32 = extern.c_function(42);
```

---

## Calling C from Aria

```aria
// Declare C functions
extern "C" {
    func:strlen = usize;(*u8:s)
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
}

// Use them
length: usize = extern.strlen("Hello");
ptr: *void = extern.malloc(1024);
// ... use ptr ...
extern.free(ptr);
```

---

## Calling Aria from C

```aria
// Export Aria function to C
#[no_mangle]
pub extern "C" fn aria_add(a: i32, b: i32) -> i32 {
    pass(a + b);
}
```

C code:
```c
// Declare in C header
extern int aria_add(int a, int b);

// Call it
int result = aria_add(5, 3);  // 8
```

---

## Type Mapping

### Integers

| Aria | C | Size |
|------|---|------|
| `i8` | `int8_t` / `char` | 8-bit |
| `i16` | `int16_t` / `short` | 16-bit |
| `i32` | `int32_t` / `int` | 32-bit |
| `i64` | `int64_t` / `long long` | 64-bit |
| `u8` | `uint8_t` / `unsigned char` | 8-bit |
| `u16` | `uint16_t` / `unsigned short` | 16-bit |
| `u32` | `uint32_t` / `unsigned int` | 32-bit |
| `u64` | `uint64_t` / `unsigned long long` | 64-bit |

### Floats

| Aria | C | Size |
|------|---|------|
| `f32` | `float` | 32-bit |
| `f64` | `double` | 64-bit |

### Pointers

| Aria | C |
|------|---|
| `*void` | `void*` |
| `*u8` | `char*` / `uint8_t*` |
| `*const T` | `const T*` |
| `*mut T` | `T*` |

---

## Common FFI Patterns

### Using libc

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:memcpy = *void;(*void:dest, *void:src, uint64:n)
    func:strlen = usize;(*u8:s)
}
```

---

### Using Math Library

```aria
#[link(name = "m")]
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:sin = f64;(flt64:x)
    func:cos = f64;(flt64:x)
    func:pow = f64;(flt64:base, flt64:exp)
}

Result: f64 = extern.sqrt(16.0);  // 4.0
```

---

### Safe Wrapper Pattern

```aria
// Unsafe extern
extern "C" {
    func:unsafe_c_function = i32;(*void:data, uint64:size)
}

// Safe Aria wrapper
pub func:safe_wrapper = Result<NIL>([]u8:data) {
    if data.len() == 0 {
        fail("Empty data");
    }
    
    Result: i32 = extern.unsafe_c_function($data[0], data.len());
    
    if result != 0 {
        fail("Operation failed");
    }
    
    pass();
}
```

---

## Memory Management

### Rule: **Allocator owns the memory**

```aria
// If C allocates, C must free
extern "C" {
    func:c_allocate = *void;()
    func:c_free = NIL(*void:ptr);
}

ptr: *void = extern.c_allocate();
// ... use ptr ...
extern.c_free(ptr);  // ✅ C frees what C allocated
```

```aria
// If Aria allocates for C, C must know how to free it
#[no_mangle]
pub extern "C" fn aria_allocate() -> *void {
    // Use C allocator so C can free
    extern "C" {
        func:malloc = *void;(uint64:size)
    }
    pass(extern.malloc(1024));
}
```

---

## String Handling

### C String to Aria

```aria
extern "C" {
    func:get_c_string = *u8;()// Returns null-terminated string
}

c_str: *u8 = extern.get_c_string();

// Convert to Aria string (copy)
aria_str: string = string.from_c_str(c_str);
```

---

### Aria String to C

```aria
#[no_mangle]
pub extern "C" fn aria_get_string() -> *u8 {
    // Must be static or heap-allocated and null-terminated
    pass("Hello from Aria\0");
}
```

---

## Error Handling

### C Functions

```aria
extern "C" {
    func:c_function = i32;()// 0 = success, -1 = error
}

Result: i32 = extern.c_function();
if result != 0 {
    fail("C function failed");
}
```

---

### Aria Functions for C

```aria
#[no_mangle]
pub extern "C" fn aria_function(out: *mut i32) -> i32 {
    if out == NULL {
        pass(-1);  // Error: null pointer
    }
    
    *out = 42;
    pass(0);  // Success
}
```

---

## Platform-Specific FFI

```aria
#[cfg(target_os = "linux")]
extern "C" {
    func:linux_specific_function = NIL();
}

#[cfg(target_os = "windows")]
extern "system" {
    func:windows_specific_function = NIL();
}

#[cfg(target_os = "macos")]
extern "C" {
    func:macos_specific_function = NIL();
}
```

---

## Callbacks

```aria
// C library expects a callback
extern "C" {
    func:register_callback = i32);(extern "C" fn(i32:cb)
}

// Define Aria callback
#[no_mangle]
extern "C" fn my_callback(value: i32) -> i32 {
    pass(value * 2);
}

// Register
extern.register_callback(my_callback);
```

---

## Best Practices

### ✅ DO: Wrap Unsafe FFI

```aria
// Hide unsafe extern behind safe API
pub func:safe_api = Result<Data>() {
    // Validate inputs
    // Call unsafe extern
    // Check results
    // Return safely
}
```

### ✅ DO: Check Pointers

```aria
if ptr == NULL {
    fail("Null pointer");
}
```

### ✅ DO: Document Ownership

```aria
// Returns pointer owned by caller - must call free()
#[no_mangle]
pub extern "C" fn aria_allocate() -> *void {
    // ...
}
```

### ⚠️ WARNING: Don't Panic

```aria
#[no_mangle]
pub extern "C" fn aria_func() -> i32 {
    // ❌ Never panic across FFI boundary!
    // ✅ Return error codes instead
}
```

### ⚠️ WARNING: ABI Matters

```aria
extern "C" { }      // ✅ For C functions
extern "system" { } // ✅ For Windows system functions
// Wrong ABI = crashes!
```

---

## Common Libraries

### SQLite

```aria
#[link(name = "sqlite3")]
extern "C" {
    func:sqlite3_open = i32;(*u8:filename, **void:db)
    func:sqlite3_close = i32;(*void:db)
    func:sqlite3_exec = i32;(*void:db, *u8:sql)
}
```

### OpenSSL

```aria
#[link(name = "ssl")]
#[link(name = "crypto")]
extern "C" {
    func:SSL_library_init = i32;()
    func:SSL_CTX_new = *void;()
}
```

---

## Related

- [extern](extern.md) - Extern keyword
- [c_interop](c_interop.md) - C interoperability
- [c_pointers](c_pointers.md) - C pointer handling
- [libc_integration](libc_integration.md) - libc integration

---

**Remember**: FFI is **powerful** but **dangerous** - always validate and wrap!
