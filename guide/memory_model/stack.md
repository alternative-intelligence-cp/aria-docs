# Stack Memory

**Category**: Memory Model → Memory Regions  
**Concept**: Automatic, fast allocation  
**Philosophy**: Predictable, efficient, safe

---

## What is Stack Memory?

The **stack** is a region of memory for automatic, short-lived allocations that are freed in reverse order of creation.

---

## Characteristics

### Automatic Management

```aria
func:calculate = int32() {
    x: i32 = 42;        // Allocated on stack
    y: i32 = x * 2;     // Allocated on stack
    pass(y);
}  // x and y automatically freed
```

### LIFO (Last In, First Out)

```aria
func:example = NIL() {
    a: i32 = 1;     // Pushed to stack
    b: i32 = 2;     // Pushed to stack
    c: i32 = 3;     // Pushed to stack
}  // Freed in reverse: c, b, a
```

### Fast Allocation

Stack allocation is **extremely fast** - just moving a pointer:

```aria
// Stack: Single pointer bump
value: i32 = 42;  // Microseconds

// vs Heap: Search free list, bookkeeping
value: Box<i32> = Box::new(42);  // Slower
```

---

## What Goes on the Stack?

### Local Variables

```aria
func:process = NIL() {
    count: i32 = 0;          // Stack
    name: string = "test";   // Stack (string header)
    active: bool = true;     // Stack
}
```

### Function Parameters

```aria
func:add = int32(int32:a, int32:b) {
    // a and b are on the stack
    pass(a + b);
}
```

### Return Values

```aria
func:create = Point() {
    p: Point = Point{x: 10, y: 20};
    pass(p);  // Copied to caller's stack
}
```

### Small Structs

```aria
struct Point {
    x: i32,
    y: i32
}

point: Point = Point{x: 10, y: 20};  // Stack
```

---

## Stack Frames

Each function call creates a **stack frame**:

```aria
func:main = NIL() {
    // main's stack frame
    x: i32 = 10;
    
    Result: i32 = calculate(x);
}

func:calculate = int32(int32:value) {
    // calculate's stack frame (on top of main's)
    doubled: i32 = value * 2;
    pass(doubled);
}  // calculate's frame popped
// Back to main's frame
```

---

## Stack Size Limits

Stack space is **limited** (typically 1-8 MB):

```aria
// ❌ Stack overflow!
func:recursive = NIL() {
    large: [i32; 1000000] = [0; 1000000];  // Too big!
    recursive();  // Infinite recursion
}

// ✅ Use heap for large data
func:better = NIL() {
    large: Vec<i32> = Vec::with_capacity(1000000);
}
```

---

## Advantages of Stack

### 1. Speed

```aria
// Fast: Stack allocation
till(999999, 1) {
    value: i32 = $;  // Instant
}
```

### 2. Automatic Cleanup

```aria
func:example = NIL() {
    file: File = pass open("data.txt");
    // file automatically closed at end of scope
}  // Automatic!
```

### 3. Cache-Friendly

Stack data is **contiguous** and **local**, great for CPU cache:

```aria
// All on stack, cache-friendly
a: i32 = 1;
b: i32 = 2;
c: i32 = 3;
Result: i32 = a + b + c;
```

### 4. No Fragmentation

Stack allocations don't fragment memory:

```aria
// Stack pointer just moves up/down
// No fragmentation issues
```

---

## Stack vs Heap

### Stack

```aria
// Automatic, fast, limited size
value: i32 = 42;
array: [i32; 10] = [0; 10];
```

**Pros:**
- ✅ Extremely fast
- ✅ Automatic cleanup
- ✅ No fragmentation
- ✅ Cache-friendly

**Cons:**
- ❌ Limited size
- ❌ Fixed lifetime (scope-bound)
- ❌ Can't resize

### Heap

```aria
// Manual, slower, unlimited size
value: Box<i32> = Box::new(42);
vector: Vec<i32> = Vec::new();
```

**Pros:**
- ✅ Unlimited size (system memory)
- ✅ Flexible lifetime
- ✅ Can resize (Vec, etc.)

**Cons:**
- ❌ Slower allocation
- ❌ Manual management needed
- ❌ Can fragment
- ❌ Less cache-friendly

---

## Scope and Lifetime

Stack variables live until the end of their scope:

```aria
func:example = NIL() {
    {
        x: i32 = 42;
        print(x);  // OK
    }  // x freed here
    
    print(x);  // ❌ Error: x doesn't exist
}
```

---

## Stack Overflow

### Causes

1. **Deep recursion**

```aria
// ❌ Stack overflow
func:factorial = int32(int32:n) {
    if n == 0 { return 1; }
    pass(n * factorial(n - 1));  // 1000000 recursive calls!
}
```

2. **Large local arrays**

```aria
// ❌ Stack overflow
func:process = NIL() {
    huge: [i32; 10000000] = [0; 10000000];  // Too big!
}
```

### Solutions

```aria
// ✅ Use heap for large data
func:process = NIL() {
    huge: Vec<i32> = vec![0; 10000000];
}

// ✅ Use iteration instead of recursion
func:factorial = int32(int32:n) {
    result: i32 = 1;
    till(n, 1) {
        result = result * $;
    }
    pass(result);
}
```

---

## Best Practices

### ✅ DO: Use Stack by Default

```aria
// Good: Small, temporary data
count: i32 = 0;
point: Point = Point{x: 10, y: 20};
```

### ✅ DO: Keep Stack Allocations Small

```aria
// Good: Small arrays
buffer: [u8; 256] = [0; 256];

// Wrong: Large arrays
huge: [u8; 10000000] = [0; 10000000];  // Use heap!
```

### ✅ DO: Limit Recursion Depth

```aria
// Good: Bounded recursion
func:search = bool(Node:node, int32:depth) {
    if depth > 100 {
        pass(false);  // Prevent overflow
    }
    // ...
}
```

### ❌ DON'T: Return Pointers to Stack

```aria
// ❌ Wrong: Dangling pointer!
func:get_pointer = int32->() {
    x: i32 = 42;
    pass($x);  // Error: x freed at end of function!
}

// ✅ Right: Return value
func:get_value = int32() {
    x: i32 = 42;
    pass(x);  // Copied to caller
}
```

---

## Real-World Examples

### Simple Function

```aria
func:calculate_area = flt64(flt64:width, flt64:height) {
    // All on stack
    area: f64 = width * height;
    pass(area);
}  // width, height, area all freed
```

### Processing Loop

```aria
func:process_items = NIL([]Item:items) {
    till(items.length - 1, 1) {
        // Loop variables on stack
        valid: bool = items[$].validate();
        
        when valid then
            result: Result = items[$].process();
            log_result(result);
        end
    }  // Loop variables freed each iteration
}
```

### Nested Scopes

```aria
func:example = NIL() {
    outer: i32 = 1;
    
    {
        inner: i32 = 2;
        print(outer + inner);  // 3
    }  // inner freed
    
    print(outer);  // OK
    print(inner);  // ❌ Error
}
```

---

## Stack Unwinding

When errors occur, stack frames are **unwound**:

```aria
func:main = NIL() {
    when process() == ERR then
        stderr_write("Error occurred\n");
    end
}

func:process = Result() {
    file: File = pass open("data.txt");
    defer file.close();  // Runs on unwind
    
    data: Data = pass parse(file);  // Error here
    pass(Ok(data));
}
// file.close() called during unwind
```

---

## Related Topics

- [Allocation](allocation.md) - Memory allocation
- [Defer](defer.md) - Cleanup on scope exit
- [RAII](raii.md) - Resource management
- [Borrowing](borrowing.md) - References to stack data

---

**Remember**: Stack is **automatic, fast, limited** - use for small, temporary data, heap for large or long-lived allocations!
