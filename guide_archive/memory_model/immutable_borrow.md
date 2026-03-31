# Immutable Borrow

**Category**: Memory Model → References  
**Operator**: `&`  
**Concept**: Read-only reference

---

## What is an Immutable Borrow?

An **immutable borrow** is a read-only reference created with `&` that allows viewing data without modifying it.

---

## Basic Syntax

```aria
value: i32 = 42;
ref: $i32 = $value;  // Immutable borrow

print(ref);  // ✅ Can read
ref = 100;      // ❌ Can't modify
```

---

## Multiple Immutable Borrows

You can have **unlimited** immutable borrows simultaneously:

```aria
value: i32 = 42;

ref1: $i32 = $value;  // ✅
ref2: $i32 = $value;  // ✅
ref3: $i32 = $value;  // ✅

print(ref1 + ref2 + ref3);  // All valid
```

---

## In Functions

```aria
func:print_value = NIL(int32->:v) {
    print(v + "\n");
}

value: i32 = 42;
print_value($value);  // Borrow
print(value);      // Still usable
```

---

## Multiple Parameters

```aria
func:compare = bool(int32->:a, int32->:b) {
    pass(a < b);
}

x: i32 = 10;
y: i32 = 20;

Result: bool = compare($x, $y);  // Both borrowed
print(x + y);  // Both still usable
```

---

## With Collections

```aria
func:find = i32?([]string->:items, string:target) {
    till(items.length - 1, 1) {
        if items[$] == target {
            pass($);
        }
    }
    pass(nil);
}

names: []string = ["Alice", "Bob", "Charlie"];
index: i32? = find($names, "Bob");
// names still usable
```

---

## Rules

### ✅ CAN: Multiple Readers

```aria
value: i32 = 42;

ref1: $i32 = $value;
ref2: $i32 = $value;
ref3: $i32 = $value;
// All OK simultaneously
```

### ❌ CAN'T: Modify Through Reference

```aria
value: i32 = 42;
ref: $i32 = $value;

ref = 100;  // ❌ Error: immutable reference
```

### ❌ CAN'T: Mix with Mutable Borrow

```aria
value: i32 = 42;

ref: $i32 = $value;      // Immutable borrow
mut_ref: $i32 = $value;  // ❌ Error: can't borrow mutably
```

---

## Automatic Dereferencing

Aria automatically dereferences in most contexts:

```aria
value: i32 = 42;
ref: $i32 = $value;

// Automatic dereference
print(ref);          // Works like ref is i32
Result: i32 = ref + 1;  // Works like ref is i32
```

---

## Common Patterns

### Read Configuration

```aria
func:get_host = string(Config->:config) {
    pass(config.host);  // Just reading
}
```

### Calculate from Data

```aria
func:calculate_average = flt64([]f64->:numbers) {
    sum: f64 = 0.0;
    till(numbers.length - 1, 1) {
        sum = sum + numbers[$];
    }
    pass(sum / numbers.length() as f64);
}
```

### Validation

```aria
func:is_valid = bool(User->:user) {
    pass(user.name.length() > 0 && user.age >= 18);
}
```

---

## Best Practices

### ✅ DO: Use for Read-Only Access

```aria
// Good: Just reading
func:display = NIL(Data->:data) {
    print(data.format());
}
```

### ✅ DO: Default to Immutable

```aria
// Good: Immutable unless mutation needed
func:process = NIL(string->:input) {  // Read-only
    analyze(input);
}
```

### ❌ DON'T: Use When Mutation Needed

```aria
// Wrong: Need mutable for this
func:double = NIL(int32->:value) {
    value = value * 2;  // ❌ Error: immutable
}

// Right: Use mutable borrow
func:double = NIL(int32->:value) {
    value = value * 2;  // ✅ OK
}
```

---

## Examples

### String Operations

```aria
func:count_words = int32(string->:text) {
    pass(text.split(" ").length());
}

doc: string = "Hello world from Aria";
count: i32 = count_words($doc);
```

### Comparison

```aria
func:max = int32(int32->:a, int32->:b) {
    if a > b {
        pass(a);
    } else {
        pass(b);
    }
}

x: i32 = 10;
y: i32 = 20;
Result: i32 = max($x, $y);
```

---

## Related Topics

- [Borrowing](borrowing.md) - Borrowing overview
- [Mutable Borrow](mutable_borrow.md) - Mutable references
- [Borrow Operator](borrow_operator.md) - & operator

---

**Remember**: Immutable borrow = **read-only reference** - unlimited readers, no writers!
