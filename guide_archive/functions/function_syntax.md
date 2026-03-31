# Function Syntax Reference

**Category**: Functions → Reference  
**Purpose**: Complete syntax reference for function declarations

---

## Basic Function Declaration

```aria
func:function_name = ReturnType(Type1:param1, Type2:param2) {
    // Function body
    pass(value);
}
```

---

## Components

### Function Keyword

```aria
// Short form (common)
func:name = NIL() { }

// Long form (rare)
func name() { }
```

### Function Name

```aria
// Convention: snake_case
func:calculate_total = NIL() { }
func:get_user_by_id = NIL() { }

// Single word
func:process = NIL() { }
func:validate = NIL() { }
```

### Parameters

```aria
// No parameters
func:greet = NIL() { }

// Single parameter
func:double = int32(int32:x) { }

// Multiple parameters
func:add = int32(int32:a, int32:b) { }

// Different types
func:format_user = string(int32:id, string:name, bool:active) { }
```

### Return Type

```aria
// No return (implicitly returns unit/void)
func:print_message = NIL() {
    print("Message\n");
}

// Explicit return type
func:calculate = int32() {
    pass(42);
}

// Optional return
func:might_fail = Data?() {
    pass(ERR);
}
```

---

## Generic Functions

```aria
// Single type parameter
func:identity = T(T:value) {
    pass(value);
}

// Multiple type parameters
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}

// With constraints
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then return a; else return b; end
}

// Multiple constraints
func:process = T(T:value)
    where T: Display, T: Clone {
    // Implementation
}
```

---

## Async Functions

```aria
// Basic async
async func:fetch_data = Data() {
    pass(await http_get("/api/data"));
}

// Async with parameters
async func:fetch_user = User?(int32:id) {
    pass(await http_get("/api/user/" + format("{}", id)));
}

// Async generic
async func:fetch = T?(string:url)where T: Deserialize {
    response: Response = await http_get(url);
    pass(response.deserialize());
}
```

---

## Method Syntax

### Instance Methods

```aria
struct User {
    name: string,
    age: i32
}

impl User {
    // Instance method (has self)
    func:greet = NIL() {
        print("Hello, " + self.name + "\n");
    }
    
    // Mutable instance method
    func:grow_older = NIL() {
        self.age = self.age + 1;
    }
}
```

### Associated Functions (Static)

```aria
impl User {
    // No self - associated function
    func:new = User(string:name, int32:age) {
        pass(User{name: name, age: age});
    }
}

// Called with ::
user: User = User::new("Alice", 30);
```

---

## Lambda/Closure Syntax

```aria
// Basic lambda
add: fn(i32, i32) -> i32 = |a, b| a + b;

// With type annotations
add: fn(i32, i32) -> i32 = |a: i32, b: i32| -> i32 {
    pass(a + b);
};

// Multi-line body
process: fn(i32) -> i32 = |x| {
    Result: i32 = x * 2;
    result = result + 1;
    pass(result);
};

// Closure (captures variables)
counter: fn() -> i32 = || {
    count: i32 = 0;
    $count = $count + 1;
    pass($count);
};
```

---

## Function Types

### As Variable Type

```aria
// Function that takes i32, returns i32
callback: fn(i32) -> i32 = double;

// Function that takes nothing, returns string
getter: fn() -> string = get_name;

// Multiple parameters
processor: fn(i32, string, bool) -> Result = process_data;
```

### As Parameter Type

```aria
func:apply = i32)(int32:value, fn(i32:f)-> i32 {
    pass(f(value));
}

func:for_each = NIL([]T:array, fn(T:action)) {
    till(array.length - 1, 1) {
        action(array[$]);
    }
}
```

### As Return Type

```aria
func:make_multiplier = fn(i32)(int32:factor)-> i32 {
    pass(|x| x * factor);
}
```

---

## Visibility Modifiers

```aria
// Public (default in Aria for functions)
pub func:public_function = NIL() { }

// Private (only visible in current module)
func:private_function = NIL() { }

// Package-private
pub(package) fn package_function() { }
```

---

## Complete Examples

### Simple Function

```aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

### Generic Function

```aria
func:swap = (T,(T:a, T:b)T) {
    pass((b, a));
}
```

### Async Function

```aria
async func:fetch_user = User?(int32:id) {
    response: Response = pass await http_get("/api/user/" + format("{}", id));
    user: User = pass response.deserialize();
    pass(user);
}
```

### Method

```aria
struct Calculator;

impl Calculator {
    func:add = int32(int32:a, int32:b) {
        pass(a + b);
    }
}
```

### Higher-Order Function

```aria
func:map = U)([]T:array, fn(T:f)-> []U {
    Result: []U = [];
    till(array.length - 1, 1) {
        result.push(f(array[$]));
    }
    pass(result);
}
```

### Complex Generic with Constraints

```aria
async fn fetch_and_process<T, U>(
    url: string,
    transform: fn(T) -> U
) -> U? 
    where T: Deserialize, 
          U: Serialize {
    
    response: Response = pass await http_get(url);
    data: T = pass response.deserialize();
    Result: U = transform(data);
    pass(result);
}
```

---

## Special Syntax

### Default Parameters (Not Supported)

```aria
// ❌ Wrong: Aria doesn't have default parameters
// fn greet(name: string = "World") { }

// ✅ Right: Use function overloading or separate functions
func:greet = NIL() {
    greet_name("World");
}

func:greet_name = NIL(string:name) {
    print("Hello, " + name + "\n");
}
```

### Variadic Functions (Not Supported)

```aria
// ❌ Wrong: Aria doesn't have variadic functions
// fn sum(numbers: ...i32) -> i32 { }

// ✅ Right: Use array
func:sum = int32([]i32:numbers) {
    total: i32 = 0;
    till(numbers.length - 1, 1) {
        total = total + numbers[$];
    }
    pass(total);
}
```

### Named Arguments (Not Supported)

```aria
// ❌ Wrong: No named arguments
// user = create_user(name: "Alice", age: 30);

// ✅ Right: Positional arguments
user = create_user("Alice", 30);
```

---

## Calling Convention

```aria
// Regular call
Result: i32 = add(5, 10);

// Method call
user.greet();

// Associated function call
user: User = User::new("Alice", 30);

// Generic function call (explicit)
value: i32 = identity::<i32>(42);

// Generic function call (inferred)
value: i32 = identity(42);

// Async function call
user: User = await fetch_user(123);

// Lambda call
Result: i32 = callback(42);
```

---

## Related Topics

- [Function Declaration](function_declaration.md) - Function basics
- [Function Parameters](function_params.md) - Parameter details
- [Function Return Types](function_return_type.md) - Return type details
- [Generic Syntax](generic_syntax.md) - Generic syntax details
- [Lambda Syntax](lambda_syntax.md) - Lambda syntax details

---

**Remember**: Aria's function syntax is **explicit and consistent** - always declare parameter types and return types!
