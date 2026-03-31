# Generic Structs

**Category**: Functions → Generics  
**Concept**: Structs that work with any type  
**Philosophy**: Reusable data structures

---

## What Are Generic Structs?

**Generic structs** are data structures that can hold **any type** instead of a specific type. They let you create reusable containers, wrappers, and data structures.

---

## Basic Generic Struct

```aria
struct Box<T> {
    value: T
}

// Use with i32
int_box: Box<i32> = Box{value: 42};

// Use with string
str_box: Box<string> = Box{value: "hello"};
```

---

## Multiple Type Parameters

```aria
struct Pair<T, U> {
    first: T,
    second: U
}

// Different types
pair: Pair<i32, string> = Pair{
    first: 42,
    second: "answer"
};

// Same types
coords: Pair<i32, i32> = Pair{
    first: 10,
    second: 20
};
```

---

## Implementing Methods

### Generic Implementation

```aria
struct Box<T> {
    value: T
}

impl<T> Box<T> {
    // Constructor
    func:new = Box<T>(T:value) {
        pass(Box{value: value});
    }
    
    // Getter
    func:get = T() {
        pass(self.value);
    }
    
    // Setter
    func:set = NIL(T:new_value) {
        self.value = new_value;
    }
}

// Usage
box: Box<i32> = Box::new(42);
value: i32 = box.get();
box.set(100);
```

### Methods with Additional Type Parameters

```aria
impl<T> Box<T> {
    // Method with its own type parameter U
    func:map = U)(fn(T:f)-> Box<U> {
        pass(Box{value: f(self.value)});
    }
}

// Usage
int_box: Box<i32> = Box::new(42);
str_box: Box<string> = int_box.map(|x| format("{}", x));
```

---

## Type-Specific Implementations

```aria
struct Box<T> {
    value: T
}

// Implementation for ALL types
impl<T> Box<T> {
    func:get = T() {
        pass(self.value);
    }
}

// Implementation ONLY for i32
impl Box<i32> {
    func:is_even = bool() {
        pass(self.value % 2 == 0);
    }
}

// Implementation ONLY for string
impl Box<string> {
    func:uppercase = string() {
        pass(self.value.to_uppercase());
    }
}

// Usage
int_box: Box<i32> = Box{value: 42};
int_box.is_even();  // Available for Box<i32>

str_box: Box<string> = Box{value: "hello"};
str_box.uppercase();  // Available for Box<string>
```

---

## Constrained Generic Structs

```aria
// T must implement Display
struct Printable<T> where T: Display {
    value: T
}

impl<T> Printable<T> where T: Display {
    func:print = NIL() {
        print(self.value + "\n");
    }
}

// Works - i32 implements Display
p1: Printable<i32> = Printable{value: 42};
p1.print();

// Works - string implements Display
p2: Printable<string> = Printable{value: "hello"};
p2.print();
```

---

## Common Generic Struct Patterns

### Option Type

```aria
enum Option<T> {
    Some(T),
    None
}

impl<T> Option<T> {
    func:is_some = bool() {
        return match self {
            Some(_) => true,
            None => false
        };
    }
    
    func:unwrap = T() {
        return match self {
            Some(value) => value,
            None => panic("Called unwrap on None")
        };
    }
    
    func:unwrap_or = T(T:default) {
        return match self {
            Some(value) => value,
            None => default
        };
    }
}
```

### Result Type

```aria
enum Result<T, E> {
    Ok(T),
    Err(E)
}

impl<T, E> Result<T, E> {
    func:is_ok = bool() {
        return match self {
            Ok(_) => true,
            Err(_) => false
        };
    }
    
    func:unwrap = T() {
        return match self {
            Ok(value) => value,
            Err(_) => panic("Called unwrap on Err")
        };
    }
}
```

### Vector/Array

```aria
struct Vec<T> {
    data: []T,
    len: usize,
    capacity: usize
}

impl<T> Vec<T> {
    func:new = Vec<T>() {
        return Vec{
            data: [],
            len: 0,
            capacity: 0
        };
    }
    
    func:push = NIL(T:item) {
        // Implementation
        self.len = self.len + 1;
    }
    
    func:get = T?(uint64:index) {
        when index >= self.len then
            pass(nil);
        end
        pass(self.data[index]);
    }
}
```

### Map/Dictionary

```aria
struct Map<K, V> {
    entries: []Entry<K, V>
}

struct Entry<K, V> {
    key: K,
    value: V
}

impl<K, V> Map<K, V> where K: Hashable {
    func:new = Map<K,()V> {
        pass(Map{entries: []});
    }
    
    func:insert = NIL(K:key, V:value) {
        // Implementation
    }
    
    func:get = V?(K:key) {
        // Implementation
    }
}
```

---

## Nested Generics

```aria
struct Container<T> {
    items: []T
}

// Container of containers
nested: Container<Container<i32>> = Container{
    items: [
        Container{items: [1, 2, 3]},
        Container{items: [4, 5, 6]}
    ]
};

// Box of optional values
maybe_box: Box<Option<i32>> = Box{
    value: Some(42)
};
```

---

## Generic Tuple Structs

```aria
struct Point<T>(T, T);

// Create instances
int_point: Point<i32> = Point(10, 20);
float_point: Point<f64> = Point(3.14, 2.71);

// Access fields
x: i32 = int_point.0;
y: i32 = int_point.1;
```

---

## Lifetime Parameters (If Supported)

```aria
struct Reference<'a, T> {
    data: &'a T
}

impl<'a, T> Reference<'a, T> {
    func:get = 'a->()T {
        pass(self.data);
    }
}
```

---

## Associated Types in Generic Structs

```aria
trait Iterator {
    type Item;
    
    func:next = Result<Self::Item>()
}

struct VecIterator<T> {
    data: Vec<T>,
    index: usize
}

impl<T> Iterator for VecIterator<T> {
    type Item = T;
    
    func:next = Result<T>() {
        when self.index >= self.data.len() then
            pass(None);
        end
        
        item: T = self.data.get(self.index);
        self.index = self.index + 1;
        pass(Some(item));
    }
}
```

---

## Best Practices

### ✅ DO: Use Generic Structs for Containers

```aria
// Good: Reusable container
struct Stack<T> {
    items: []T
}

impl<T> Stack<T> {
    func:push = NIL(T:item) { }
    func:pop = T?() { }
}
```

### ✅ DO: Add Constraints When Needed

```aria
// Good: Clear requirements
struct SortedSet<T> where T: Comparable {
    items: []T
}

impl<T> SortedSet<T> where T: Comparable {
    func:insert = NIL(T:item) {
        // Can use comparison operators
    }
}
```

### ✅ DO: Use Descriptive Type Names

```aria
// Good: Clear purpose
struct Cache<Key, Value> {
    data: Map<Key, Value>
}
```

### ❌ DON'T: Overuse Generics

```aria
// Wrong: Doesn't need to be generic
struct UserId<T> {
    value: T  // Should always be i32
}

// Right: Be specific
struct UserId {
    value: i32
}
```

### ❌ DON'T: Make Every Field Generic

```aria
// Wrong: Too generic
struct Config<T, U, V, W> {
    host: T,
    port: U,
    timeout: V,
    retry: W
}

// Right: Only genericize what varies
struct Config {
    host: string,
    port: i32,
    timeout: i32,
    retry: bool
}
```

---

## Real-World Examples

### Generic Cache

```aria
struct Cache<K, V> {
    data: Map<K, V>,
    max_size: usize
}

impl<K, V> Cache<K, V> where K: Hashable {
    func:new = Cache<K,(uint64:max_size)V> {
        return Cache{
            data: Map::new(),
            max_size: max_size
        };
    }
    
    func:get = V?(K:key) {
        pass(self.data.get(key));
    }
    
    func:set = NIL(K:key, V:value) {
        when self.data.len() >= self.max_size then
            self.evict_oldest();
        end
        self.data.insert(key, value);
    }
}

// Usage
user_cache: Cache<i32, User> = Cache::new(100);
session_cache: Cache<string, Session> = Cache::new(1000);
```

### Generic Builder

```aria
struct Builder<T> {
    items: []T,
    validators: []fn(T) -> bool
}

impl<T> Builder<T> {
    func:new = Builder<T>() {
        return Builder{
            items: [],
            validators: []
        };
    }
    
    func:add = Builder<T>(T:item) {
        till(self.validators.length - 1, 1) {
            validator: fn(T) -> bool = self.validators[$];
            when !validator(item) then
                panic("Validation failed");
            end
        }
        self.items.push(item);
        pass(self);
    }
    
    func:validate = bool)(fn(T:f)-> Builder<T> {
        self.validators.push(f);
        pass(self);
    }
    
    func:build = []T() {
        pass(self.items);
    }
}
```

### Generic Tree

```aria
struct Tree<T> {
    value: T,
    left: Box<Tree<T>>?,
    right: Box<Tree<T>>?
}

impl<T> Tree<T> {
    func:leaf = Tree<T>(T:value) {
        return Tree{
            value: value,
            left: nil,
            right: nil
        };
    }
    
    func:insert = NIL(T:value)where T: Comparable {
        when value < self.value then
            when self.left == nil then
                self.left = Some(Box::new(Tree::leaf(value)));
            else
                self.left.unwrap().insert(value);
            end
        else
            // Insert right
        end
    }
}
```

---

## Related Topics

- [Generic Functions](generic_functions.md) - Generic functions
- [Generics](generics.md) - Generic overview
- [Generic Syntax](generic_syntax.md) - Syntax reference
- [Type Inference](type_inference.md) - Type parameter inference

---

**Remember**: Generic structs are **type-safe templates** - write the structure once, use it with any type!
