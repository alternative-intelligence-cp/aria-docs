# Comments

**Category**: Advanced Features → Syntax  
**Purpose**: Document code and disable code execution

---

## Overview

Comments explain code and provide documentation without affecting execution.

---

## Single-Line Comments

```aria
// This is a single-line comment
x: i32 = 42;  // Comment after code

// Comments can explain what code does
// Multiple single-line comments
// Can be stacked
```

---

## Multi-Line Comments

```aria
/*
 This is a multi-line comment.
 It can span multiple lines.
 Useful for longer explanations.
*/

func:complex_function = NIL() {
    /* Inline comment */ x: i32 = 42;
}
```

---

## Documentation Comments

```aria
/// This function adds two numbers.
///
/// # Arguments
/// * `a` - The first number
/// * `b` - The second number
///
/// # Returns
/// The sum of `a` and `b`
///
/// # Example
/// ```
/// Result: i32 = add(2, 3);
/// assert(result == 5);
/// ```
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## Module Documentation

```aria
//! This module provides mathematical operations.
//!
//! It includes basic arithmetic functions like
//! addition, subtraction, multiplication, and division.

mod math;

pub func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## Nested Comments

```aria
/*
 Outer comment
 /* Nested comment */
 Still in outer comment
*/

func:example = NIL() {
    /* This /* works */ too */
    x: i32 = 42;
}
```

---

## TODO Comments

```aria
func:incomplete_feature = NIL() {
    // TODO: Implement error handling
    // FIXME: This has a bug with negative numbers
    // NOTE: Consider refactoring this
    // HACK: Temporary workaround
    // XXX: This needs attention
}
```

---

## Common Patterns

### Section Headers

```aria
// ============================================
// File Operations
// ============================================

func:read_file = Result<string>(string:path) {
    // Implementation
}

// ============================================
// Data Processing
// ============================================

func:process_data = Result<NIL>(Data:data) {
    // Implementation
}
```

---

### Explaining Complex Logic

```aria
func:calculate_hash = uint64([]u8:data) {
    // Use FNV-1a hash algorithm
    // https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function
    
    hash: u64 = 0xcbf29ce484222325;  // FNV offset basis
    
    till(data.length - 1, 1) {
        hash ^= data[$];
        hash *= 0x100000001b3;  // FNV prime
    }
    
    pass(hash);
}
```

---

### Disabling Code

```aria
func:debug_mode = NIL() {
    print("Running");
    
    // Temporarily disabled
    // debug_print_state();
    // validate_invariants();
    
    print("Done");
}
```

---

### API Documentation

```aria
/// Represents a user in the system.
///
/// # Fields
/// * `id` - Unique identifier
/// * `name` - User's display name
/// * `email` - Email address
/// * `created_at` - Account creation timestamp
struct User {
    id: i32,
    name: string,
    email: string,
    created_at: i64,
}

impl User {
    /// Creates a new user.
    ///
    /// # Arguments
    /// * `name` - The user's name
    /// * `email` - The user's email
    ///
    /// # Returns
    /// A new `User` instance with generated ID and timestamp
    pub func:new = User(string:name, string:email) {
        return User {
            id: generate_id(),
            name: name,
            email: email,
            created_at: now(),
        };
    }
}
```

---

### Copyright Headers

```aria
/*
 Copyright (c) 2025 Example Corp.
 
 This file is part of the Aria project.
 
 Licensed under the MIT License.
 See LICENSE file in the project root for details.
*/

mod core;
```

---

## Best Practices

### ✅ DO: Explain Why, Not What

```aria
// ❌ Bad - obvious what code does
func:increment = int32(int32:x) {
    // Add 1 to x
    pass(x + 1);
}

// ✅ Good - explains why
func:increment_retry_count = int32(int32:count) {
    // Increment to track failed attempts
    // Max retries is 3 (see config)
    pass(count + 1);
}
```

### ✅ DO: Document Public APIs

```aria
/// Fetches user data from the database.
///
/// # Arguments
/// * `user_id` - The ID of the user to fetch
///
/// # Returns
/// * `Ok(User)` - User data if found
/// * `Err(string)` - Error message if user not found or DB error
///
/// # Example
/// ```
/// user: User = fetch_user(42)?;
/// print(user.name);
/// ```
pub func:fetch_user = Result<User>(int32:user_id) {
    // Implementation
}
```

### ✅ DO: Keep Comments Updated

```aria
// ✅ Good - accurate comment
func:calculate_tax = flt64(flt64:amount) {
    // Tax rate is 8.5% (updated 2025)
    pass(amount * 0.085);
}

// ❌ Bad - outdated comment
func:calculate_tax = flt64(flt64:amount) {
    // Tax rate is 7.5%  <- WRONG!
    pass(amount * 0.085);
}
```

### ⚠️ WARNING: Avoid Over-Commenting

```aria
// ❌ Too many obvious comments
func:add = int32(int32:a, int32:b) {
    // Declare result variable
    Result: i32 = 0;
    
    // Add a to result
    result = result + a;
    
    // Add b to result
    result = result + b;
    
    // Return the result
    pass(result);
}

// ✅ Clean, self-documenting code
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

### ❌ DON'T: Leave Commented-Out Code

```aria
// ❌ Bad - commented-out code
func:process = NIL() {
    do_step1();
    // do_step2();  // <- Remove this or use version control
    // old_implementation();
    do_step3();
}

// ✅ Good - clean code
func:process = NIL() {
    do_step1();
    do_step3();
}
```

---

## Comment Markers

```aria
func:example = NIL() {
    // TODO: Implement feature X
    // FIXME: Bug with negative values
    // HACK: Temporary workaround for issue #123
    // NOTE: This needs refactoring
    // XXX: Critical issue
    // OPTIMIZE: This could be faster
    // DEPRECATED: Use new_function() instead
}
```

---

## Related

- [multiline_comments](multiline_comments.md) - Multi-line comment details

---

**Remember**: Good comments explain **why**, not **what** - let the code show what it does!
