# Conditional Compilation

**Category**: Modules → Compilation  
**Purpose**: Compile different code for different platforms and features

---

## Overview

Conditional compilation allows **one codebase** to support multiple platforms, architectures, and feature sets.

---

## Why Conditional Compilation?

- ✅ Cross-platform code
- ✅ Feature flags
- ✅ Debug vs release builds
- ✅ Architecture-specific optimizations
- ✅ Optional dependencies

---

## Platform-Specific Code

### Operating System

```aria
#[cfg(os = "linux")]
func:get_home_dir = string() {
    pass("/home/user");
}

#[cfg(os = "windows")]
func:get_home_dir = string() {
    pass("C:\\Users\\user");
}

#[cfg(os = "macos")]
func:get_home_dir = string() {
    pass("/Users/user");
}
```

---

### Architecture

```aria
#[cfg(arch = "x86_64")]
func:optimized_function = NIL() {
    // x86_64 SIMD instructions
}

#[cfg(arch = "aarch64")]
func:optimized_function = NIL() {
    // ARM NEON instructions
}

#[cfg(not(any(arch = "x86_64", arch = "aarch64")))]
func:optimized_function = NIL() {
    // Generic fallback
}
```

---

## Feature Flags

### Enabling Features

```toml
# In Cargo.toml or package config
[features]
default = ["logging"]
logging = []
database = []
cache = []
full = ["logging", "database", "cache"]
```

### Using Features

```aria
#[cfg(feature = "logging")]
pub func:log = NIL(string:msg) {
    print(`[LOG] &{msg}`);
}

#[cfg(feature = "database")]
pub mod database {
    pub func:connect = Connection() {
        // Database connection
    }
}

#[cfg(feature = "cache")]
pub mod cache {
    pub func:get = ?Data(string:key) {
        // Cache lookup
    }
}
```

---

## Debug vs Release

```aria
#[cfg(debug)]
pub func:debug_info = NIL() {
    print("Debug build");
    print("Extra checks enabled");
}

#[cfg(release)]
pub func:debug_info = NIL() {
    // Optimized - no debug output
}

#[cfg(debug)]
const LOG_LEVEL: string = "DEBUG";

#[cfg(release)]
const LOG_LEVEL: string = "INFO";
```

---

## Complex Conditions

### AND Logic

```aria
#[cfg(all(os = "linux", arch = "x86_64"))]
func:linux_x86_64_specific = NIL() {
    // Only on 64-bit Linux
}
```

---

### OR Logic

```aria
#[cfg(any(os = "linux", os = "macos", os = "freebsd"))]
func:unix_like = NIL() {
    // Any Unix-like system
}
```

---

### NOT Logic

```aria
#[cfg(not(os = "windows"))]
func:non_windows = NIL() {
    // Everything except Windows
}
```

---

### Combined

```aria
#[cfg(all(
    any(os = "linux", os = "macos"),
    arch = "x86_64",
    not(feature = "minimal")
))]
func:advanced_unix_feature = NIL() {
    // 64-bit Unix with full features
}
```

---

## Common Patterns

### Platform Abstraction

```aria
// Platform-specific implementations
#[cfg(os = "linux")]
mod platform {
    pub func:sleep = NIL(int32:ms) {
        extern "C" {
            func:usleep = NIL(int32:usec);
        }
        extern.usleep(ms * 1000);
    }
}

#[cfg(os = "windows")]
mod platform {
    pub func:sleep = NIL(int32:ms) {
        extern "system" {
            func:Sleep = NIL(int32:ms);
        }
        extern.Sleep(ms);
    }
}

// Common interface
pub func:sleep = NIL(int32:ms) {
    platform.sleep(ms);
}
```

---

### Feature-Gated Modules

```aria
// Main module
pub mod core {
    pub func:basic_function = NIL() { }
}

// Optional modules
#[cfg(feature = "advanced")]
pub mod advanced {
    pub func:advanced_function = NIL() { }
}

#[cfg(feature = "experimental")]
pub mod experimental {
    pub func:experimental_function = NIL() { }
}
```

---

### Fallback Implementations

```aria
// Optimized version for x86_64
#[cfg(arch = "x86_64")]
func:fast_checksum = uint32([]u8:data) {
    // SIMD implementation
}

// Generic fallback
#[cfg(not(arch = "x86_64"))]
func:fast_checksum = uint32([]u8:data) {
    // Portable implementation
}
```

---

## Conditional Imports

```aria
#[cfg(feature = "json")]
use serde_json;

#[cfg(feature = "yaml")]
use serde_yaml;

#[cfg(feature = "database")]
use database.{connect, query};

#[cfg(all(os = "linux", feature = "epoll"))]
use linux.epoll;
```

---

## Conditional Structs

```aria
pub struct Config {
    // Common fields
    pub name: string,
    pub version: string,
    
    // Platform-specific
    #[cfg(os = "windows")]
    pub windows_registry_key: string,
    
    #[cfg(os = "linux")]
    pub linux_systemd_unit: string,
    
    // Feature-gated
    #[cfg(feature = "database")]
    pub db_connection_string: string,
    
    #[cfg(feature = "cache")]
    pub cache_size: usize,
}
```

---

## Conditional Tests

```aria
#[cfg(test)]
mod tests {
    #[cfg(os = "linux")]
    #[test]
    func:test_linux_specific = NIL() {
        // Only runs on Linux
    }
    
    #[cfg(feature = "database")]
    #[test]
    func:test_database = NIL() {
        // Only runs when database feature enabled
    }
}
```

---

## Development Helpers

```aria
#[cfg(debug)]
pub func:assert_valid = NIL(Data:data) {
    if !data.is_valid() {
        panic("Invalid data in debug mode");
    }
}

#[cfg(release)]
pub func:assert_valid = NIL(Data:data) {
    // No-op in release
}
```

---

## Target Detection

```aria
// Check at compile time
#[cfg(target_pointer_width = "64")]
const IS_64_BIT: bool = true;

#[cfg(target_pointer_width = "32")]
const IS_64_BIT: bool = false;

#[cfg(target_endian = "little")]
const IS_LITTLE_ENDIAN: bool = true;

#[cfg(target_endian = "big")]
const IS_LITTLE_ENDIAN: bool = false;
```

---

## Best Practices

### ✅ DO: Provide Fallbacks

```aria
#[cfg(feature = "optimized")]
func:process = NIL() {
    // Optimized version
}

#[cfg(not(feature = "optimized"))]
func:process = NIL() {
    // Standard version
}
```

### ✅ DO: Group Platform Code

```aria
// Good organization
#[cfg(os = "linux")]
mod linux_impl;

#[cfg(os = "windows")]
mod windows_impl;

#[cfg(os = "macos")]
mod macos_impl;

// Common interface
pub use platform_impl::*;
```

### ✅ DO: Document Conditions

```aria
/// Available on Unix-like systems only
#[cfg(any(os = "linux", os = "macos", os = "freebsd"))]
pub func:unix_socket_connect = Socket(string:path) {
    // Implementation
}
```

### ❌ DON'T: Duplicate Code

```aria
// Bad
#[cfg(os = "linux")]
func:process = NIL() {
    common_logic();  // Duplicated
    linux_specific();
}

#[cfg(os = "windows")]
func:process = NIL() {
    common_logic();  // Duplicated
    windows_specific();
}

// Better
func:process = NIL() {
    common_logic();
    
    #[cfg(os = "linux")]
    linux_specific();
    
    #[cfg(os = "windows")]
    windows_specific();
}
```

---

## Common Configurations

### Cross-Platform File Paths

```aria
#[cfg(os = "windows")]
const PATH_SEPARATOR: u8 = '\\';

#[cfg(not(os = "windows"))]
const PATH_SEPARATOR: u8 = '/';
```

### Networking

```aria
#[cfg(os = "windows")]
#[link(name = "ws2_32")]
extern "system" {
    func:WSAStartup = i32;(int32:version, *void:data)
}

#[cfg(any(os = "linux", os = "macos"))]
extern "C" {
    func:socket = i32;(int32:domain, int32:type, int32:protocol)
}
```

---

## Related

- [cfg](cfg.md) - cfg attribute

---

**Remember**: One codebase, **many targets** - use conditional compilation wisely!
