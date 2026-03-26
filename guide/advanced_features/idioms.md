# Idioms

**Category**: Advanced Features → Best Practices  
**Purpose**: Idiomatic Aria programming style

---

## Overview

Idioms are language-specific patterns that make code more **expressive** and **readable**.

---

## Use `?` for Error Propagation

```aria
// ✅ Idiomatic
func:load_config = Result<Config>() {
    content: string = readFile("config.toml")?;
    config: Config = parse(content)?;
    pass(Ok(config));
}

// ❌ Not idiomatic
func:load_config_verbose = Result<Config>() {
    match readFile("config.toml") {
        Ok(content) => {
            match parse(content) {
                Ok(config) => return Ok(config),
                Err(e) => return Err(e),
            }
        }
        Err(e) => return Err(e),
    }
}
```

---

## Prefer `if let` for Single Pattern Match

```aria
// ✅ Idiomatic
if let Some(user) = find_user(id) {
    print(user.name);
}

// ❌ Verbose
match find_user(id) {
    Some(user) => print(user.name),
    None => {},
}
```

---

## Use Early Returns

```aria
// ✅ Idiomatic
func:validate = Result<NIL>(User:user) {
    if user.name == "" {
        pass(Err("Name required"));
    }
    
    if user.age < 0 {
        pass(Err("Invalid age"));
    }
    
    pass(Ok());
}

// ❌ Nested
func:validate_nested = Result<NIL>(User:user) {
    if user.name != "" {
        if user.age >= 0 {
            pass(Ok());
        } else {
            pass(Err("Invalid age"));
        }
    } else {
        pass(Err("Name required"));
    }
}
```

---

## Destructure in Function Parameters

```aria
// ✅ Idiomatic
func:greet = NIL(User { name, age, .. }: User) {
    print(`Hello &{name}, age &{age}`);
}

// ❌ Verbose
func:greet_verbose = NIL(User:user) {
    print("Hello &{user.name}, age &{user.age}");
}
```

---

## Use Iterator Chains

```aria
// ✅ Idiomatic
total: i32 = numbers
    .filter(|n| n > 0)
    .map(|n| n * 2)
    .sum();

// ❌ Imperative
total: i32 = 0;
till(numbers.length - 1, 1) {
    if numbers[$] > 0 {
        total += numbers[$] * 2;
    }
}
```

---

## Prefer `unwrap_or` Over `match`

```aria
// ✅ Idiomatic
value: i32 = maybe_value.unwrap_or(0);

// ❌ Verbose
value: i32 = match maybe_value {
    Some(v) => v,
    None => 0,
};
```

---

## Use Type Inference

```aria
// ✅ Idiomatic
items := vec![1, 2, 3];
result := calculate_sum(items);

// ❌ Overly explicit (when obvious)
items: Vec<i32> = vec![1, 2, 3];
Result: i32 = calculate_sum(items);
```

---

## Prefer Methods Over Functions

```aria
// ✅ Idiomatic
struct User {
    name: string,
}

impl User {
    func:greet = NIL() {
        print(`Hello, &{self.name}`);
    }
}

user.greet();

// ❌ Less idiomatic
func:greet_user = NIL(User:user) {
    print("Hello, &{user.name}");
}

greet_user(user);
```

---

## Use `defer` for Cleanup

```aria
// ✅ Idiomatic
func:process = Result<NIL>() {
    file: File = open("data.txt")?;
    defer file.close();
    
    lock: Mutex = acquire_lock()?;
    defer lock.unlock();
    
    // Work with resources
    pass(Ok());
}

// ❌ Manual cleanup
func:process_manual = Result<NIL>() {
    file: File = open("data.txt")?;
    lock: Mutex = acquire_lock()?;
    
    // Work with resources
    
    lock.unlock();
    file.close();
    pass(Ok());
}
```

---

## Tuple Returns for Multiple Values

```aria
// ✅ Idiomatic
func:divide = (i32,(int32:a, int32:b)i32) {
    pass((a / b, a % b));
}

(quotient, remainder) = divide(17, 5);

// ❌ Out parameters (C-style)
func:divide_out = NIL(int32:a, int32:b, *i32:quotient, *i32:remainder) {
    *quotient = a / b;
    *remainder = a % b;
}
```

---

## Use Guards in Match

```aria
// ✅ Idiomatic
match value {
    n if n < 0 => print("negative"),
    n if n == 0 => print("zero"),
    n if n % 2 == 0 => print("positive even"),
    _ => print("positive odd"),
}

// ❌ Nested if
match value {
    n => {
        if n < 0 {
            print("negative");
        } else if n == 0 {
            print("zero");
        } else if n % 2 == 0 {
            print("positive even");
        } else {
            print("positive odd");
        }
    }
}
```

---

## Prefer `while let` for Iteration

```aria
// ✅ Idiomatic
while let Some(item) = iterator.next() {
    process(item);
}

// ❌ Verbose
loop {
    match iterator.next() {
        Some(item) => process(item),
        None => break,
    }
}
```

---

## Use Closures for Short Functions

```aria
// ✅ Idiomatic
numbers.map(|x| x * 2);
numbers.filter(|x| x > 0);

// ❌ Verbose (when function is simple)
func:double = int32(int32:x) {
    pass(x * 2);
}

func:is_positive = bool(int32:x) {
    pass(x > 0);
}

numbers.map(double);
numbers.filter(is_positive);
```

---

## String Interpolation

```aria
// ✅ Idiomatic
name: string = "Alice";
age: i32 = 30;
message: string = `Hello, &{name}! You are &{age} years old.`;

// ❌ Concatenation
message: string = "Hello, " ++ name ++ "! You are " ++ age.to_string() ++ " years old.";
```

---

## Explicit Error Types

```aria
// ✅ Idiomatic
enum AppError {
    NotFound,
    PermissionDenied,
    InvalidInput(string),
}

func:fetch_user = Result<User,(int32:id)AppError> {
    // Implementation
}

// ❌ String errors
func:fetch_user_bad = Result<User,(int32:id)string> {
    // Less type-safe
}
```

---

## Use Ranges

```aria
// ✅ Idiomatic
till(9, 1) {
    print($);
}

// ❌ C-style
for i: i32 = 0; i < 10; i += 1 {
    print(i);
}
```

---

## Default Values with `unwrap_or`

```aria
// ✅ Idiomatic
timeout: i32 = config.timeout.unwrap_or(30);
retries: i32 = config.retries.unwrap_or(3);

// ❌ Verbose
timeout: i32 = if config.timeout is Some {
    config.timeout?
} else {
    30
};
```

---

## Slice Patterns

```aria
// ✅ Idiomatic
match items {
    [] => print("empty"),
    [x] => print(`one: &{x}`),
    [x, y] => print(`two: &{x}, &{y}`),
    [first, ...rest] => print(`many, first: &{first}`),
}

// ❌ Manual indexing
if items.len() == 0 {
    print("empty");
} else if items.len() == 1 {
    print("one: &{items[0]}");
} else if items.len() == 2 {
    print("two: &{items[0]}, &{items[1]}");
} else {
    print("many, first: &{items[0]}");
}
```

---

## Named Loop Labels

```aria
// ✅ Idiomatic (for complex loops)
outer: till(9, 1) {
    i = $;
    inner: till(9, 1) {
        j = $;
        if found(i, j) {
            break outer;
        }
    }
}

// Avoid when simple
// ❌ Over-labeled
simple: till(9, 1) {
    print($);
}
```

---

## Related

- [best_practices](best_practices.md) - Best practices
- [common_patterns](common_patterns.md) - Common patterns
- [code_examples](code_examples.md) - Code examples

---

**Remember**: Idiomatic code is **easier to read** and **maintain** for developers familiar with Aria!
