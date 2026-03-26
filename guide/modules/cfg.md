# `cfg` Attribute

**Category**: Modules → Conditional Compilation  
**Syntax**: `#[cfg(...)]`  
**Purpose**: Conditionally compile code based on configuration

---

## Overview

`cfg` enables **platform-specific** and **feature-specific** compilation.

---

## Basic Syntax

```aria
#[cfg(condition)]
item
```

---

## Operating System

```aria
#[cfg(os = "linux")]
func:linux_specific = NIL() {
    print("Running on Linux");
}

#[cfg(os = "windows")]
func:windows_specific = NIL() {
    print("Running on Windows");
}

#[cfg(os = "macos")]
func:macos_specific = NIL() {
    print("Running on macOS");
}
```

---

## Architecture

```aria
#[cfg(arch = "x86_64")]
func:x86_64_code = NIL() {
    print("64-bit x86");
}

#[cfg(arch = "aarch64")]
func:arm_code = NIL() {
    print("ARM 64-bit");
}

#[cfg(arch = "wasm32")]
func:wasm_code = NIL() {
    print("WebAssembly");
}
```

---

## Features

```aria
#[cfg(feature = "logging")]
func:log_message = NIL(string:msg) {
    print(`[LOG] &{msg}`);
}

#[cfg(feature = "database")]
use database.*;

#[cfg(feature = "cache")]
mod cache {
    // Cache implementation
}
```

---

## Build Configuration

```aria
#[cfg(debug)]
func:debug_only = NIL() {
    print("Debug build");
}

#[cfg(release)]
func:release_only = NIL() {
    print("Release build");
}
```

---

## Boolean Logic

### `not`

```aria
#[cfg(not(os = "windows"))]
func:unix_like = NIL() {
    // Runs on non-Windows platforms
}
```

### `all` (AND)

```aria
#[cfg(all(os = "linux", arch = "x86_64"))]
func:linux_x86_64 = NIL() {
    // Only on 64-bit Linux
}
```

### `any` (OR)

```aria
#[cfg(any(os = "linux", os = "macos"))]
func:unix_platforms = NIL() {
    // Runs on Linux or macOS
}
```

---

## Complex Conditions

```aria
#[cfg(all(
    any(os = "linux", os = "macos"),
    not(arch = "wasm32")
))]
func:native_unix = NIL() {
    // Linux or macOS, but not WebAssembly
}
```

---

## Module-Level

```aria
#[cfg(os = "linux")]
mod linux {
    pub func:platform_init = NIL() {
        // Linux initialization
    }
}

#[cfg(os = "windows")]
mod windows {
    pub func:platform_init = NIL() {
        // Windows initialization
    }
}
```

---

## Import Conditional

```aria
#[cfg(feature = "http")]
use std.http;

#[cfg(feature = "database")]
use database.sqlite;
```

---

## Struct Fields

```aria
struct Config {
    common_field: string,
    
    #[cfg(os = "windows")]
    windows_setting: i32,
    
    #[cfg(os = "linux")]
    linux_setting: i32,
}
```

---

## Function Body

```aria
func:platform_specific_code = NIL() {
    #[cfg(os = "linux")]
    {
        print("Linux code");
    }
    
    #[cfg(os = "windows")]
    {
        print("Windows code");
    }
}
```

---

## Common Conditions

### Operating Systems

```aria
os = "linux"
os = "windows"
os = "macos"
os = "ios"
os = "android"
os = "freebsd"
```

### Architectures

```aria
arch = "x86"
arch = "x86_64"
arch = "arm"
arch = "aarch64"
arch = "wasm32"
arch = "wasm64"
```

### Pointer Width

```aria
target_pointer_width = "32"
target_pointer_width = "64"
```

### Endianness

```aria
target_endian = "little"
target_endian = "big"
```

---

## Best Practices

### ✅ DO: Provide Defaults

```aria
#[cfg(os = "windows")]
const PATH_SEP: u8 = '\\';

#[cfg(not(os = "windows"))]
const PATH_SEP: u8 = '/';
```

### ✅ DO: Group Platform Code

```aria
#[cfg(os = "linux")]
mod platform {
    pub func:init = NIL() { }
    pub func:cleanup = NIL() { }
}

#[cfg(os = "windows")]
mod platform {
    pub func:init = NIL() { }
    pub func:cleanup = NIL() { }
}
```

### ✅ DO: Document Conditions

```aria
// Only available on Unix-like systems
#[cfg(any(os = "linux", os = "macos"))]
pub func:unix_socket = Socket() {
    // Implementation
}
```

### ❌ DON'T: Over-complicate

```aria
#[cfg(all(
    any(os = "linux", os = "macos"),
    not(any(arch = "wasm32", arch = "wasm64")),
    feature = "advanced"
))]  // ❌ Too complex!
```

---

## cfg_attr

Apply attributes conditionally:

```aria
#[cfg_attr(os = "windows", link(name = "ws2_32"))]
extern "C" {
    func:network_function = NIL();
}
```

---

## Related

- [conditional_compilation](conditional_compilation.md) - Conditional compilation patterns

---

**Remember**: Use `cfg` for **platform** and **feature** differences!
