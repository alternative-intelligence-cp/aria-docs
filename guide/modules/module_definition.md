# Module Definition

**Category**: Modules → Structure  
**Purpose**: How to define and structure modules

---

## Overview

Modules are defined either as **files** or **inline declarations**.

---

## File-Based Modules

### Single File

```aria
// math.aria
pub func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

pub func:subtract = int32(int32:a, int32:b) {
    pass(a - b);
}
```

File `math.aria` automatically becomes module `math`.

---

### Directory Module

```
utils/
├── mod.aria       # Module entry point
├── string.aria    # Submodule
└── array.aria     # Submodule
```

```aria
// utils/mod.aria
pub mod string;
pub mod array;

pub func:common_util = NIL() {
    // Shared utility
}
```

```aria
// utils/string.aria
pub func:reverse = string(string:s) {
    // Implementation
}
```

---

## Inline Modules

```aria
// main.aria
mod helpers {
    pub func:format_output = string(int32:value) {
        pass(`Value: &{value}`);
    }
}

func:main = NIL() {
    output: string = helpers.format_output(42);
    print(output);
}
```

---

## Module Structure

### Typical Module

```aria
// database.aria

// Private constants
const CONNECTION_TIMEOUT: i32 = 30;

// Private types
struct ConnectionPool {
    connections: []Connection
}

// Public API
pub func:connect = Result<Connection>(string:url) {
    // Use private items
    pool: ConnectionPool = get_pool();
    pass(pool.acquire());
}

pub func:query = Result<Data>(string:sql) {
    // Implementation
}

// Private helper
func:get_pool = ConnectionPool() {
    // Implementation
}
```

---

## Module Organization

### Flat Structure

```
src/
├── main.aria
├── auth.aria
├── database.aria
└── api.aria
```

```aria
// main.aria
mod auth;
mod database;
mod api;
```

---

### Hierarchical Structure

```
src/
├── main.aria
├── auth/
│   ├── mod.aria
│   ├── login.aria
│   └── tokens.aria
└── database/
    ├── mod.aria
    ├── connection.aria
    └── queries.aria
```

```aria
// main.aria
mod auth;
mod database;

// auth/mod.aria
pub mod login;
pub mod tokens;
```

---

## Module Items

### What Can Be in a Module?

```aria
// All valid module items:

pub const MAX_SIZE: i32 = 1000;        // Constants

pub struct User {                      // Structs
    name: string,
    age: i32
}

pub enum Status {                      // Enums
    Active,
    Inactive
}

pub func:process = Result<NIL>() {     // Functions
    // Implementation
}

pub mod submodule {                    // Nested modules
    pub func:helper = NIL() { }
}
```

---

## Best Practices

### ✅ DO: Group Related Functionality

```aria
// user.aria - everything user-related
pub struct User { }
pub func:create_user = User() { }
pub func:delete_user = NIL() { }
pub func:find_user = ?User() { }
```

### ✅ DO: Use mod.aria for Complex Modules

```
database/
  mod.aria          # Public API
  connection.aria   # Private implementation
  pool.aria         # Private implementation
  queries.aria      # Private implementation
```

```aria
// database/mod.aria
mod connection;
mod pool;
mod queries;

// Re-export public API
pub use connection.connect;
pub use queries.query;
```

### ❌ DON'T: Mix Unrelated Code

```aria
// bad_module.aria
pub func:user_login = NIL() { }
pub func:database_query = NIL() { }  // ❌ Different concerns
pub func:send_email = NIL() { }      // ❌ Different concerns
```

---

## Related

- [mod](mod.md) - Modules overview
- [mod_keyword](mod_keyword.md) - mod keyword
- [module_paths](module_paths.md) - Module paths

---

**Remember**: Keep modules **focused** on one responsibility!
