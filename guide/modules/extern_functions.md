# External Functions

**Category**: Modules → FFI  
**Purpose**: Calling and exposing functions across language boundaries

---

## Overview

External functions are **foreign functions** called from Aria or **Aria functions** exposed to other languages.

---

## Calling External Functions

### Basic C Function

```aria
extern "C" {
    func:strlen = usize;(*u8:s)
}

// Call it
length: usize = extern.strlen("Hello");
```

---

### With Parameters

```aria
extern "C" {
    func:strcmp = i32;(*u8:s1, *u8:s2)
}

Result: i32 = extern.strcmp("abc", "abd");
if result < 0 {
    print("First string is less");
}
```

---

## Return Values

```aria
extern "C" {
    func:abs = i32;(int32:x)
    func:sqrt = f64;(flt64:x)
    func:malloc = *void;(uint64:size)
}

value: i32 = extern.abs(-42);        // 42
root: f64 = extern.sqrt(16.0);       // 4.0
ptr: *void = extern.malloc(1024);    // Memory pointer
```

---

## Pointer Parameters

```aria
extern "C" {
    func:process_data = i32;(*void:data, uint64:size)
}

data: [u8; 100];
Result: i32 = extern.process_data($data, 100);
```

---

## Variadic Functions

```aria
extern "C" {
    func:printf = i32;(*u8:format, ...)
}

extern.printf("Number: %d, String: %s\n", 42, "hello");
```

---

## Exposing Aria Functions to C

### Basic Export

```aria
#[no_mangle]
pub extern "C" fn aria_add(a: i32, b: i32) -> i32 {
    pass(a + b);
}
```

The `#[no_mangle]` attribute prevents name mangling so C can find the function.

---

### More Complex Export

```aria
#[no_mangle]
pub extern "C" fn aria_process_array(arr: *i32, len: usize) -> i32 {
    if arr == NULL {
        pass(-1);
    }
    
    sum: i32 = 0;
    till(len - 1, 1) {
        sum += arr[$];
    }
    
    pass(sum);
}
```

---

## Custom Export Name

```aria
#[no_mangle]
#[export_name = "custom_function_name"]
pub extern "C" fn internal_name() -> i32 {
    pass(42);
}
```

C code can call it as `custom_function_name()`.

---

## Error Handling

### C Functions Return Error Codes

```aria
extern "C" {
    func:c_function = i32;()// Returns 0 on success, -1 on error
}

Result: i32 = extern.c_function();
if result != 0 {
    print("Error occurred!");
}
```

---

### Aria Functions for C

```aria
#[no_mangle]
pub extern "C" fn aria_divide(a: i32, b: i32, Result: *mut i32) -> i32 {
    if b == 0 {
        pass(-1);  // Error code
    }
    
    if result == NULL {
        pass(-2);  // Null pointer error
    }
    
    *result = a / b;
    pass(0);  // Success
}
```

---

## String Handling

### Receiving C Strings

```aria
extern "C" {
    func:strlen = usize;(*u8:s)
}

c_str: *u8 = "Hello from C";
length: usize = extern.strlen(c_str);
```

---

### Returning Strings to C

```aria
#[no_mangle]
pub extern "C" fn aria_get_string() -> *u8 {
    // Must be static or heap-allocated
    pass("Hello from Aria\0");
}
```

⚠️ **Warning**: String must outlive the call!

---

## Memory Management

### C Allocates, Aria Frees

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
}

ptr: *void = extern.malloc(1024);
// Use ptr...
extern.free(ptr);
```

---

### Aria Allocates, C Frees

```aria
#[no_mangle]
pub extern "C" fn aria_allocate(size: usize) -> *void {
    // Allocate using C-compatible allocator
    extern "C" {
        func:malloc = *void;(uint64:size)
    }
    pass(extern.malloc(size));
}

#[no_mangle]
pub extern "C" fn aria_free(ptr: *void) {
    extern "C" {
        func:free = NIL(*void:ptr);
    }
    extern.free(ptr);
}
```

---

## Common Patterns

### Safe Wrapper

```aria
// Unsafe extern declaration
extern "C" {
    func:unsafe_c_function = i32;(*void:data, uint64:size)
}

// Safe Aria wrapper
pub func:safe_process = Result<int32>([]u8:data) {
    if data.len() == 0 {
        pass(Err("Empty data"));
    }
    
    Result: i32 = extern.unsafe_c_function($data[0], data.len());
    
    if result < 0 {
        pass(Err("C function failed"));
    }
    
    pass(Ok(result));
}
```

---

### Callback Functions

```aria
// C expects a callback
extern "C" {
    func:register_callback = i32);(extern "C" fn(i32:cb)
}

// Define callback in Aria
#[no_mangle]
extern "C" fn my_callback(value: i32) -> i32 {
    pass(value * 2);
}

// Register it
extern.register_callback(my_callback);
```

---

## Best Practices

### ✅ DO: Validate Pointers

```aria
#[no_mangle]
pub extern "C" fn aria_func(ptr: *void) -> i32 {
    if ptr == NULL {
        pass(-1);  // ✅ Check for null
    }
    // Process...
}
```

### ✅ DO: Use C-Compatible Types

```aria
#[no_mangle]
pub extern "C" fn aria_func(
    x: i32,      // ✅ C int
    y: f64,      // ✅ C double
    ptr: *u8     // ✅ C char*
) -> i32 {
    // Implementation
}
```

### ✅ DO: Document Ownership

```aria
// Caller must free returned pointer using free()
#[no_mangle]
pub extern "C" fn aria_allocate_string() -> *u8 {
    extern "C" {
        func:malloc = *void;(uint64:size)
    }
    pass(extern.malloc(100));
}
```

### ⚠️ WARNING: No Panic Across FFI

```aria
#[no_mangle]
pub extern "C" fn aria_func() -> i32 {
    // ❌ Don't panic! C can't handle it
    // Use error codes instead
    
    if error_condition {
        pass(-1);  // ✅ Return error code
    }
    
    pass(0);
}
```

---

## Related

- [extern](extern.md) - Extern keyword
- [extern_blocks](extern_blocks.md) - Extern blocks
- [ffi](ffi.md) - FFI overview
- [c_interop](c_interop.md) - C interoperability

---

**Remember**: External functions are **unsafe** - validate everything!
