# Generic Parameters

**Category**: Functions → Generics → Reference  
**Concept**: Type parameters in generic definitions  
**Syntax**: `<T>`, `<T, U>`, `<T: Trait>`

---

## What Are Generic Parameters?

**Generic parameters** are the type variables (`T`, `U`, `K`, `V`, etc.) used in generic definitions. They're placeholders for concrete types.

---

## Basic Syntax

```aria
// Single parameter
func:identity = T(T:value) {
    pass(value);
}

// Multiple parameters
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}
```

---

## Naming Conventions

### Standard Names

```aria
// T - Generic Type
func:wrap = Box<T>(T:value) { }

// T, U - Two types
func:convert = U(T:input) { }

// K, V - Key, Value
func:insert = NIL(K:key, V:value) { }

// E - Error type
func:try_parse = Result<i32,(string:input)E> { }

// R - Result type
func:execute = R(string:query) { }
```

### Descriptive Names

```aria
// Longer names when helpful
func:transform = Output(Input:data) { }

func:cache = NIL(Key:k, Value:v) { }

func:convert = Target(Source:s) { }
```

---

## Parameter Declaration Locations

### On Functions

```aria
func:function = T(T:param) {
    // T available in this function
}
```

### On Structs

```aria
struct Container<T> {
    items: []T
}
// T available for all fields
```

### On Enums

```aria
enum Result<T, E> {
    Ok(T),
    Err(E)
}
// T and E available for all variants
```

### On Impl Blocks

```aria
impl<T> Container<T> {
    // T available for all methods
    func:new = Container<T>() { }
    func:add = NIL(T:item) { }
}
```

### On Traits

```aria
trait Convert<T> {
    func:convert = Self;(T:value)
}
// T available for all trait methods
```

---

## Type Parameter Scope

```aria
struct Box<T> {
    value: T
    // T in scope for struct
}

impl<T> Box<T> {
    // T from struct is in scope
    
    func:get = T() {
        pass(self.value);
    }
    
    // U is NEW parameter, separate from T
    func:map = U)(fn(T:f)-> Box<U> {
        pass(Box{value: f(self.value)});
    }
}
```

---

## Parameter Constraints

### Single Constraint

```aria
// T must implement Display
func:print = NIL(T:value)where T: Display {
    print(value + "\n");
}

// Inline syntax
func:print = NIL(T:value) {
    print(value + "\n");
}
```

### Multiple Constraints

```aria
// T must implement multiple traits
func:process = NIL(T:value)
    where T: Display, 
          T: Clone,
          T: Debug {
    // Implementation
}

// Inline with +
func:process = NIL(T:value) {
    // Implementation
}
```

### Different Parameters, Different Constraints

```aria
func:transform = U(T:input)
    where T: Serialize,
          U: Deserialize {
    data: []u8 = input.serialize();
    pass(U::deserialize(data));
}
```

---

## Default Type Parameters (If Supported)

```aria
// Default type for T
struct Container<T = i32> {
    items: []T
}

// Can omit type parameter
default: Container = Container{items: []};  // Container<i32>

// Or specify explicitly
custom: Container<string> = Container{items: []};
```

---

## Const Generic Parameters (If Supported)

```aria
// Generic over constant value
struct Array<T, const N: usize> {
    data: [T; N]
}

// N is compile-time constant
arr: Array<i32, 10>;  // Array of exactly 10 i32s
```

---

## Lifetime Parameters (If Supported)

```aria
// Lifetime parameter
func:longest = 'a->('a string->:s1, 'a string->:s2)string {
    when s1.len() > s2.len() then
        pass(s1);
    else
        pass(s2);
    end
}

// Multiple lifetimes
struct Pair<'a, 'b, T> {
    first: &'a T,
    second: &'b T
}
```

---

## Parameter Position

### In Function Signature

```aria
fn process<T>(
    input: T,        // T used in parameter
    count: i32       // Regular type
) -> Vec<T> {        // T used in return
    // Implementation
}
```

### In Struct Fields

```aria
struct Container<T> {
    data: T,         // T used in field
    size: usize      // Regular type
}
```

### In Type Aliases

```aria
type Result<T> = Result<T, Error>;
//         ↑ parameter    ↑ used here
```

---

## Unused Parameters (Phantom Types)

```aria
// T not used in fields but marks the type
struct Marker<T> {
    _phantom: PhantomData<T>
}

// Different markers are different types
type UserId = Marker<User>;
type ProductId = Marker<Product>;
```

---

## Parameter Inference

### From Arguments

```aria
func:identity = T(T:value) {
    pass(value);
}

// T inferred as i32
Result: i32 = identity(42);
```

### From Return Type

```aria
func:default = T() {
    pass(T::default());
}

// T inferred as string
text: string = default();
```

### Partial Inference

```aria
func:convert = To(From:value) {
    // Implementation
}

// From inferred, To specified
Result: string = convert::<_, string>(42);
```

---

## Multiple Parameter Relationships

### Independent Parameters

```aria
// T and U are unrelated
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}
```

### Related Parameters

```aria
// U depends on T through constraint
func:map = U)([]T:array, fn(T:f)-> []U {
    Result: []U = [];
    till(array.length - 1, 1) {
        item: T = array[$];
        result.push(f(item));
    }
    pass(result);
}
```

### Same Parameter Used Multiple Times

```aria
// T used in multiple positions
func:swap = (T,(T:a, T:b)T) {
    pass((b, a));
}
```

---

## Best Practices

### ✅ DO: Use Single Letters for Simple Generics

```aria
// Good: Standard and clear
func:identity = T(T:value) { }
func:pair = (T,(T:a, U:b)U) { }
```

### ✅ DO: Use Descriptive Names for Complex Generics

```aria
// Good: Clear purpose
fn transform<InputData, OutputFormat>(
    data: InputData
) -> OutputFormat { }
```

### ✅ DO: Add Constraints When Needed

```aria
// Good: Clear requirements
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then return a; else return b; end
}
```

### ✅ DO: Keep Parameter Count Reasonable

```aria
// Good: 2-3 parameters
func:convert = To(From:value) { }

// OK: 4 parameters if needed
func:process = NIL(...) { }
```

### ❌ DON'T: Use Too Many Parameters

```aria
// Wrong: Too many
func:complex = NIL(...) { }

// Right: Simplify or use structs
struct Config<T, U> { }
func:complex = NIL(Config<V, W>:config) { }
```

### ❌ DON'T: Use Generic When Specific Works

```aria
// Wrong: Always i32 anyway
func:add = T(T:a, T:b) {
    pass(a + b);
}

// Right: Be specific
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## Common Parameter Patterns

### Container Element Type

```aria
struct Vec<T> {
    items: []T
}
```

### Key-Value Pairs

```aria
struct Map<K, V> {
    entries: []Entry<K, V>
}
```

### Input-Output Transform

```aria
func:map = U)([]T:array, fn(T:f)-> []U { }
```

### Error Type Parameter

```aria
enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

---

## Related Topics

- [Generics](generics.md) - Generic overview
- [Generic Functions](generic_functions.md) - Generic functions
- [Generic Syntax](generic_syntax.md) - Complete syntax
- [Type Inference](type_inference.md) - Parameter inference

---

**Remember**: Generic parameters are **type placeholders** - they get replaced with concrete types when you use the generic type or function!
