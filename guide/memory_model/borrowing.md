# Borrowing

**Category**: Memory Model → References  
**Concept**: Temporary access without ownership  
**Philosophy**: Safe sharing through rules

---

## What is Borrowing?

**Borrowing** is taking a **temporary reference** to data without taking ownership, allowing safe sharing of data.

---

## Basic Concept

```aria
func:main = NIL() {
    value: i32 = 42;
    
    print_value($value);  // Borrow value
    
    print(value);  // Still owns value
}

func:print_value = NIL(int32->:v) {
    print(v + "\n");  // Temporary access
}  // Borrow ends here
```

---

## Ownership vs Borrowing

### Ownership (Transfer)

```aria
func:take_ownership = NIL(string:s) {
    print(s);
}  // s dropped here

value: string = "hello";
take_ownership(value);
print(value);  // ❌ Error: value moved
```

### Borrowing (Reference)

```aria
func:borrow = NIL(string->:s) {
    print(s);
}  // Borrow ends, s still valid

value: string = "hello";
borrow($value);
print(value);  // ✅ OK: still owns value
```

---

## Immutable Borrowing

Default borrows are **read-only**:

```aria
value: i32 = 42;
ref: $i32 = $value;

print(ref);   // ✅ Can read
ref = 100;       // ❌ Can't modify
```

---

## Mutable Borrowing

Use `$` for mutable borrows:

```aria
value: i32 = 42;
ref: $i32 = $value;  // Mutable borrow

ref = 100;  // ✅ Can modify through reference

print(value);  // 100
```

---

## Borrowing Rules

### Rule 1: Multiple Immutable Borrows OK

```aria
value: i32 = 42;

ref1: $i32 = $value;  // ✅ OK
ref2: $i32 = $value;  // ✅ OK
ref3: $i32 = $value;  // ✅ OK

print(ref1 + ref2 + ref3);  // All can read
```

### Rule 2: Only One Mutable Borrow

```aria
value: i32 = 42;

mut_ref1: $i32 = $value;  // ✅ OK
mut_ref2: $i32 = $value;  // ❌ Error: already borrowed mutably
```

### Rule 3: Can't Mix Immutable and Mutable

```aria
value: i32 = 42;

ref: $i32 = $value;      // Immutable borrow
mut_ref: $i32 = $value;  // ❌ Error: can't borrow mutably while immutably borrowed
```

---

## Why These Rules?

### Prevent Data Races

```aria
// Without rules (unsafe):
value: i32 = 42;
ref1: $i32 = $value;
ref2: $i32 = $value;  // Danger!

// Thread 1 reads ref1
// Thread 2 modifies through ref2
// Data race!

// With rules (safe):
// Can't have mutable and immutable borrows simultaneously
```

### Prevent Iterator Invalidation

```aria
// Without rules (unsafe):
vec: Vec<i32> = vec![1, 2, 3];
till(vec.length - 1, 1) {     // Iterate over vec
    vec.push(4);              // ❌ Modify while iterating
    // Iterator now invalid!
}

// With rules (safe):
// Can't modify while borrowed, iterator stays valid
```

---

## Borrow Scope

Borrows last until their **last use**:

```aria
value: i32 = 42;

{
    ref: $i32 = $value;  // Borrow starts
    print(ref);       // Last use of ref
}  // Borrow ends

// value can be borrowed again
mut_ref: $i32 = $value;  // ✅ OK
```

---

## Function Parameters

### By Value (Takes Ownership)

```aria
func:consume = NIL(string:s) {
    print(s);
}  // s dropped

text: string = "hello";
consume(text);
// text no longer valid
```

### By Reference (Borrows)

```aria
func:display = NIL(string->:s) {
    print(s);
}  // Borrow ends

text: string = "hello";
display($text);  // Borrow
// text still valid
```

### Mutable Reference

```aria
func:modify = NIL(string->:s) {
    s.push_str(" world");
}

text: string = "hello";
modify($text);  // Mutable borrow
print(text);  // "hello world"
```

---

## Returning References

Can return references if they **outlive the function**:

```aria
// ✅ OK: Reference to parameter
func:first = int32->([]i32->:arr) {
    pass($arr[0]);
}

// ❌ Error: Reference to local
func:create = int32->() {
    x: i32 = 42;
    pass($x);  // x freed at end of function!
}
```

---

## Common Patterns

### Read-Only Access

```aria
func:get_length = int32(string->:s) {
    pass(s.length());  // Read only
}

text: string = "hello";
len: i32 = get_length($text);
```

### Modify in Place

```aria
func:double = NIL(int32->:value) {
    value = value * 2;
}

x: i32 = 21;
double($x);
print(x);  // 42
```

### Multiple Readers

```aria
func:compare = bool(int32->:a, int32->:b) {
    pass(a < b);
}

x: i32 = 10;
y: i32 = 20;

// Both borrowed simultaneously
Result: bool = compare($x, $y);
```

---

## Borrow Checker

Aria's **borrow checker** enforces these rules at compile time:

```aria
// ❌ Compile error caught early
value: i32 = 42;
ref1: $i32 = $value;
ref2: $i32 = $value;  // Error: can't borrow mutably while immutably borrowed

// vs C++ (runtime error)
// Compiles but crashes at runtime!
```

---

## Best Practices

### ✅ DO: Prefer Borrowing

```aria
// Good: Borrow when you don't need ownership
func:process = NIL(Data->:data) {
    analyze(data);
}
```

### ✅ DO: Use Immutable by Default

```aria
// Good: Immutable unless mutation needed
func:read = NIL(string->:s) { }     // Read-only
func:modify = NIL(string->:s) { }   // Needs mutation
```

### ✅ DO: Keep Borrows Short

```aria
// Good: Short borrow scope
{
    ref: $i32 = $value;
    print(ref);
}  // Borrow ends

// Can now modify value
value = 100;
```

### ❌ DON'T: Hold Borrows Too Long

```aria
// Wrong: Borrow held too long
ref: $i32 = $value;

// Lots of code...

value = 100;  // ❌ Error: can't modify while borrowed
print(ref);  // Still using borrow
```

### ❌ DON'T: Return References to Locals

```aria
// Wrong: Dangling reference
func:bad = int32->() {
    x: i32 = 42;
    pass($x);  // ❌ x freed at end
}

// Right: Return value
func:good = int32() {
    x: i32 = 42;
    pass(x);  // Copied
}
```

---

## Real-World Examples

### Configuration Reader

```aria
struct Config {
    host: string,
    port: i32
}

func:get_connection_string = string(Config->:config) {
    // Borrows config, doesn't take ownership
    pass(format("{}:{}", config.host, config.port));
}

config: Config = load_config();
conn_str: string = get_connection_string($config);
// config still available
```

### Collection Processing

```aria
func:sum = int32([]i32->:numbers) {
    total: i32 = 0;
    till(numbers.length - 1, 1) {
        total = total + numbers[$];
    }
    pass(total);
}

values: []i32 = [1, 2, 3, 4, 5];
Result: i32 = sum($values);
// values still usable
```

### In-Place Update

```aria
func:apply_discount = NIL(flt64->:price, flt64:discount) {
    price = price * (1.0 - discount);
}

item_price: f64 = 100.0;
apply_discount($item_price, 0.2);
print(item_price);  // 80.0
```

---

## Borrowing in Loops

```aria
items: []Item = load_items();

// Immutable borrow
till(items.length - 1, 1) {
    print(items[$].name);  // Read only
}

// Mutable borrow - using $ for mutable index
till(items.length - 1, 1) {
    items[$].update();  // Can modify
}
```

---

## Related Topics

- [Immutable Borrow](immutable_borrow.md) - Read-only references
- [Mutable Borrow](mutable_borrow.md) - Mutable references
- [Borrow Operator](borrow_operator.md) - & and $ operators
- [Ownership](ownership.md) - Ownership system

---

**Remember**: Borrowing = **temporary access without ownership** - multiple readers OR one writer, enforced at compile time!
