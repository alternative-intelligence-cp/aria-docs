# Function Return Types

**Category**: Functions → Syntax  
**Syntax**: `-> ReturnType`  
**Purpose**: Specify what a function returns

---

## Basic Return Type Syntax

```aria
func:function_name = ReturnType() {
    pass(value);
}
```

---

## No Return Type (Unit)

Functions with no return type implicitly return "unit" (like `void`):

```aria
func:print_message = NIL() {
    print("Hello, World!\n");
}

// Equivalent to:
func:print_message = ()() {
    print("Hello, World!\n");
}
```

---

## Primitive Return Types

```aria
// Integer
func:get_age = int32() {
    pass(25);
}

// Float
func:get_price = flt64() {
    pass(19.99);
}

// Boolean
func:is_valid = bool() {
    pass(true);
}

// Character
func:get_grade = char() {
    pass('A');
}

// String
func:get_name = string() {
    pass("Alice");
}
```

---

## Optional Return Types

Use `?` suffix for functions that might fail:

```aria
// Returns i32 or ERR
func:parse_number = i32?(string:input) {
    when input == "" then
        pass(ERR);
    end
    pass(input.parse());
}

// Usage
Result: i32? = parse_number("42");
when result == ERR then
    stderr_write("Parse failed\n");
else
    print("Number: " + result + "\n");
end
```

---

## Struct Return Types

```aria
struct User {
    name: string,
    age: i32
}

func:create_user = User(string:name, int32:age) {
    pass(User{name: name, age: age});
}

// Usage
user: User = create_user("Alice", 30);
```

---

## Tuple Return Types

Return multiple values:

```aria
func:divide_with_remainder = (i32,(int32:a, int32:b)i32) {
    quotient: i32 = a / b;
    remainder: i32 = a % b;
    pass((quotient, remainder));
}

// Usage
(quot, rem): (i32, i32) = divide_with_remainder(17, 5);
print(quot);  // 3
print(rem);   // 2
```

---

## Array Return Types

```aria
func:get_primes = []i32() {
    pass([2, 3, 5, 7, 11]);
}

// Usage
primes: []i32 = get_primes();
```

---

## Enum Return Types

```aria
enum Status {
    Success,
    Error(string),
    Pending
}

func:check_status = Status() {
    pass(Status::Success);
}
```

---

## Generic Return Types

```aria
func:identity = T(T:value) {
    pass(value);
}

// T inferred from usage
num: i32 = identity(42);      // Returns i32
text: string = identity("hi"); // Returns string
```

---

## Function Return Types

Functions can return other functions:

```aria
func:make_multiplier = fn(i32)(int32:factor)-> i32 {
    pass(|x| x * factor);
}

// Usage
times_3: fn(i32) -> i32 = make_multiplier(3);
Result: i32 = times_3(5);  // 15
```

---

## Async Return Types

```aria
async func:fetch_data = Data() {
    pass(await http_get("/api/data"));
}

// Returns Future<Data>
// Must await to get Data
data: Data = await fetch_data();
```

---

## Optional with Generics

```aria
func:find = T?([]T:array, T:target) {
    till(array.length - 1, 1) {
        when array[$] == target then
            pass(array[$]);
        end
    }
    pass(nil);
}

// Usage
numbers: []i32 = [1, 2, 3, 4, 5];
found: i32? = find(numbers, 3);  // Returns 3
not_found: i32? = find(numbers, 99);  // Returns ERR
```

---

## Complex Return Types

```aria
// Result type (custom)
enum Result<T, E> {
    Ok(T),
    Err(E)
}

func:safe_divide = Result<int32, string>(int32:a, int32:b) {
    when b == 0 then
        fail("Division by zero");
    end
    pass(a / b);
}
```

---

## Return Statement

### Explicit Return

```aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

### Multiple Returns

```aria
func:abs = int32(int32:x) {
    when x < 0 then
        pass(-x);
    else
        pass(x);
    end
}
```

### Early Return

```aria
func:validate_age = bool(int32:age) {
    when age < 0 then
        pass(false);  // Early exit
    end
    
    when age > 120 then
        pass(false);  // Early exit
    end
    
    pass(true);
}
```

---

## Return Type Inference (Limited)

Aria requires **explicit return types** on functions:

```aria
// ❌ Wrong: No return type inference for functions
// fn add(a: i32, b: i32) {
//     return a + b;
// }

// ✅ Right: Explicit return type
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

**Exception**: Lambdas can infer return types:

```aria
// Lambda: Return type inferred
add = |a, b| a + b;

// Function: Must specify
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## Never Return Type

For functions that never return:

```aria
func:panic = !(string:message) {
    stderr_write("PANIC: " + message + "\n");
    exit(1);
    // Never reaches here
}

func:infinite_loop = !() {
    while true {
        process_events();
    }
    // Never returns
}
```

---

## Unit Return Type (Explicit)

```aria
// Implicit unit
func:log = NIL(string:message) {
    print(message + "\n");
}

// Explicit unit
func:log = ()(string:message) {
    print(message + "\n");
    pass(());
}
```

---

## Best Practices

### ✅ DO: Use Descriptive Return Types

```aria
// Good: Clear what's returned
func:get_user_count = int32() { }
func:is_authenticated = bool() { }
func:load_config = Config() { }
```

### ✅ DO: Use Option for Failable Operations

```aria
// Good: Clear it might fail
func:find_user = User?(int32:id) {
    // Returns User or ERR
}
```

### ✅ DO: Use Tuples for Multiple Related Values

```aria
// Good: Related values
func:get_dimensions = (i32,()i32) {
    pass((width, height));
}
```

### ✅ DO: Use Struct for Multiple Unrelated Values

```aria
// Better: Named fields
struct UserInfo {
    name: string,
    age: i32,
    email: string
}

func:get_user_info = UserInfo() {
    pass(UserInfo{...});
}
```

### ❌ DON'T: Return Complex Tuples

```aria
// Wrong: Hard to remember order
func:get_stats = (i32,()i32, f64, bool, string) {
    pass((count, total, avg, active, status));
}

// Right: Use struct
struct Stats {
    count: i32,
    total: i32,
    avg: f64,
    active: bool,
    status: string
}

func:get_stats = Stats() { }
```

### ❌ DON'T: Forget Return Statement

```aria
// Wrong: Missing return
func:add = int32(int32:a, int32:b) {
    sum: i32 = a + b;
    // Error: No return statement
}

// Right: Always return
func:add = int32(int32:a, int32:b) {
    sum: i32 = a + b;
    pass(sum);
}
```

---

## Common Patterns

### Factory Functions

```aria
func:new_user = User(string:name) {
    return User{
        name: name,
        created_at: Time::now(),
        id: generate_id()
    };
}
```

### Conversion Functions

```aria
func:to_string = string(int32:value) {
    pass(format("{}", value));
}

func:from_string = i32?(string:s) {
    pass(s.parse());
}
```

### Validation Functions

```aria
func:validate_email = bool(string:email) {
    pass(email.contains("@") and email.contains("."));
}
```

### Builder Pattern

```aria
func:build = Config() {
    return Config{
        host: self.host,
        port: self.port,
        timeout: self.timeout
    };
}
```

---

## Related Topics

- [Function Declaration](function_declaration.md) - Function basics
- [Function Syntax](function_syntax.md) - Complete syntax
- [Pass Keyword](pass_keyword.md) - Handling optional returns
- [Fail Keyword](fail_keyword.md) - Error handling

---

**Remember**: Aria requires **explicit return types** for all functions (except lambdas). Be clear about what your function returns!
