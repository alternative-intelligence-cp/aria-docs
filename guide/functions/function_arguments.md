# Function Arguments

**Category**: Functions → Calling  
**Concept**: Values passed to functions  
**Relation**: Arguments are the actual values; parameters are the placeholders

---

## Parameters vs Arguments

```aria
// Parameters: name and age
func:greet = NIL(string:name, int32:age) {
    print("Hello, " + name + ", age " + age + "\n");
}

// Arguments: "Alice" and 30
greet("Alice", 30);
```

---

## Positional Arguments

Arguments must match parameter order:

```aria
func:divide = int32(int32:numerator, int32:denominator) {
    pass(numerator / denominator);
}

// Order matters!
Result: i32 = divide(10, 2);  // 5
Result: i32 = divide(2, 10);  // 0
```

---

## Type Matching

Arguments must match parameter types:

```aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

// ✅ Correct: Types match
Result: i32 = add(5, 10);

// ❌ Error: Wrong types
// result = add(5.5, 10);     // f64, not i32
// result = add("5", "10");   // string, not i32
```

---

## Argument Count

Must match parameter count exactly:

```aria
func:calculate = int32(int32:a, int32:b, int32:c) {
    pass(a + b + c);
}

// ✅ Correct: 3 arguments
Result: i32 = calculate(1, 2, 3);

// ❌ Error: Wrong count
// result = calculate(1, 2);        // Too few
// result = calculate(1, 2, 3, 4);  // Too many
```

---

## Literal Arguments

```aria
func:process = NIL(int32:id, string:name, bool:active) {
    // Implementation
}

// Direct literals
process(123, "Alice", true);
```

---

## Variable Arguments

```aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

x: i32 = 5;
y: i32 = 10;

// Pass variables
Result: i32 = add(x, y);
```

---

## Expression Arguments

```aria
func:multiply = int32(int32:a, int32:b) {
    pass(a * b);
}

// Expressions as arguments
Result: i32 = multiply(2 + 3, 4 * 5);  // (5, 20) = 100
Result: i32 = multiply(get_x(), get_y());
```

---

## Array Arguments

```aria
func:sum = int32([]i32:numbers) {
    total: i32 = 0;
    till(numbers.length - 1, 1) {
        total = total + numbers[$];
    }
    pass(total);
}

// Array literal
Result: i32 = sum([1, 2, 3, 4, 5]);

// Array variable
numbers: []i32 = [10, 20, 30];
Result: i32 = sum(numbers);
```

---

## Tuple Arguments

```aria
func:process = NIL((i32, string:pair)) {
    (num, text) = pair;
    print(num + ": " + text + "\n");
}

// Tuple literal
process((42, "answer"));

// Tuple variable
data: (i32, string) = (100, "hundred");
process(data);
```

---

## Struct Arguments

```aria
struct Point {
    x: i32,
    y: i32
}

func:print_point = NIL(Point:p) {
    print("(" + p.x + ", " + p.y + ")\n");
}

// Struct literal
print_point(Point{x: 10, y: 20});

// Struct variable
point: Point = Point{x: 5, y: 15};
print_point(point);
```

---

## Function Arguments (Callbacks)

```aria
func:apply = i32)(int32:value, fn(i32:f)-> i32 {
    pass(f(value));
}

// Lambda as argument
Result: i32 = apply(5, |x| x * 2);

// Named function as argument
func:double = int32(int32:x) { return x * 2; }
Result: i32 = apply(5, double);

// Variable holding function
doubler: fn(i32) -> i32 = |x| x * 2;
Result: i32 = apply(5, doubler);
```

---

## Optional Arguments (using Option)

```aria
func:greet = NIL(string:name, string?:title) {
    when title != nil then
        print(title + " " + name + "\n");
    else
        print(name + "\n");
    end
}

// With value
greet("Alice", "Dr.");

// Without value (nil)
greet("Bob", nil);
```

---

## Mutable Arguments

```aria
func:increment = NIL($i32:count) {
    $count = $count + 1;
}

// Must pass with $
value: i32 = 10;
increment($value);  // Pass by reference
print(value);    // 11
```

---

## Generic Arguments

```aria
func:identity = T(T:value) {
    pass(value);
}

// Type inferred from argument
num: i32 = identity(42);        // T = i32
text: string = identity("hi");   // T = string

// Explicit type
num: i32 = identity::<i32>(42);
```

---

## Evaluation Order

Arguments are evaluated **left to right**:

```aria
func:test = NIL(int32:a, int32:b, int32:c) {
    print(a + ", " + b + ", " + c + "\n");
}

count: i32 = 0;

func:next = int32() {
    $count = $count + 1;
    pass($count);
}

// Evaluates: next(), next(), next()
test(next(), next(), next());  // "1, 2, 3"
```

---

## Argument Passing Semantics

### Copy Types (Pass by Value)

```aria
func:modify = NIL(int32:x) {
    x = x + 10;  // Modifies local copy
}

value: i32 = 5;
modify(value);
print(value);  // Still 5 (not modified)
```

### Move Types (Ownership Transfer)

```aria
func:consume = NIL(Vec<i32>:data) {
    // data is now owned by this function
    print(data.len() + "\n");
}

vec: Vec<i32> = vec![1, 2, 3];
consume(vec);
// vec is no longer valid here - was moved
```

### Reference Types (Pass by Reference)

```aria
func:modify = NIL($Vec<i32>:data) {
    $data.push(4);  // Modifies original
}

vec: Vec<i32> = vec![1, 2, 3];
modify($vec);  // Pass reference
print(vec.len());  // 4 (modified)
```

---

## Argument Patterns

### Spread Arguments (Not Supported)

```aria
// ❌ Wrong: No spread operator
// numbers: []i32 = [1, 2, 3];
// result = add(...numbers);

// ✅ Right: Pass array directly
func:sum = int32([]i32:numbers) { ... }
result = sum([1, 2, 3]);
```

### Named Arguments (Not Supported)

```aria
// ❌ Wrong: No named arguments
// user = create_user(name: "Alice", age: 30);

// ✅ Right: Positional only
user = create_user("Alice", 30);

// Or use struct for clarity
params = UserParams{name: "Alice", age: 30};
user = create_user(params);
```

---

## Method Call Arguments

```aria
struct Calculator;

impl Calculator {
    func:add = int32(int32:a, int32:b) {
        pass(a + b);
    }
}

calc: Calculator = Calculator;

// Method call - arguments after self
Result: i32 = calc.add(5, 10);
```

---

## Common Patterns

### Builder Pattern Arguments

```aria
user: User = UserBuilder::new()
    .name("Alice")
    .email("alice@example.com")
    .age(30)
    .build();
```

### Configuration Struct

```aria
struct Config {
    host: string,
    port: i32,
    timeout: i32,
    retry: bool
}

func:connect = Connection(Config:config) {
    // Use config.host, config.port, etc.
}

// Clear what each argument means
connection = connect(Config{
    host: "localhost",
    port: 8080,
    timeout: 5000,
    retry: true
});
```

---

## Best Practices

### ✅ DO: Use Clear Variable Names

```aria
// Good: Clear what's being passed
user_id: i32 = 123;
user_name: string = "Alice";
create_user(user_id, user_name);
```

### ✅ DO: Use Structs for Many Arguments

```aria
// Good: Clear structure
request: HttpRequest = HttpRequest{
    method: "GET",
    url: "/api/users",
    headers: headers,
    body: nil
};
send_request(request);
```

### ✅ DO: Validate Before Passing

```aria
// Good: Validate first
age: i32 = parse_age(input);
when age >= 0 and age <= 120 then
    set_user_age(user, age);
else
    stderr_write("Invalid age\n");
end
```

### ❌ DON'T: Use Magic Numbers

```aria
// Wrong: What does 5 mean?
process(data, 5, true);

// Right: Named constants
MAX_RETRIES: i32 = 5;
VERBOSE: bool = true;
process(data, MAX_RETRIES, VERBOSE);
```

### ❌ DON'T: Pass Wrong Order

```aria
// Wrong: Easy to mix up
func:create_rectangle = NIL(int32:width, int32:height) { }
create_rectangle(10, 20);  // Which is which?

// Better: Use struct or named types
struct Dimensions {
    width: i32,
    height: i32
}
func:create_rectangle = NIL(Dimensions:dims) { }
```

---

## Related Topics

- [Function Parameters](function_params.md) - Parameter definition
- [Function Declaration](function_declaration.md) - Function basics
- [Function Syntax](function_syntax.md) - Complete syntax

---

**Remember**: Arguments are the **actual values** you pass - they must match the parameter types and order exactly!
