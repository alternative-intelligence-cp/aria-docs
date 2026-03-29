# Public Visibility

**Category**: Modules → Visibility  
**Purpose**: Understanding visibility and access control

---

## Overview

Visibility controls which items can be accessed from outside their defining module.

---

## Visibility Levels

### Private (Default)

```aria
func:function = NIL() { }           // Private to module
struct Data { }             // Private to module
const VALUE: i32 = 42;      // Private to module
```

### Public

```aria
pub func:function = NIL() { }       // Public everywhere
pub struct Data { }         // Public everywhere
pub const VALUE: i32 = 42;  // Public everywhere
```

### Public in Crate

```aria
pub(crate) fn function() { }  // Public within crate only
```

---

## Module Boundaries

```aria
mod database {
    func:private_helper = NIL() { }    // Only in 'database'
    pub func:connect = NIL() { }       // Available outside 'database'
}

// From outside
database.connect();      // ✅ OK - public
database.private_helper(); // ❌ Error - private
```

---

## Struct Visibility

### Public Struct, Public Fields

```aria
pub struct Config {
    pub host: string,
    pub port: i32,
}

config: Config = Config {
    host: "localhost",
    port: 8080
};
```

---

### Public Struct, Private Fields

```aria
pub struct User {
    name: string,     // Private
    age: i32,         // Private
}

impl User {
    pub func:new = User(string:name, int32:age) {
        pass(User { name: name, age: age });
    }
    
    pub func:get_name = string() {
        pass(self.name);
    }
}

// From outside
user: User = User.new("Alice", 30);  // ✅ Public constructor
name: string = user.get_name();      // ✅ Public method
direct: string = user.name;          // ❌ Error - private field
```

---

### Private Struct

```aria
struct InternalData {  // Private struct
    value: i32
}

pub func:create = InternalData() {  // ❌ Error - can't expose private type
}
```

---

## Nested Module Visibility

```aria
pub mod outer {
    pub func:outer_public = NIL() { }
    func:outer_private = NIL() { }
    
    pub mod inner {
        pub func:inner_public = NIL() { }
        func:inner_private = NIL() { }
        
        pub func:use_outer = NIL() {
            super.outer_public();   // ✅ OK
            super.outer_private();  // ✅ OK - same parent module
        }
    }
}

// From outside
outer.outer_public();        // ✅ OK
outer.inner.inner_public();  // ✅ OK
outer.outer_private();       // ❌ Error
outer.inner.inner_private(); // ❌ Error
```

---

## Re-exporting

```aria
// internal.aria
pub func:helper = NIL() { }

// lib.aria
mod internal;

// Re-export to public API
pub use internal.helper;

// Users can now:
// import mylib;
// mylib.helper();
```

---

## Crate-Level Visibility

```aria
// Only public within your crate, not to external users
pub(crate) fn internal_helper() { }
pub(crate) struct InternalType { }
```

---

## Common Patterns

### Public API, Private Implementation

```aria
// lib.aria
mod database {
    struct Connection { }  // Private
    
    func:connect_internal = Connection() { }  // Private
}

pub func:connect_to_database = Result<NIL>() {
    database.connect_internal()?;  // Use private items
    pass();
}
```

---

### Encapsulation

```aria
pub struct Account {
    balance: i32,  // Private - can't be directly modified
}

impl Account {
    pub func:new = Account() {
        pass(Account { balance: 0 });
    }
    
    pub func:deposit = NIL(int32:amount) {
        self.balance += amount;
    }
    
    pub func:get_balance = int32() {
        pass(self.balance);
    }
}

// Users can't directly modify balance
account: Account = Account.new();
account.deposit(100);           // ✅ OK
balance: i32 = account.get_balance();  // ✅ OK
account.balance = 1000;         // ❌ Error - private field
```

---

### Selective Exposure

```aria
mod utils {
    pub func:public_util = NIL() { }    // Public
    func:internal_util = NIL() { }      // Private
    func:helper = NIL() { }             // Private
}

// Only expose specific items
pub use utils.public_util;
```

---

## Visibility Rules

### Functions

```aria
func:private_fn = NIL() { }      // Private to module
pub func:public_fn = NIL() { }   // Public everywhere
```

### Structs

```aria
struct PrivateStruct { }        // Private struct
pub struct PublicStruct { }     // Public struct

pub struct MixedStruct {
    pub public_field: i32,      // Public field
    private_field: string,      // Private field
}
```

### Enums

```aria
enum PrivateEnum { A, B }       // Private enum
pub enum PublicEnum { A, B }    // Public enum (all variants public)
```

### Constants

```aria
const PRIVATE: i32 = 42;        // Private constant
pub const PUBLIC: i32 = 42;     // Public constant
```

---

## Best Practices

### ✅ DO: Minimize Public Surface

```aria
// Expose only what's needed
pub func:api_function = NIL() { }

// Keep helpers private
func:helper1 = NIL() { }
func:helper2 = NIL() { }
func:validate = NIL() { }
```

### ✅ DO: Encapsulate State

```aria
pub struct Server {
    socket: Socket,     // Private - implementation detail
    is_running: bool,   // Private - internal state
}

impl Server {
    pub func:start = NIL() { }  // Public interface
    pub func:stop = NIL() { }   // Public interface
    func:cleanup = NIL() { }    // Private helper
}
```

### ✅ DO: Use Re-exports for Clean API

```aria
// lib.aria
mod internal {
    pub func:important_function = NIL() { }
}

// Clean public API
pub use internal.important_function;
```

### ❌ DON'T: Expose Implementation Details

```aria
pub func:process_user = NIL(User:user) {
    internal_validate(user);  // ❌ Don't expose this
    internal_save(user);      // ❌ Or this
}

pub func:internal_validate = NIL(User:user) { }  // ❌ Should be private
pub func:internal_save = NIL(User:user) { }      // ❌ Should be private
```

---

## Visibility Checklist

- ✅ Is this item part of your public API?
- ✅ Do external users need direct access?
- ✅ Can you provide a better abstraction?
- ✅ Will making this public lock you into maintaining it?

**When in doubt, keep it private** - you can always make things public later!

---

## Related

- [pub](pub.md) - pub keyword
- [mod](mod.md) - Modules overview
- [use](use.md) - Importing items

---

**Remember**: **Private by default** - make things public only when necessary!
