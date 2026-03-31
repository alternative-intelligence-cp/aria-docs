# Generics Overview

**Category**: Functions → Generics  
**Concept**: Write code once, use with any type  
**Philosophy**: Type safety without code duplication

---

## What are Generics?

**Generics** let you write functions, structs, and types that work with **any type** instead of a specific type. They're like templates that get filled in with actual types when you use them.

---

## Why Generics?

### Without Generics (Code Duplication)

```aria
func:max_i32 = int32(int32:a, int32:b) {
    when a > b then return a; else return b; end
}

func:max_f64 = flt64(flt64:a, flt64:b) {
    when a > b then return a; else return b; end
}

func:max_string = string(string:a, string:b) {
    when a > b then return a; else return b; end
}

// And so on for every type...
```

### With Generics (Write Once)

```aria
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then return a; else return b; end
}

// Works with any comparable type!
x: i32 = max(10, 20);
y: f64 = max(3.14, 2.71);
z: string = max("alice", "bob");
```

---

## Generic Type Parameters

### Syntax

```aria
func:function_name = T(T:param) {
    // T is a type parameter
    // Can be any type
}
```

### Single Type Parameter

```aria
func:identity = T(T:value) {
    pass(value);
}

num: i32 = identity(42);
text: string = identity("hello");
```

### Multiple Type Parameters

```aria
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}

Result: (i32, string) = pair(42, "answer");
```

### Descriptive Names

```aria
// Single generic: T (type)
func:wrap = Box<T>(T:value) { ... }

// Input/Output: In, Out
func:convert = Out(In:value) { ... }

// Key/Value: K, V
func:insert = NIL(K:key, V:value) { ... }

// Element: T or E
func:sum = T([]T:array) { ... }
```

---

## Type Constraints

### Trait Bounds

```aria
// T must implement Display
func:print = NIL(T:value)where T: Display {
    print(value + "\n");
}

// T must implement Comparable
func:sort = NIL([]T:array)where T: Comparable {
    // Can use >, <, ==, etc.
}

// T must implement both traits
func:process = NIL(T:value)where T: Display, T: Clone {
    copy: T = value.clone();
    print(copy + "\n");
}
```

### Multiple Constraints

```aria
// Method 1: where clause
func:unique = []T([]T:array)
    where T: Comparable, T: Hashable {
    // Implementation
}

// Method 2: inline (less common)
func:unique = []T([]T:array) {
    // Implementation
}
```

---

## Generic Structs

```aria
struct Box<T> {
    value: T
}

impl<T> Box<T> {
    func:new = Box<T>(T:value) {
        pass(Box{value: value});
    }
    
    func:get = T() {
        pass(self.value);
    }
}

int_box: Box<i32> = Box::new(42);
str_box: Box<string> = Box::new("hello");
```

---

## Generic Enums

```aria
enum Option<T> {
    Some(T),
    None
}

enum Result<T, E> {
    Ok(T),
    Err(E)
}

// Usage
maybe_num: Option<i32> = Some(42);
Result: Result<string, Error> = Ok("success");
```

---

## Type Inference

The compiler can often infer generic types:

```aria
func:first = T([]T:array) {
    pass(array[0]);
}

// Explicit type
value: i32 = first<i32>([1, 2, 3]);

// Inferred from array type
value: i32 = first([1, 2, 3]);  // T inferred as i32

// Inferred from return type
numbers: []i32 = [1, 2, 3];
value: i32 = first(numbers);  // T inferred as i32
```

---

## Monomorphization

Aria generates **specialized code** for each type you use:

```aria
func:add = T(T:a, T:b) {
    pass(a + b);
}

x: i32 = add(1, 2);
y: f64 = add(1.5, 2.5);
```

**Compiler generates**:
```aria
// Generated for i32
func:add_i32 = int32(int32:a, int32:b) {
    pass(a + b);
}

// Generated for f64
func:add_f64 = flt64(flt64:a, flt64:b) {
    pass(a + b);
}
```

**Benefits**:
- ✅ **Zero runtime overhead** - as fast as hand-written code
- ✅ **Full optimization** - each version optimized independently
- ✅ **No dynamic dispatch** - direct function calls

**Trade-off**:
- ⚠️ **Larger binary** - separate code for each type

---

## Common Generic Patterns

### Container Types

```aria
struct Vec<T> {
    data: []T,
    len: usize,
    capacity: usize
}

struct Map<K, V> {
    entries: []Entry<K, V>
}

struct Set<T> {
    data: Map<T, bool>
}
```

### Iterator Pattern

```aria
func:map = U)([]T:array, fn(T:f)-> []U {
    Result: []U = [];
    till(array.length - 1, 1) {
        item: T = array[$];
        result.push(f(item));
    }
    pass(result);
}

numbers: []i32 = [1, 2, 3];
doubled: []i32 = map(numbers, |x| x * 2);
```

### Builder Pattern

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
```

---

## Best Practices

### ✅ DO: Use Generics for Reusable Code

```aria
// Good: Works with any type
func:swap = (T,(T:a, T:b)T) {
    pass((b, a));
}
```

### ✅ DO: Add Constraints When Needed

```aria
// Good: Clear what T must support
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then return a; else return b; end
}
```

### ✅ DO: Use Descriptive Type Names

```aria
// Good: Clear purpose
func:cache_get = Value?(Key:key) { ... }

// Avoid: Unclear
func:cache_get = B?(A:key) { ... }
```

### ❌ DON'T: Overuse Generics

```aria
// Wrong: Only works with numbers anyway
func:add_numbers = T(T:a, T:b) {
    pass(a + b);
}

// Right: Be specific
func:add_numbers = int32(int32:a, int32:b) {
    pass(a + b);
}
```

### ❌ DON'T: Forget Type Constraints

```aria
// Wrong: Won't compile - T might not support +
func:sum = T([]T:array) {
    total: T = 0;  // Error: can't assign 0 to T
    till(array.length - 1, 1) {
        item: T = array[$];
        total = total + item;  // Error: T might not have +
    }
    pass(total);
}

// Right: Add constraint
func:sum = T([]T:array)where T: Numeric {
    total: T = T::zero();
    till(array.length - 1, 1) {
        item: T = array[$];
        total = total + item;
    }
    pass(total);
}
```

---

## Related Topics

- [Generic Functions](generic_functions.md) - Function generics in detail
- [Generic Structs](generic_structs.md) - Struct generics
- [Generic Syntax](generic_syntax.md) - Syntax reference
- [Monomorphization](monomorphization.md) - How generics compile
- [Type Inference](type_inference.md) - Type parameter inference

---

**Remember**: Generics are **compile-time templates** - they generate specialized code for each type you use. Zero runtime overhead!
