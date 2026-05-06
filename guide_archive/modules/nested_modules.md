# Nested Modules

**Category**: Modules → Structure  
**Purpose**: Organizing modules hierarchically

---

## Overview

Modules can contain other modules, creating a **hierarchy** for better organization.

---

## Inline Nested Modules

```aria
mod outer {
    pub func:outer_function = NIL() {
        print("Outer");
    }
    
    pub mod inner {
        pub func:inner_function = NIL() {
            print("Inner");
        }
    }
}

// Access
outer.outer_function();
outer.inner.inner_function();
```

---

## File-Based Nested Modules

### Directory Structure

```
database/
├── mod.npk          # database module root
├── connection.npk   # database.connection
└── models/
    ├── mod.npk      # database.models
    ├── user.npk     # database.models.user
    └── post.npk     # database.models.post
```

### Module Declarations

```aria
// database/mod.npk
pub mod connection;
pub mod models;

pub func:init = NIL() {
    connection.connect()?;
}
```

```aria
// database/models/mod.npk
pub mod user;
pub mod post;
```

---

## Accessing Nested Modules

```aria
// Import parent
use database;

// Access nested
user: database.models.user.User = database.models.user.create();

// Or import nested directly
use database.models.user;

user: user.User = user.create();
```

---

## Visibility in Nested Modules

```aria
mod parent {
    func:private_parent = NIL() { }      // Private to parent
    pub func:public_parent = NIL() { }   // Public
    
    pub mod child {
        func:private_child = NIL() { }   // Private to child
        pub func:public_child = NIL() {  // Public
            // Can access parent's private items
            super.private_parent();
        }
    }
}

// From outside
parent.public_parent();           // ✅ OK
parent.child.public_child();      // ✅ OK
parent.private_parent();          // ❌ Error
parent.child.private_child();     // ❌ Error
```

---

## Common Patterns

### Feature Modules

```
app/
├── auth/
│   ├── mod.npk
│   ├── login.npk
│   ├── register.npk
│   └── tokens/
│       ├── mod.npk
│       ├── jwt.npk
│       └── refresh.npk
├── database/
│   ├── mod.npk
│   ├── connection.npk
│   └── models/
│       ├── mod.npk
│       ├── user.npk
│       └── post.npk
└── api/
    ├── mod.npk
    ├── routes.npk
    └── handlers/
        ├── mod.npk
        ├── users.npk
        └── posts.npk
```

---

### Library Organization

```aria
// lib.npk - Main entry point
pub mod core {
    pub mod types;
    pub mod traits;
}

pub mod utils {
    pub mod string;
    pub mod math;
}

pub mod io {
    pub mod file;
    pub mod network;
}

// Re-export commonly used items
pub use core.types.MainType;
pub use utils.string.format;
```

---

## Best Practices

### ✅ DO: Group Related Functionality

```
user_management/
  mod.npk
  create.npk       # Create user
  update.npk       # Update user
  delete.npk       # Delete user
  validation/       # Nested validation
    mod.npk
    email.npk
    password.npk
```

### ✅ DO: Keep Hierarchy Shallow

```
// Good - 2-3 levels
app.database.models.user  // ✅

// Too deep
app.modules.features.user.management.operations.create  // ❌
```

### ✅ DO: Use mod.npk for Public API

```aria
// database/mod.npk
mod connection;   // Private
mod pool;         // Private

pub mod models;   // Public nested module

// Re-export public API
pub use connection.connect;
pub use connection.disconnect;
```

### ❌ DON'T: Over-nest

```aria
mod level1 {
    mod level2 {
        mod level3 {
            mod level4 {
                mod level5 {  // ❌ Way too deep!
                }
            }
        }
    }
}
```

---

## Accessing Parent Items

```aria
mod parent {
    pub const CONFIG: string = "config";
    
    pub mod child {
        pub func:use_parent_config = NIL() {
            // Access parent's item
            config: string = super.CONFIG;
        }
        
        pub mod grandchild {
            pub func:use_grandparent_config = NIL() {
                // Access grandparent's item
                config: string = super.super.CONFIG;
            }
        }
    }
}
```

---

## Re-exporting from Nested Modules

```aria
// lib.npk
mod internal {
    pub mod deeply {
        pub mod nested {
            pub struct Important { }
        }
    }
}

// Flatten for users
pub use internal.deeply.nested.Important;

// Users can now do:
// import mylib;
// item: mylib.Important = ...
// Instead of: mylib.internal.deeply.nested.Important
```

---

## Related

- [mod](mod.md) - Modules overview
- [module_paths](module_paths.md) - Module path navigation
- [pub](pub.md) - Visibility control

---

**Remember**: Keep nesting **shallow** - aim for 2-3 levels max!
