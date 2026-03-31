# Function Parameters

**Category**: Functions → Syntax  
**Concept**: Inputs to functions  
**Philosophy**: Explicit types, clear intent

---

## Basic Parameter Syntax

```aria
func:function_name = NIL(Type:param_name) {
    // Use param_name here
}
```

---

## Single Parameter

```aria
func:greet = NIL(string:name) {
    print("Hello, " + name + "\n");
}

// Call
greet("Alice");
```

---

## Multiple Parameters

```aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

// Call with positional arguments
Result: i32 = add(5, 10);
```

---

## Parameter Types

### Primitives

```aria
fn process(
    id: i32,
    price: f64,
    active: bool,
    letter: char
) {
    // Implementation
}
```

### Complex Types

```aria
fn format_user(
    user: User,
    options: FormatOptions,
    template: string
) -> string {
    // Implementation
}
```

### References

```aria
func:modify = NIL($Vec<i32>:data) {
    // data is mutable reference
    $data.push(42);
}

func:read = NIL(Vec<i32>:data) {
    // data is owned/copied
    print(data.len() + "\n");
}
```

---

## Optional Parameters (via Option Type)

Aria doesn't have default parameters, use `Option<T>`:

```aria
func:greet = NIL(string:name, string?:title) {
    when title != nil then
        print(title + " " + name + "\n");
    else
        print(name + "\n");
    end
}

// Call
greet("Alice", "Dr.");
greet("Bob", nil);
```

---

## Multiple Parameters of Same Type

```aria
func:calculate_area = int32(int32:width, int32:height) {
    pass(width * height);
}

// Must specify each type
// ❌ Wrong: fn calculate_area(width, height: i32)
// ✅ Right: fn calculate_area(width: i32, height: i32)
```

---

## Array Parameters

```aria
func:sum = int32([]i32:numbers) {
    total: i32 = 0;
    till(numbers.length - 1, 1) {
        num: i32 = numbers[$];
        total = total + num;
    }
    pass(total);
}

// Call
Result: i32 = sum([1, 2, 3, 4, 5]);
```

---

## Tuple Parameters

```aria
func:process_pair = NIL((i32, string:pair)) {
    (num, text) = pair;
    print(num + ": " + text + "\n");
}

// Call
process_pair((42, "answer"));
```

---

## Struct Parameters

```aria
struct Point {
    x: i32,
    y: i32
}

func:distance = flt64(Point:p) {
    pass(sqrt((p.x * p.x + p.y * p.y) as f64));
}

// Call
point: Point = Point{x: 3, y: 4};
dist: f64 = distance(point);
```

---

## Function Parameters (Higher-Order)

```aria
func:apply = i32)(int32:value, fn(i32:transform)-> i32 {
    pass(transform(value));
}

// Call with lambda
Result: i32 = apply(5, |x| x * 2);

// Call with named function
func:double = int32(int32:x) { return x * 2; }
Result: i32 = apply(5, double);
```

---

## Generic Parameters

```aria
func:identity = T(T:value) {
    pass(value);
}

// T inferred from argument
num: i32 = identity(42);
text: string = identity("hello");
```

---

## Mutable Parameters

```aria
func:increment = NIL($i32:count) {
    $count = $count + 1;
}

// Usage
value: i32 = 10;
increment($value);
print(value);  // 11
```

---

## Self Parameter (Methods)

```aria
struct Counter {
    count: i32
}

impl Counter {
    // Immutable self
    func:get = int32() {
        pass(self.count);
    }
    
    // Mutable self
    func:increment = NIL() {
        self.count = self.count + 1;
    }
    
    // Consuming self
    func:into_i32 = int32() {
        pass(self.count);
    }
}
```

---

## Parameter Patterns

### Destructuring Tuple Parameter

```aria
func:process = NIL((x, y): (i32, i32)) -> i32 {
    pass(x + y);
}

// Call
Result: i32 = process((5, 10));  // 15
```

### Underscore for Unused

```aria
func:get_first = NIL((first, _): (i32, i32)) -> i32 {
    pass(first);
}
```

---

## Parameter Order Convention

```aria
// Good: Most important/required first
fn create_user(
    name: string,      // Required
    email: string,     // Required
    age: i32?,         // Optional
    bio: string?       // Optional
) -> User {
    // Implementation
}
```

---

## Variadic Parameters (Not Supported)

```aria
// ❌ Wrong: No variadic parameters
// fn sum(numbers: ...i32) -> i32 { }

// ✅ Right: Use array
func:sum = int32([]i32:numbers) {
    total: i32 = 0;
    till(numbers.length - 1, 1) {
        num: i32 = numbers[$];
        total = total + num;
    }
    pass(total);
}

// Call
Result: i32 = sum([1, 2, 3, 4, 5]);
```

---

## Named Arguments (Not Supported)

```aria
// ❌ Wrong: No named arguments
// user = create_user(name: "Alice", age: 30);

// ✅ Right: Use struct for clarity
struct UserParams {
    name: string,
    email: string,
    age: i32?
}

func:create_user = User(UserParams:params) {
    // Implementation
}

// Call
user = create_user(UserParams{
    name: "Alice",
    email: "alice@example.com",
    age: 30
});
```

---

## Default Parameters (Not Supported)

```aria
// ❌ Wrong: No default parameters
// fn greet(name: string = "World") { }

// ✅ Right: Use separate functions or Option
func:greet = NIL() {
    greet_name("World");
}

func:greet_name = NIL(string:name) {
    print("Hello, " + name + "\n");
}

// Or use Option
func:greet_optional = NIL(string?:name) {
    actual_name: string = name ?? "World";
    print("Hello, " + actual_name + "\n");
}
```

---

## Best Practices

### ✅ DO: Use Descriptive Names

```aria
// Good: Clear parameter names
func:create_invoice = NIL(int32:customer_id, flt64:amount, Date:date) { }
```

### ✅ DO: Keep Parameter Count Low

```aria
// Good: 3-4 parameters
func:format_address = NIL(string:street, string:city, string:zip) { }

// Better for many parameters: use struct
struct Address {
    street: string,
    city: string,
    state: string,
    zip: string,
    country: string
}

func:format_address = NIL(Address:addr) { }
```

### ✅ DO: Use Types That Prevent Errors

```aria
// Good: Type safety
func:set_age = NIL(uint8:age) { }  // Can't be negative

// Instead of:
// fn set_age(age: i32) { }  // Could be negative
```

### ❌ DON'T: Use Too Many Parameters

```aria
// Wrong: Too many parameters
fn create_user(
    name: string,
    email: string,
    age: i32,
    address: string,
    phone: string,
    bio: string,
    avatar: string,
    role: string
) { }

// Right: Use struct
struct UserData {
    name: string,
    email: string,
    age: i32,
    // ... other fields
}

func:create_user = NIL(UserData:data) { }
```

### ❌ DON'T: Use Ambiguous Types

```aria
// Wrong: What do these booleans mean?
func:configure = NIL(true, false, true) { }

// Right: Use enum or named struct
enum Mode { Active, Inactive }
func:configure = NIL(Mode:mode, bool:debug, bool:verbose) { }
```

---

## Related Topics

- [Function Arguments](function_arguments.md) - Argument passing
- [Function Declaration](function_declaration.md) - Function basics
- [Function Syntax](function_syntax.md) - Complete syntax reference

---

**Remember**: Aria requires **explicit types** for all parameters - no inference at function boundaries!
