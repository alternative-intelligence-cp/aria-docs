# Modules

**Category**: Modules → Overview  
**Purpose**: Code organization and namespacing

---

## Overview

Modules organize code into logical units, provide **namespacing**, and control **visibility**.

---

## Module Basics

### File as Module

```aria
// math.npk - automatically becomes module 'math'

pub func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

pub func:multiply = int32(int32:a, int32:b) {
    pass(a * b);
}
```

### Using a Module

```aria
// main.npk
use math.*;

Result: i32 = add(5, 3);  // 8
```

---

## Module Declaration

### Inline Module

```aria
mod math {
    pub func:add = int32(int32:a, int32:b) {
        pass(a + b);
    }
}

// Use in same file
Result: i32 = math.add(5, 3);
```

### Module from File

```aria
// Reference external file
mod math;  // Loads math.npk

// Or from directory
mod utils;  // Loads utils/mod.npk
```

---

## Module Hierarchy

```
project/
├── main.npk
├── math.npk
└── utils/
    ├── mod.npk
    ├── string.npk
    └── array.npk
```

```aria
// main.npk
use math.*;
use utils.*;
use utils.string.*;

add(1, 2);
reverse("hello");
```

---

## Visibility

### Private by Default

```aria
// math.npk
func:helper = int32() {  // Private
    pass(42);
}

pub func:public_fn = int32() {  // Public
    pass(helper());
}
```

### Public Items

```aria
pub func:function = NIL() { }       // Public function
pub struct Data { }         // Public struct
pub const MAX: i32 = 100;   // Public constant
```

---

## Re-exports

```aria
// lib.npk
mod internal;

// Re-export from internal module
pub use internal.important_function;
pub use internal.ImportantType;
```

---

## Namespace Path

```aria
// Full path
Result: i32 = std.math.sqrt(16);

// Import to shorten
use std.math;
Result: i32 = math.sqrt(16);

// Import specific function
use std.math.sqrt;
Result: i32 = sqrt(16);
```

---

## Common Patterns

### Library Structure

```
mylib/
├── lib.npk          # Main entry point
├── core/
│   ├── mod.npk
│   ├── types.npk
│   └── traits.npk
└── utils/
    ├── mod.npk
    └── helpers.npk
```

```aria
// lib.npk
pub mod core;
pub mod utils;

// Re-export commonly used items
pub use core.types.MainType;
pub use utils.helpers.helper_function;
```

### Feature Modules

```aria
mod database {
    pub func:connect = Connection() { }
    pub func:query = Result<Data>() { }
}

mod api {
    pub func:start_server = Result<NIL>() { }
    pub func:handle_request = Response() { }
}

mod auth {
    pub func:login = Result<User>() { }
    pub func:verify_token = bool() { }
}
```

---

## Best Practices

### ✅ DO: Organize by Feature

```aria
// Good structure
auth/
  login.npk
  register.npk
  tokens.npk
database/
  connection.npk
  queries.npk
api/
  routes.npk
  handlers.npk
```

### ✅ DO: Use Clear Names

```aria
mod user_management;  // ✅ Clear
mod um;               // ❌ Unclear
```

### ✅ DO: Control Visibility

```aria
// Public interface
pub func:create_user = User() { }

// Private implementation
func:validate_email = bool() { }  // Hidden
func:hash_password = string() { } // Hidden
```

### ❌ DON'T: Deep Nesting

```aria
// Too deep
use app.modules.features.user.auth.login.handlers;  // ❌

// Better - flatten
use app.auth.login;  // ✅
```

---

## Related

- [mod keyword](mod_keyword.md) - Module declaration
- [import](import.md) - Importing modules
- [pub](pub.md) - Public visibility
- [use](use.md) - Bringing items into scope

---

**Remember**: Modules provide **organization** and **encapsulation**!
