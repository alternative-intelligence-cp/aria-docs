# Best Practices

**Category**: Advanced Features → Best Practices  
**Purpose**: Guidelines for writing idiomatic Aria code

---

## Overview

Follow these best practices to write **clean**, **maintainable**, and **efficient** Aria code.

---

## Code Organization

### File Structure

```aria
// File: user_service.aria

// 1. Imports
use std.io.{File, readFile, writeFile};
use std.collections.HashMap;

// 2. Constants
USER_MAX_AGE: i32 = 150;
DEFAULT_TIMEOUT: i32 = 30;

// 3. Types
struct User {
    id: i32,
    name: string,
    age: i32,
}

// 4. Functions
pub func:create_user = Result<User>(string:name, int32:age) {
    // Implementation
}

// 5. Main (if applicable)
func:main = NIL() {
    // Entry point
}
```

---

### Module Organization

```aria
// Good module structure:
project/
  src/
    main.aria
    lib.aria
    user/
      mod.aria
      service.aria
      repository.aria
    product/
      mod.aria
      catalog.aria
```

---

## Naming Conventions

### Variables and Functions

```aria
// ✅ snake_case for variables and functions
user_count: i32 = 0;
max_retry_attempts: i32 = 3;

func:calculate_total_price = flt64([]Item:items) {
    // Implementation
}

func:fetch_user_by_id = Result<User>(int32:id) {
    // Implementation
}
```

---

### Types and Traits

```aria
// ✅ PascalCase for types, structs, enums, traits
struct UserProfile {
    name: string,
    email: string,
}

enum Status {
    Active,
    Inactive,
    Pending,
}

trait Displayable {
    func:display = string;()
}
```

---

### Constants

```aria
// ✅ SCREAMING_SNAKE_CASE for constants
MAX_CONNECTIONS: i32 = 100;
DEFAULT_TIMEOUT_MS: i32 = 5000;
API_BASE_URL: string = "https://api.example.com";
```

---

## Error Handling

### Use Result for Fallible Operations

```aria
// ✅ Return Result for operations that can fail
func:read_config = Result<Config>(string:path) {
    content: string = readFile(path)?;
    config: Config = parse_config(content)?;
    pass(config);
}

// ❌ Don't panic for expected errors
func:bad_read_config = Config(string:path) {
    content: string = readFile(path).expect("Failed to read");  // ❌
    // ...
}
```

---

### Provide Context

```aria
// ✅ Add context to errors
func:load_user_data = Result<UserData>(int32:user_id) {
    user: User = fetch_user(user_id)
        .context(`Failed to fetch user &{user_id}`)?;
    
    profile: Profile = fetch_profile(user_id)
        .context(`Failed to fetch profile for user &{user_id}`)?;
    
    pass(UserData { user, profile });
}
```

---

## Resource Management

### Use RAII

```aria
// ✅ Resources cleaned up automatically
func:process_file = Result<NIL>(string:path) {
    file: File = open(path)?;
    defer file.close();  // Automatically called
    
    data: string = file.read_all()?;
    process(data);
    
    pass();
}
```

---

### Avoid Manual Memory Management

```aria
// ❌ Manual allocation/deallocation
func:bad_buffer = NIL() {
    buffer: *u8 = alloc(1024);
    // Use buffer
    free(buffer);  // Easy to forget!
}

// ✅ Use safe containers
func:good_buffer = NIL() {
    buffer: []u8 = [0; 1024];  // Automatically managed
    // Use buffer
}  // Automatically freed
```

---

## Performance

### Prefer Immutability

```aria
// ✅ Immutable by default
func:calculate = int32([]i32:data) {
    total: i32 = data.sum();  // data not modified
    pass(total * 2);
}

// Only use mut when necessary
func:process = NIL([]mut i32:data) {
    till(data.len() - 1, 1) {
        data[$] *= 2;  // Mutation needed
    }
}
```

---

### Avoid Unnecessary Allocations

```aria
// ❌ Inefficient - creates temp string each iteration
func:bad_concat = string([]string:words) {
    Result: string = "";
    till(words.length - 1, 1) {
        result = result ++ words[$];  // Reallocates each time
    }
    pass(result);
}

// ✅ Efficient - pre-allocate capacity
func:good_concat = string([]string:words) {
    total_len: i32 = words.map(|w| w.len()).sum();
    Result: StringBuilder = StringBuilder.with_capacity(total_len);
    till(words.length - 1, 1) {
        result.append(words[$]);
    }
    pass(result.to_string());
}
```

---

## Code Style

### Keep Functions Small

```aria
// ✅ Small, focused functions
func:validate_user = Result<NIL>(User:user) {
    validate_name(user.name)?;
    validate_email(user.email)?;
    validate_age(user.age)?;
    pass();
}

func:validate_name = Result<NIL>(string:name) {
    if name.len() == 0 {
        fail("Name cannot be empty");
    }
    if name.len() > 100 {
        fail("Name too long");
    }
    pass();
}
```

---

### Use Type Inference

```aria
// ✅ Let compiler infer obvious types
x := 42;
name := "Alice";
items := [1, 2, 3, 4, 5];

// ✅ Be explicit for clarity when needed
timeout: Duration = Duration.seconds(30);
config: Config = load_config()?;
```

---

### Avoid Deep Nesting

```aria
// ❌ Too much nesting
func:bad_nested = string(?User:user) {
    if user is Some {
        if user?.age > 18 {
            if user?.name.len() > 0 {
                pass(user?.name);
            }
        }
    }
    pass("Unknown");
}

// ✅ Early returns
func:good_early_return = string(?User:user) {
    if user is None {
        pass("Unknown");
    }
    
    if user?.age <= 18 {
        pass("Minor");
    }
    
    if user?.name.len() == 0 {
        pass("Unnamed");
    }
    
    pass(user?.name);
}
```

---

## Documentation

### Document Public APIs

```aria
/// Fetches user data from the database.
///
/// # Arguments
/// * `user_id` - The unique identifier of the user
///
/// # Returns
/// * `Ok(User)` - The user data if found
/// * `Err(string)` - Error message if user not found or database error
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

---

### Use Meaningful Comments

```aria
// ✅ Explain why, not what
func:calculate_discount = flt64(flt64:price) {
    // Apply 10% discount for bulk orders
    // Based on business rule BR-2025-01
    pass(price * 0.9);
}

// ❌ Obvious comment
func:add = int32(int32:a, int32:b) {
    // Add a and b together  ← Obvious!
    pass(a + b);
}
```

---

## Testing

### Write Tests for Public APIs

```aria
#[test]
func:test_calculate_total = NIL() {
    items: []Item = [
        Item { price: 10.0 },
        Item { price: 20.0 },
    ];
    
    total: f64 = calculate_total(items);
    
    assert(total == 30.0);
}

#[test]
func:test_validate_email_invalid = NIL() {
    Result: Result<void> = validate_email("invalid");
    assert(result.is_err());
}
```

---

## Security

### Validate Input

```aria
// ✅ Always validate user input
func:create_user = Result<User>(string:name, int32:age) {
    if name.len() == 0 {
        fail("Name cannot be empty");
    }
    
    if age < 0 || age > 150 {
        fail("Invalid age");
    }
    
    // Create user
}
```

---

### Avoid SQL Injection

```aria
// ❌ SQL injection vulnerability
func:bad_query = Result<User>(string:username) {
    query: string = `SELECT * FROM users WHERE name = '&{username}'`;
    pass(db.execute(query));  // ❌ Vulnerable!
}

// ✅ Use parameterized queries
func:good_query = Result<User>(string:username) {
    query: string = "SELECT * FROM users WHERE name = ?";
    pass(db.execute(query, username));  // ✅ Safe
}
```

---

## Concurrency

### Prefer Message Passing

```aria
// ✅ Use channels for communication
func:producer_consumer = NIL() {
    channel: Channel<i32> = Channel.new();
    
    // Producer thread
    Thread.spawn(|| {
        till(99, 1) {
            channel.send($);
        }
    });
    
    // Consumer thread
    Thread.spawn(|| {
        while value = channel.recv() {
            process(value);
        }
    });
}
```

---

### Use Proper Synchronization

```aria
// ✅ Use Mutex for shared state
shared_counter: Mutex<i32> = Mutex.new(0);

func:increment = NIL() {
    lock = shared_counter.lock();
    *lock += 1;
}  // Automatically unlocked
```

---

## Related

- [common_patterns](common_patterns.md) - Common code patterns
- [idioms](idioms.md) - Aria idioms
- [error_handling](error_handling.md) - Error handling patterns

---

**Remember**: These practices help you write **cleaner**, **safer**, and more **maintainable** code!
