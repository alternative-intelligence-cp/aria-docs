# The `void` Type

**Category**: Types → Special  
**Syntax**: `void`  
**Purpose**: Absence of a return value

---

## Overview

`void` represents the **absence of a value**. Used for functions that don't return anything.

---

## Function Returns

```aria
// Function that returns nothing
func:print_message = NIL(string:msg) {
    print(msg);
    // No return statement needed
}

// Explicit return
func:log_error = NIL(string:err) {
    stderr_write(err);
    pass(NIL);  // Optional
}
```

---

## Can be Omitted

```aria
// void can be omitted
func:greet = NIL(string:name) {
    print(`Hello, &{name}!`);
}

// Equivalent to
func:greet = NIL(string:name) {
    print(`Hello, &{name}!`);
}
```

---

## Use Cases

### Side Effects Only

```aria
// Modify state, no return
func:increment_counter = NIL() {
    counter += 1;
}
```

### I/O Operations

```aria
func:write_file = NIL(string:path, string:data) {
    file: File = File.open(path, "w");
    file.write(data);
    file.close();
}
```

---

## Best Practices

### ✅ DO: Omit for Clarity

```aria
// Clear without -> void
func:process = NIL() {
    ...
}
```

### ✅ DO: Use for Side Effects

```aria
// Function changes state
func:update_display = NIL() {
    screen.refresh();
}
```

### ❌ DON'T: Try to Use the Value

```aria
func:do_something = NIL() {
    ...
}

result := do_something();  // ❌ Error - void has no value
```

---

## Difference from nil

| Type | Purpose | Example |
|------|---------|---------|
| `void` | No return value | `fn print() -> void` |
| `nil` | Null pointer/optional | `ptr: *i32 = nil;` |

---

## Related

- [nil](nil_null_void.md) - Null value
- [Result](result.md) - For error handling
- [Functions](../functions/README.md)

---

**Remember**: `void` means **no return value**, `nil` means **null**!
