# Generic Syntax Reference

**Category**: Functions → Generics → Reference  
**Basic Form**: `<T>`  
**Purpose**: Complete syntax reference for generic types

---

## Basic Generic Syntax

### Function Declaration

```aria
// Single type parameter
func:identity = T(T:value) {
    pass(value);
}

// Multiple type parameters
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}
```

### Struct Declaration

```aria
// Generic struct
struct Box<T> {
    value: T
}

// Multiple type parameters
struct Pair<T, U> {
    first: T,
    second: U
}
```

### Enum Declaration

```aria
// Generic enum
enum Option<T> {
    Some(T),
    None
}

// Multiple type parameters
enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

---

## Type Parameter Naming

### Convention: Single Letters

```aria
// T for single generic type
func:wrap = Box<T>(T:value) { }

// T, U for two types
func:convert = U(T:input) { }

// K, V for Key/Value
func:insert = NIL(K:key, V:value) { }
```

### Descriptive Names (Rare)

```aria
// More descriptive when needed
func:transform = Output(Input:data) { }

func:cache = NIL(Key:k, Value:v) { }
```

---

## Trait Bounds (Constraints)

### Single Constraint

```aria
// Using where clause
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then return a; else return b; end
}

// Inline constraint (alternative)
func:max = T(T:a, T:b) {
    when a > b then return a; else return b; end
}
```

### Multiple Constraints

```aria
// Multiple where clauses
func:process = T(T:value)
    where T: Display, 
          T: Clone {
    copy: T = value.clone();
    print(copy + "\n");
    pass(copy);
}

// Inline with +
func:process = T(T:value) {
    copy: T = value.clone();
    print(copy + "\n");
    pass(copy);
}
```

### Multiple Type Parameters with Constraints

```aria
func:transform = U(T:input)
    where T: Serializable,
          U: Deserializable {
    data: []u8 = input.serialize();
    pass(U::deserialize(data));
}
```

---

## Type Argument Syntax

### Explicit Type Arguments

```aria
// Turbofish syntax ::<>
Result: i32 = identity::<i32>(42);

// Multiple arguments
pair: (i32, string) = pair::<i32, string>(42, "hello");
```

### Type Inference

```aria
// Inferred from argument
Result: i32 = identity(42);  // T inferred as i32

// Inferred from return type
num: i32 = parse("42");  // T inferred as i32
```

### Partial Inference (Underscore)

```aria
// Specify some, infer others
Result: string = convert::<_, string>(42);  // From inferred, To specified
```

---

## Generic Struct Usage

### Instantiation

```aria
struct Box<T> {
    value: T
}

// Explicit type
int_box: Box<i32> = Box{value: 42};

// Type inferred from field
str_box = Box{value: "hello"};  // Box<string>
```

### Implementation Blocks

```aria
// Generic implementation
impl<T> Box<T> {
    func:new = Box<T>(T:value) {
        pass(Box{value: value});
    }
    
    func:get = T() {
        pass(self.value);
    }
}

// Implementation for specific type
impl Box<i32> {
    func:is_even = bool() {
        pass(self.value % 2 == 0);
    }
}

// Implementation with constraints
impl<T> Box<T> where T: Display {
    func:print = NIL() {
        print(self.value + "\n");
    }
}
```

---

## Generic Enum Usage

```aria
enum Option<T> {
    Some(T),
    None
}

// Create variants
some_value: Option<i32> = Some(42);
no_value: Option<i32> = None;

// Match on generic enum
Result: i32 = match some_value {
    Some(x) => x,
    None => 0
};
```

---

## Lifetime Parameters (If Supported)

```aria
// Lifetime parameter (rare in Aria)
func:longest = 'a->('a string->:s1, 'a string->:s2)string {
    when s1.len() > s2.len() then
        pass(s1);
    else
        pass(s2);
    end
}
```

---

## Associated Types

```aria
trait Iterator {
    type Item;  // Associated type
    
    func:next = Result<Self::Item>()
}

struct Counter;

impl Iterator for Counter {
    type Item = i32;  // Concrete type
    
    func:next = Result<int32>() {
        // Implementation
    }
}
```

---

## Generic Type Aliases

```aria
// Type alias with generics
type Result<T> = Result<T, Error>;

// Usage
func:parse = Result<int32>(string:input) {
    // Returns Result<i32, Error>
}

// Multiple parameters
type HashMap<K, V> = Map<K, V>;
```

---

## Nested Generics

```aria
// Box of box
nested: Box<Box<i32>> = Box{value: Box{value: 42}};

// Vec of optional values
items: Vec<Option<i32>> = vec![Some(1), None, Some(3)];

// Map with generic values
cache: Map<string, Box<User>> = Map::new();
```

---

## Generic Function Pointers

```aria
// Generic function type
transformer: fn<T>(T) -> T;  // Function that works with any T

// Concrete function type
doubler: fn(i32) -> i32 = |x| x * 2;
```

---

## Special Syntax

### Star Prefix (Alternative Syntax)

```aria
// Alternative generic syntax (if supported)
func:identity = T(T:value) {
    pass(value);
}

// Equivalent to:
func:identity = T(T:value) {
    pass(value);
}
```

### Const Generics (If Supported)

```aria
// Generic over constant values
struct Array<T, const N: usize> {
    data: [T; N]
}

// Usage
arr: Array<i32, 10> = Array::new();
```

---

## Complete Examples

### Generic Container

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
    }
    
    func:get = T?(uint64:index) {
        when index >= self.len then
            pass(nil);
        end
        pass(self.data[index]);
    }
}

// Usage
numbers: Vec<i32> = Vec::new();
numbers.push(1);
numbers.push(2);
```

### Generic Function with Multiple Constraints

```aria
func:unique_sorted = []T([]T:array)
    where T: Comparable,
          T: Hashable,
          T: Clone {
    
    seen: Set<T> = Set::new();
    unique: []T = [];
    
    till(array.length - 1, 1) {
        item: T = array[$];
        when !seen.contains(item.clone()) then
            seen.insert(item.clone());
            unique.push(item);
        end
    }
    
    pass(sort(unique));
}

// Usage
numbers: []i32 = [3, 1, 4, 1, 5, 9, 2, 6, 5];
Result: []i32 = unique_sorted(numbers);  // [1, 2, 3, 4, 5, 6, 9]
```

---

## Best Practices

### ✅ DO: Use where Clauses for Complex Constraints

```aria
// Good: Readable
func:process = Result(T:input, U:output)
    where T: Serializable + Clone,
          U: Deserializable + Display {
    // Implementation
}
```

### ✅ DO: Use Type Inference When Obvious

```aria
// Good: Types clear from context
numbers: []i32 = [1, 2, 3];
doubled: []i32 = numbers.map(|x| x * 2);
```

### ✅ DO: Use Descriptive Names for Domain Types

```aria
// Good: Clear purpose
func:cache_lookup = Value?(Key:key) { }
```

### ❌ DON'T: Over-specify Types

```aria
// Wrong: Too explicit
Result: i32 = identity::<i32>(42_i32);

// Right: Let inference work
Result: i32 = identity(42);
```

### ❌ DON'T: Use Too Many Type Parameters

```aria
// Wrong: Confusing
func:complex = X(T:a, U:b, V:c, W:d) { }

// Right: Simplify or use structs
struct Config<T, U, V, W> { }
func:complex = X(Config:config, ...) { }
```

---

## Related Topics

- [Generic Functions](generic_functions.md) - Generic function details
- [Generics](generics.md) - Generic concept overview
- [Type Inference](type_inference.md) - Type parameter inference
- [Monomorphization](monomorphization.md) - How generics compile

---

**Remember**: Generic syntax is **angle brackets** `<T>` for declaration, **turbofish** `::<T>` for explicit instantiation!
