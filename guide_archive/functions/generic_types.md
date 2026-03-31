# Generic Types Overview

**Category**: Functions → Generics  
**Concept**: Types that work with any type parameter  
**Philosophy**: Write once, use with unlimited types

---

## What Are Generic Types?

**Generic types** are type definitions (structs, enums, type aliases) that work with **type parameters** instead of concrete types.

---

## Generic Struct Types

```aria
struct Box<T> {
    value: T
}

// Different instantiations create different types
int_box: Box<i32>;      // Type: Box<i32>
str_box: Box<string>;   // Type: Box<string>
bool_box: Box<bool>;    // Type: Box<bool>

// These are all DIFFERENT types
// Can't assign Box<i32> to Box<string>
```

---

## Generic Enum Types

```aria
enum Option<T> {
    Some(T),
    None
}

// Different types
maybe_int: Option<i32> = Some(42);
maybe_str: Option<string> = Some("hello");
```

---

## Generic Type Aliases

```aria
// Simplify complex types
type Result<T> = Result<T, Error>;

// Now use simpler name
func:parse = Result<int32>(string:input) {
    // Returns Result<i32, Error>
}

// Multiple parameters
type HashMap<K, V> = Map<K, V>;
type Callback<T> = fn(T) -> bool;
```

---

## Type Parameter Constraints

```aria
// T must implement Display
struct Printable<T> where T: Display {
    value: T
}

// Multiple constraints
struct Comparable<T> where T: Ord, T: Hash {
    items: []T
}
```

---

## Nested Generic Types

```aria
// Generic types can be nested
vec_of_options: Vec<Option<i32>>;
map_of_vecs: Map<string, Vec<User>>;
box_of_results: Box<Result<Data, Error>>;
```

---

## Type Equivalence

```aria
// These are the SAME type
type A = Box<i32>;
type B = Box<i32>;

a: A = Box{value: 42};
b: B = a;  // OK - same type

// These are DIFFERENT types
type C = Box<i32>;
type D = Box<i64>;

c: C = Box{value: 42};
// d: D = c;  // Error - different types
```

---

## Generic Type Parameters in Functions

```aria
// Function accepts any Box<T>
func:unbox = T(Box<T>:box) {
    pass(box.value);
}

// T inferred from argument type
int_box: Box<i32> = Box{value: 42};
value: i32 = unbox(int_box);  // T = i32
```

---

## Monomorphization

Each generic type instantiation creates a separate concrete type:

```aria
struct Vec<T> {
    data: []T
}

// Compiler generates:
// struct Vec_i32 { data: []i32 }
// struct Vec_string { data: []string }
// struct Vec_User { data: []User }

numbers: Vec<i32>;     // Uses Vec_i32
names: Vec<string>;    // Uses Vec_string
users: Vec<User>;      // Uses Vec_User
```

---

## Type Parameter Scope

```aria
// T in scope for entire impl block
impl<T> Vec<T> {
    func:new = Vec<T>() {
        // T available here
    }
    
    func:push = NIL(T:item) {
        // T available here
    }
    
    // U is separate type parameter
    func:map = U)(fn(T:f)-> Vec<U> {
        // Both T and U available here
    }
}
```

---

## Associated Types vs Generic Types

### Generic Type

```aria
// Type parameter on struct
struct Container<T> {
    items: []T
}

// Must specify T when using
func:process = NIL(Container<i32>:c) { }
```

### Associated Type

```aria
// Type decided by implementation
trait Iterator {
    type Item;  // Associated type
    
    func:next = Result<Self::Item>()
}

// Type determined by trait impl
struct NumberIterator;

impl Iterator for NumberIterator {
    type Item = i32;  // Concrete type
    
    func:next = Result<int32>() { }
}
```

---

## Const Generic Types (If Supported)

```aria
// Generic over constant value
struct Array<T, const N: usize> {
    data: [T; N]
}

// Different sizes are different types
small: Array<i32, 10>;
large: Array<i32, 1000>;
```

---

## Higher-Kinded Types (If Supported)

```aria
// Generic over type constructors
trait Functor<F<_>> {
    func:map = B)(F<A>:fa, fn(A:f)-> F<B>;
}

// F could be Box, Option, Vec, etc.
```

---

## Phantom Types

```aria
// Type parameter not used in fields
struct Marker<T> {
    _phantom: PhantomData<T>  // Marks type without storing it
}

// Different markers are different types
type UserId = Marker<User>;
type ProductId = Marker<Product>;

// Can't mix them up
user_id: UserId;
product_id: ProductId;
// user_id = product_id;  // Error!
```

---

## Common Generic Type Patterns

### Container Types

```aria
struct Vec<T>;
struct Set<T>;
struct Map<K, V>;
struct Queue<T>;
struct Stack<T>;
```

### Wrapper Types

```aria
struct Box<T>;
struct Rc<T>;    // Reference counted
struct Arc<T>;   // Atomic reference counted
struct Mutex<T>; // Thread-safe wrapper
```

### Error Handling Types

```aria
enum Option<T>;
enum Result<T, E>;
```

### Smart Pointer Types

```aria
struct Unique<T>;  // Owned pointer
struct Shared<T>;  // Shared ownership
struct Weak<T>;    // Weak reference
```

---

## Type Bounds in Practice

### Single Bound

```aria
func:print = NIL(T:value)where T: Display {
    print(value + "\n");
}
```

### Multiple Bounds

```aria
func:process = NIL(T:value)
    where T: Clone, T: Debug, T: Send {
    // Can clone, debug, send to threads
}
```

### Bounds on Associated Types

```aria
func:iterate = NIL(I:iter)
    where I: Iterator,
          I::Item: Display {
    // Iterator's items must be displayable
}
```

---

## Generic Type Best Practices

### ✅ DO: Use Generics for Reusable Types

```aria
// Good: Reusable for any type
struct Stack<T> {
    items: []T
}
```

### ✅ DO: Add Meaningful Constraints

```aria
// Good: Clear what T must support
struct SortedList<T> where T: Ord {
    items: []T
}
```

### ✅ DO: Use Type Aliases for Clarity

```aria
// Good: Simpler name
type StringMap<V> = Map<string, V>;

// Instead of Map<string, User> everywhere
users: StringMap<User>;
```

### ❌ DON'T: Make Everything Generic

```aria
// Wrong: UserId should always be i32
struct UserId<T> {
    value: T
}

// Right: Concrete type
struct UserId {
    value: i32
}
```

### ❌ DON'T: Use Too Many Type Parameters

```aria
// Wrong: Confusing
struct Complex<T, U, V, W, X, Y, Z> {
    // Too many!
}

// Right: Group related types
struct Config {
    network: NetworkConfig,
    database: DatabaseConfig
}
```

---

## Type Inference

```aria
// Type inferred from constructor
numbers: Vec<i32> = Vec::new();  // T = i32

// Type inferred from usage
vec = Vec::new();  // Type unknown yet
vec.push(42);      // Now Vec<i32>

// Type inferred from return
func:get_numbers = Vec<i32>() {
    pass(Vec::new());  // T = i32 from return type
}
```

---

## Generic Type Compatibility

```aria
struct Box<T> {
    value: T
}

// Same type parameters = compatible
a: Box<i32> = Box{value: 42};
b: Box<i32> = a;  // OK

// Different type parameters = incompatible
c: Box<i64> = Box{value: 42};
// d: Box<i32> = c;  // Error: Box<i64> ≠ Box<i32>

// Even if inner types are convertible!
```

---

## Real-World Example

```aria
// Generic response wrapper
struct ApiResponse<T> {
    data: T,
    status: i32,
    timestamp: i64
}

impl<T> ApiResponse<T> where T: Deserialize {
    func:from_json = ApiResponse<T>?(string:json) {
        // Parse JSON and create response
    }
}

// Usage with different types
user_response: ApiResponse<User> = 
    ApiResponse::from_json(json_string);

product_response: ApiResponse<Product> = 
    ApiResponse::from_json(json_string);
```

---

## Related Topics

- [Generics](generics.md) - Generic overview
- [Generic Functions](generic_functions.md) - Generic functions
- [Generic Structs](generic_structs.md) - Generic structs in detail
- [Type Inference](type_inference.md) - Type parameter inference
- [Monomorphization](monomorphization.md) - How generics compile

---

**Remember**: Generic types are **type templates** - each instantiation with different type parameters creates a **distinct type**!
