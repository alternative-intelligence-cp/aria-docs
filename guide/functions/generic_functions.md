# Generic Functions

**Category**: Functions → Generics  
**Syntax**: `fn name<T>(param: T) -> T`  
**Philosophy**: Write once, use with any type - safely

---

## What are Generic Functions?

**Generic functions** work with **any type** instead of a specific type. They let you write code once that works for `i32`, `string`, `User`, or any other type.

---

## Basic Syntax

```aria
// Generic function with type parameter T
func:identity = T(T:value) {
    pass(value);
}

// Use with i32
num: i32 = identity<i32>(42);

// Use with string
text: string = identity<string>("hello");
```

---

## Type Parameters

### Single Type Parameter

```aria
func:first = T([]T:array) {
    pass(array[0]);
}

numbers: []i32 = [1, 2, 3];
first_num: i32 = first<i32>(numbers);  // 1

names: []string = ["Alice", "Bob"];
first_name: string = first<string>(names);  // "Alice"
```

### Multiple Type Parameters

```aria
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}

// i32 and string
Result: (i32, string) = pair<i32, string>(42, "answer");

// string and bool
flag: (string, bool) = pair<string, bool>("active", true);
```

---

## Type Inference

The compiler can often **infer** generic types from arguments:

```aria
func:swap = (T,(T:a, T:b)T) {
    pass((b, a));
}

// Explicit types
Result: (i32, i32) = swap<i32>(1, 2);

// Inferred from arguments
Result: (i32, i32) = swap(1, 2);  // T inferred as i32
```

---

## Common Generic Functions

### Container Operations

```aria
// Generic array length
func:len = uint64([]T:array) {
    pass(array.length());
}

// Generic array reverse
func:reverse = []T([]T:array) {
    reversed: []T = [];
    i = array.len() - 1;
    till(array.len() - 1, 1) {
        reversed.push(array[i - $]);
    }
    pass(reversed);
}
```

### Option/Result Handling

```aria
// Generic unwrap with default
func:unwrap_or = T(T?:value, T:default) {
    when value == ERR then
        pass(default);
    end
    pass(value);
}

// Usage
num: tbb32? = parse("42");
Result: tbb32 = unwrap_or(num, 0);  // 42 or 0 if parse failed
```

### Comparison

```aria
// Generic max function
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then
        pass(a);
    else
        pass(b);
    end
}

largest: i32 = max(10, 20);  // 20
biggest: f64 = max(3.14, 2.71);  // 3.14
```

---

## Type Constraints

### Trait Bounds

```aria
// T must implement Display trait
func:print_value = NIL(T:value)where T: Display {
    print(value + "\n");
}

// T must implement Comparable
func:is_sorted = bool([]T:array)where T: Comparable {
    till(array.len() - 2, 1) {
        i: i32 = $;
        when array[i] > array[i + 1] then
            pass(false);
        end
    }
    pass(true);
}
```

### Multiple Constraints

```aria
// T must be both Comparable and Hashable
func:unique_sorted = []T([]T:array)
    where T: Comparable, T: Hashable {
    
    seen: Set<T> = Set::new();
    unique: []T = [];
    
    till(array.length - 1, 1) {
        item: T = array[$];
        when !seen.contains(item) then
            seen.insert(item);
            unique.push(item);
        end
    }
    
    pass(sort(unique));
}
```

---

## Generic Structs with Generic Methods

```aria
struct Box<T> {
    value: T
}

impl<T> Box<T> {
    // Generic method on generic type
    func:new = Box<T>(T:value) {
        pass(Box{value: value});
    }
    
    func:get = T() {
        pass(self.value);
    }
    
    func:set = NIL(T:new_value) {
        self.value = new_value;
    }
    
    // Method with additional type parameter
    func:map = U)(fn(T:f)-> Box<U> {
        pass(Box<U>{value: f(self.value)});
    }
}

// Usage
int_box: Box<i32> = Box::new(42);
string_box: Box<string> = int_box.map(|x: i32| -> string {
    pass(format("{}", x));
});
```

---

## Monomorphization

Aria uses **monomorphization** - the compiler generates a separate version of the generic function for each type used:

```aria
func:add = T(T:a, T:b) {
    pass(a + b);
}

// Compiler generates:
// fn add_i32(a: i32, b: i32) -> i32 { return a + b; }
// fn add_f64(a: f64, b: f64) -> f64 { return a + b; }

x: i32 = add(1, 2);      // Calls add_i32
y: f64 = add(1.5, 2.5);  // Calls add_f64
```

**Benefits**:
- **Zero runtime overhead** - as fast as hand-written specific functions
- **No dynamic dispatch** - direct function calls
- **Full optimization** - compiler optimizes each version

**Trade-off**:
- **Larger binary size** - each type gets its own copy of the function

---

## Advanced Patterns

### Generic Builder Pattern

```aria
struct Builder<T> {
    items: []T
}

impl<T> Builder<T> {
    func:new = Builder<T>() {
        pass(Builder{items: []});
    }
    
    func:add = Builder<T>(T:item) {
        self.items.push(item);
        pass(self);
    }
    
    func:build = []T() {
        pass(self.items);
    }
}

// Usage
numbers: []i32 = Builder<i32>::new()
    .add(1)
    .add(2)
    .add(3)
    .build();
```

### Generic Iterator

```aria
func:filter = bool)([]T:array, fn(T:predicate)-> []T {
    Result: []T = [];
    till(array.length - 1, 1) {
        item: T = array[$];
        when predicate(item) then
            result.push(item);
        end
    }
    pass(result);
}

func:map = U)([]T:array, fn(T:transform)-> []U {
    Result: []U = [];
    till(array.length - 1, 1) {
        item: T = array[$];
        result.push(transform(item));
    }
    pass(result);
}

// Usage
numbers: []i32 = [1, 2, 3, 4, 5];
evens: []i32 = filter(numbers, |x| x % 2 == 0);
doubled: []i32 = map(evens, |x| x * 2);
```

---

## Best Practices

### ✅ DO: Use Generics for Container Operations

```aria
// Good: Generic, reusable
func:last = T([]T:array) {
    pass(array[array.len() - 1]);
}
```

### ✅ DO: Add Type Constraints When Needed

```aria
// Good: Clear requirements
func:sum = T([]T:array)where T: Numeric {
    total: T = 0;
    till(array.length - 1, 1) {
        item: T = array[$];
        total = total + item;
    }
    pass(total);
}
```

### ✅ DO: Use Descriptive Type Parameter Names

```aria
// Good for single type
func:wrap = Box<T>(T:value) { ... }

// Good for multiple types
func:convert = Output(Input:value) { ... }

// Good for specific domains
func:store = NIL(Key:key, Value:value) { ... }
```

### ❌ DON'T: Overuse Generics

```aria
// Wrong: Unnecessarily generic
func:print_number = NIL(T:x) {  // Only works with numbers anyway!
    print(x + "\n");
}

// Right: Be specific when you can
func:print_number = NIL(int32:x) {
    print(x + "\n");
}
```

### ❌ DON'T: Forget Type Constraints

```aria
// Wrong: Won't compile - T might not support +
func:add = T(T:a, T:b) {
    pass(a + b);  // Error if T isn't numeric
}

// Right: Add constraint
func:add = T(T:a, T:b)where T: Add {
    pass(a + b);
}
```

---

## Real-World Examples

### Generic Cache

```aria
struct Cache<K, V> {
    data: Map<K, V>
}

impl<K, V> Cache<K, V> where K: Hashable {
    func:new = Cache<K,()V> {
        pass(Cache{data: Map::new()});
    }
    
    func:get = V?(K:key) {
        pass(self.data.get(key));
    }
    
    func:set = NIL(K:key, V:value) {
        self.data.insert(key, value);
    }
    
    func:has = bool(K:key) {
        pass(self.data.contains_key(key));
    }
}

// Usage with different types
user_cache: Cache<i32, User> = Cache::new();
session_cache: Cache<string, Session> = Cache::new();
```

### Generic Result Type

```aria
enum Result<T, E> {
    Ok(T),
    Err(E)
}

impl<T, E> Result<T, E> {
    func:is_ok = bool() {
        return match self {
            Ok(_) => true,
            Err(_) => false,
        };
    }
    
    func:unwrap = T() {
        return match self {
            Ok(value) => value,
            Err(err) => panic("Called unwrap on Err"),
        };
    }
    
    func:map = U)(fn(T:f)-> Result<U, E> {
        return match self {
            Ok(value) => Ok(f(value)),
            Err(err) => Err(err),
        };
    }
}
```

### Generic API Client

```aria
struct ApiClient<T> {
    base_url: string,
    response_type: type(T)
}

impl<T> ApiClient<T> where T: Deserializable {
    func:new = ApiClient<T>(string:base_url) {
        pass(ApiClient{base_url: base_url, response_type: T});
    }
    
    async func:get = T?(string:endpoint) {
        url: string = self.base_url + endpoint;
        response: Response = pass await http_get(url);
        data: T = pass response.deserialize<T>();
        pass(data);
    }
    
    async func:post = T?(string:endpoint, T:payload) {
        url: string = self.base_url + endpoint;
        response: Response = pass await http_post(url, payload);
        data: T = pass response.deserialize<T>();
        pass(data);
    }
}

// Usage
user_api: ApiClient<User> = ApiClient::new("https://api.example.com");
user: User = await user_api.get("/users/123");
```

---

## Generic Type Aliases

```aria
// Create aliases for common generic combinations
type IntPair = (i32, i32);
type StringMap<T> = Map<string, T>;
type Result<T> = Result<T, Error>;

// Usage
coords: IntPair = (10, 20);
config: StringMap<string> = Map::new();
data: Result<Data> = fetch_data();
```

---

## Related Topics

- [Generic Syntax](generic_syntax.md) - Detailed syntax reference
- [Monomorphization](monomorphization.md) - How generics compile
- [Generic Structs](generic_structs.md) - Generic data structures
- [Type Inference](type_inference.md) - Type parameter inference
- [Generic Star Prefix](generic_star_prefix.md) - Alternative syntax

---

**Remember**: Generics let you **write once, use everywhere** - but the compiler generates optimized code for each type. Zero abstraction overhead!
