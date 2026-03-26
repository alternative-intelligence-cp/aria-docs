# libc Integration

**Category**: Modules → FFI  
**Purpose**: Using the C standard library from Aria

---

## Overview

libc (C standard library) provides **fundamental** system operations - memory, strings, I/O, math, and more.

---

## Commonly Used libc Functions

### Memory Management

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
    func:calloc = *void;(uint64:num, uint64:size)
    func:realloc = *void;(*void:ptr, uint64:new_size)
    func:memcpy = *void;(*void:dest, *void:src, uint64:n)
    func:memset = *void;(*void:ptr, int32:value, uint64:n)
    func:memmove = *void;(*void:dest, *void:src, uint64:n)
}
```

---

### String Operations

```aria
extern "C" {
    func:strlen = usize;(*u8:s)
    func:strcmp = i32;(*u8:s1, *u8:s2)
    func:strcpy = *u8;(*u8:dest, *u8:src)
    func:strcat = *u8;(*u8:dest, *u8:src)
    func:strncpy = *u8;(*u8:dest, *u8:src, uint64:n)
    func:strncmp = i32;(*u8:s1, *u8:s2, uint64:n)
}
```

---

### File I/O

```aria
extern "C" {
    func:fopen = *void;(*u8:path, *u8:mode)
    func:fclose = i32;(*void:file)
    func:fread = usize;(*void:ptr, uint64:size, uint64:count, *void:file)
    func:fwrite = usize;(*void:ptr, uint64:size, uint64:count, *void:file)
    func:fseek = i32;(*void:file, int64:offset, int32:whence)
    func:ftell = i64;(*void:file)
}
```

---

### Standard I/O

```aria
extern "C" {
    func:printf = i32;(*u8:format, ...)
    func:scanf = i32;(*u8:format, ...)
    func:puts = i32;(*u8:s)
    func:putchar = i32;(int32:c)
    func:getchar = i32;()
}
```

---

### Math Functions

```aria
#[link(name = "m")]
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:pow = f64;(flt64:base, flt64:exp)
    func:sin = f64;(flt64:x)
    func:cos = f64;(flt64:x)
    func:tan = f64;(flt64:x)
    func:log = f64;(flt64:x)
    func:log10 = f64;(flt64:x)
    func:exp = f64;(flt64:x)
    func:floor = f64;(flt64:x)
    func:ceil = f64;(flt64:x)
    func:fabs = f64;(flt64:x)
}
```

---

## Memory Examples

### malloc/free

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
}

// Allocate
ptr: *void = extern.malloc(1024);
if ptr == NULL {
    pass(Err("Allocation failed"));
}

// Use ptr...

// Free
extern.free(ptr);
```

---

### calloc (Zero-initialized)

```aria
extern "C" {
    func:calloc = *void;(uint64:num, uint64:size)
    func:free = NIL(*void:ptr);
}

// Allocate array of 10 integers, zero-initialized
arr: *i32 = extern.calloc(10, sizeof(i32)) as *i32;

// Use arr[0] through arr[9]

extern.free(arr as *void);
```

---

### memcpy

```aria
extern "C" {
    func:memcpy = *void;(*void:dest, *void:src, uint64:n)
}

src: [u8; 100];
dest: [u8; 100];

// Copy 100 bytes from src to dest
extern.memcpy($dest[0], $src[0], 100);
```

---

## String Examples

### strlen

```aria
extern "C" {
    func:strlen = usize;(*u8:s)
}

str: *u8 = "Hello, World!\0";
len: usize = extern.strlen(str);  // 13
```

---

### strcmp

```aria
extern "C" {
    func:strcmp = i32;(*u8:s1, *u8:s2)
}

Result: i32 = extern.strcmp("apple\0", "banana\0");
if result < 0 {
    print("apple comes first");
} else if result > 0 {
    print("banana comes first");
} else {
    print("strings are equal");
}
```

---

### strcpy

```aria
extern "C" {
    func:strcpy = *u8;(*u8:dest, *u8:src)
}

src: *u8 = "Hello\0";
dest: [u8; 100];

extern.strcpy($dest[0], src);
// dest now contains "Hello\0"
```

---

## File I/O Examples

### Reading a File

```aria
extern "C" {
    func:fopen = *void;(*u8:path, *u8:mode)
    func:fclose = i32;(*void:file)
    func:fread = usize;(*void:ptr, uint64:size, uint64:count, *void:file)
}

file: *void = extern.fopen("data.txt\0", "r\0");
if file == NULL {
    pass(Err("Failed to open file"));
}

buffer: [u8; 1024];
bytes_read: usize = extern.fread($buffer[0], 1, 1024, file);

print(`Read &{bytes_read} bytes`);

extern.fclose(file);
```

---

### Writing a File

```aria
extern "C" {
    func:fopen = *void;(*u8:path, *u8:mode)
    func:fclose = i32;(*void:file)
    func:fwrite = usize;(*void:ptr, uint64:size, uint64:count, *void:file)
}

file: *void = extern.fopen("output.txt\0", "w\0");
if file == NULL {
    pass(Err("Failed to open file"));
}

data: *u8 = "Hello, File!\0";
bytes_written: usize = extern.fwrite(data, 1, 12, file);

extern.fclose(file);
```

---

## Math Examples

```aria
#[link(name = "m")]
extern "C" {
    func:sqrt = f64;(flt64:x)
    func:pow = f64;(flt64:base, flt64:exp)
    func:sin = f64;(flt64:x)
}

root: f64 = extern.sqrt(16.0);        // 4.0
power: f64 = extern.pow(2.0, 10.0);   // 1024.0
sine: f64 = extern.sin(0.0);          // 0.0
```

---

## Error Handling

### errno

```aria
extern "C" {
    static errno: i32;
    func:fopen = *void;(*u8:path, *u8:mode)
}

file: *void = extern.fopen("nonexistent.txt\0", "r\0");
if file == NULL {
    error: i32 = extern.errno;
    pass(Err(`fopen failed with errno: &{error}`));
}
```

---

### Checking Return Values

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
}

ptr: *void = extern.malloc(1024);
if ptr == NULL {
    pass(Err("malloc failed - out of memory"));
}
```

---

## Safe Wrappers

### Memory Allocation

```aria
extern "C" {
    func:malloc = *void;(uint64:size)
    func:free = NIL(*void:ptr);
}

pub func:safe_alloc = Result<*void>(uint64:size) {
    if size == 0 {
        pass(Err("Invalid size"));
    }
    
    ptr: *void = extern.malloc(size);
    if ptr == NULL {
        pass(Err("Allocation failed"));
    }
    
    pass(Ok(ptr));
}

pub func:safe_free = NIL(*void:ptr) {
    if ptr != NULL {
        extern.free(ptr);
    }
}
```

---

### String Operations

```aria
extern "C" {
    func:strlen = usize;(*u8:s)
}

pub func:safe_strlen = Result<uint64>(*u8:s) {
    if s == NULL {
        pass(Err("Null string"));
    }
    
    pass(Ok(extern.strlen(s)));
}
```

---

## Platform-Specific libc

```aria
#[cfg(target_os = "linux")]
extern "C" {
    func:getpid = i32;()
    func:getuid = i32;()
}

#[cfg(target_os = "windows")]
extern "C" {
    func:GetCurrentProcessId = u32;()
}
```

---

## Best Practices

### ✅ DO: Check Return Values

```aria
ptr: *void = extern.malloc(1024);
if ptr == NULL {  // ✅ Always check!
    pass(Err("Allocation failed"));
}
```

### ✅ DO: Free Allocated Memory

```aria
ptr: *void = extern.malloc(1024);
// ... use ptr ...
extern.free(ptr);  // ✅ Don't leak!
```

### ✅ DO: Null-Terminate Strings

```aria
str: *u8 = "Hello\0";  // ✅ Null terminator for C
```

### ⚠️ WARNING: Buffer Overflows

```aria
extern "C" {
    func:strcpy = *u8;(*u8:dest, *u8:src)
}

dest: [u8; 10];
src: *u8 = "This string is way too long\0";
extern.strcpy($dest[0], src);  // ⚠️ Buffer overflow!

// Use strncpy instead
extern "C" {
    func:strncpy = *u8;(*u8:dest, *u8:src, uint64:n)
}
extern.strncpy($dest[0], src, 9);  // ✅ Safe
```

---

## Common libc Constants

```aria
// File modes
const O_RDONLY: i32 = 0;
const O_WRONLY: i32 = 1;
const O_RDWR: i32 = 2;

// Seek whence
const SEEK_SET: i32 = 0;
const SEEK_CUR: i32 = 1;
const SEEK_END: i32 = 2;

// File stream pointers
extern "C" {
    static stdin: *void;
    static stdout: *void;
    static stderr: *void;
}
```

---

## Related

- [c_interop](c_interop.md) - C interoperability
- [c_pointers](c_pointers.md) - C pointer handling
- [ffi](ffi.md) - FFI overview

---

**Remember**: libc is **battle-tested** but **unsafe** - validate inputs and check returns!
