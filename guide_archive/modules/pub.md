# `pub` Keyword

**Category**: Modules → Visibility  
**Syntax**: `pub <item>`  
**Purpose**: Make items publicly accessible

---

## Overview

`pub` makes items **visible** outside their defining module.

---

## Default Visibility

By default, items are **private**:

```aria
func:private_function = NIL() { }     // Private
struct PrivateStruct { }      // Private
const PRIVATE: i32 = 42;      // Private
```

---

## Public Items

```aria
pub func:public_function = NIL() { }     // Public
pub struct PublicStruct { }      // Public
pub const PUBLIC: i32 = 42;      // Public
```

---

## What Can Be Public?

```aria
pub func:function = NIL() { }           // Functions
pub struct Data { }             // Structs
pub enum Status { }             // Enums
pub const MAX: i32 = 100;       // Constants
pub type Alias = i32;           // Type aliases
pub mod submodule { }           // Modules
```

---

## Struct Field Visibility

```aria
pub struct User {
    pub name: string,     // Public field
    pub email: string,    // Public field
    age: i32,             // Private field
    password_hash: string // Private field
}
```

---

## Module Visibility

```aria
// Module is private
mod internal {
    pub func:helper = NIL() { }  // Function is public within module
}

// Module is public
pub mod api {
    pub func:endpoint = NIL() { }  // Public everywhere
    func:internal = NIL() { }      // Private to api module
}
```

---

## Public API Example

```aria
// lib.aria
mod internal {  // Private module
    pub func:helper = int32() {
        pass(42);
    }
}

pub func:public_api = int32() {  // Public function
    pass(internal.helper());  // Can use internal items
}
```

---

## Nested Visibility

```aria
pub mod outer {
    pub func:outer_fn = NIL() { }
    func:outer_private = NIL() { }
    
    pub mod inner {
        pub func:inner_fn = NIL() { }
        func:inner_private = NIL() { }
    }
}

// Accessible
outer.outer_fn();
outer.inner.inner_fn();

// Not accessible
outer.outer_private();      // ❌ Private
outer.inner.inner_private(); // ❌ Private
```

---

## Re-exporting

```aria
mod internal {
    pub func:important = NIL() { }
}

// Re-export to make available at crate root
pub use internal.important;

// Users can now:
// import mycrate;
// mycrate.important();
```

---

## Common Patterns

### Public Interface, Private Implementation

```aria
// Public API
pub func:create_user = User(string:name) {
    return User {
        name: name,
        id: generate_id(),  // Private helper
    };
}

// Private implementation
func:generate_id = int32() {
    // Implementation details
}
```

---

### Struct with Public Fields

```aria
pub struct Config {
    pub host: string,
    pub port: i32,
    pub timeout: i32,
}
```

---

### Struct with Private Fields

```aria
pub struct Connection {
    handle: *void,      // Private - internal detail
    is_open: bool,      // Private - internal state
}

impl Connection {
    pub func:new = Connection() {  // Public constructor
        // Implementation
    }
    
    pub func:query = Result<Data>() {  // Public method
        // Implementation
    }
}
```

---

## Best Practices

### ✅ DO: Expose Minimal API

```aria
pub func:create_user = User() { }  // Public API
func:validate_email = bool() { }   // Private helper
func:hash_password = string() { }  // Private helper
```

### ✅ DO: Keep Implementation Private

```aria
pub struct Database {
    connection_pool: Pool,  // Private - implementation detail
}

impl Database {
    pub func:query = NIL() { }      // Public interface
    func:reconnect = NIL() { }      // Private helper
}
```

### ✅ DO: Public Struct, Private Fields with Methods

```aria
pub struct User {
    name: string,  // Private
    age: i32,      // Private
}

impl User {
    pub func:new = User(string:name, int32:age) { }
    pub func:get_name = string() { }
    pub func:get_age = int32() { }
}
```

### ❌ DON'T: Make Everything Public

```aria
pub func:helper1 = NIL() { }  // ❌ If only used internally
pub func:helper2 = NIL() { }  // ❌ Keep private
pub func:helper3 = NIL() { }  // ❌ Exposes implementation
```

---

## Visibility Levels

```aria
// Private (default) - only in same module
func:private = NIL() { }

// Public - accessible from anywhere
pub func:public = NIL() { }

// Public in crate - accessible within crate only (not exported)
pub(crate) fn crate_public() { }
```

---

## Related

- [public_visibility](public_visibility.md) - Visibility rules
- [mod](mod.md) - Modules overview

---

**Remember**: **Minimize** public API - you can always make private items public later!
