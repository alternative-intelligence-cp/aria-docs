# Function Declaration

**Category**: Functions → Basics  
**Keyword**: `fn`  
**Philosophy**: Functions are first-class, type-safe, and explicit

---

## Basic Syntax

```aria
func:function_name = ReturnType(Type1:param1, Type2:param2) {
    // Function body
    pass(value);
}
```

---

## Components

### Function Keyword: `fn`

```aria
func:greet = string(string:name) {
    pass("Hello, " + name);
}
```

### Parameters with Types

```aria
// Parameters must have explicit types
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

### Return Type

```aria
// Return type declared with ->
func:calculate = flt64() {
    pass(42.5);
}

// No return value (void)
func:print_message = NIL(string:msg) {
    print(msg + "\n");
    // No return statement needed
}
```

---

## Examples

### Simple Function

```aria
func:square = int32(int32:x) {
    pass(x * x);
}

// Usage
Result: i32 = square(5);  // 25
```

### Multiple Parameters

```aria
func:calculate_area = flt64(flt64:width, flt64:height) {
    pass(width * height);
}

area: f64 = calculate_area(10.0, 20.0);  // 200.0
```

### No Return Value

```aria
func:log_message = NIL(string:level, string:msg) {
    print("[" + level + "] " + msg + "\n");
}

log_message("INFO", "Application started");
```

### With TBB Return Type

```aria
func:divide = tbb32(tbb32:a, tbb32:b) {
    when b == 0 then
        pass(ERR);  // TBB error sentinel
    end
    
    pass(a / b);
}

Result: tbb32 = divide(10, 2);  // 5
error: tbb32 = divide(10, 0);   // ERR
```

---

## Early Return

```aria
func:check_valid = bool(int32:value) {
    when value < 0 then
        pass(false);  // Early return
    end
    
    when value > 100 then
        pass(false);
    end
    
    pass(true);
}
```

---

## Function Naming Conventions

### Standard Style
```aria
// snake_case for functions
func:calculate_total = int32() { ... }
func:process_user_input = string() { ... }
```

### Predicates (Boolean Returns)
```aria
// Common: is_, has_, can_ prefixes
func:is_valid = bool(int32:x) { ... }
func:has_permission = bool(User:user) { ... }
func:can_access = bool(Resource:resource) { ... }
```

---

## Parameter Passing

### By Value (Default)
```aria
func:modify = int32(int32:x) {
    x = x + 1;  // Modifies local copy
    pass(x);
}

value: i32 = 10;
Result: i32 = modify(value);
// value is still 10, result is 11
```

### By Reference (with $)
```aria
func:modify_ref = NIL($i32:x) {
    x = x + 1;  // Modifies original
}

value: i32 = 10;
modify_ref($value);
// value is now 11
```

---

## Visibility

### Public Functions
```aria
// Accessible from other modules
pub func:public_api = string() {
    pass("Available to all");
}
```

### Private Functions (Default)
```aria
// Only accessible within same module
func:internal_helper = int32() {
    pass(42);
}
```

---

## Best Practices

### ✅ DO: Be Explicit with Types

```aria
// Good: Clear types
func:process = int32(string:data) {
    pass(data.len());
}
```

### ✅ DO: Use TBB for Error-Prone Operations

```aria
// Good: TBB handles errors
func:parse_number = tbb32(string:s) {
    // Returns ERR on parse failure
    pass(parse(s));
}
```

### ✅ DO: Keep Functions Focused

```aria
// Good: Single responsibility
func:validate_email = bool(string:email) {
    pass(email.contains("@"));
}

func:send_email = NIL(string:to, string:message) {
    // Separate concern
}
```

### ❌ DON'T: Omit Return Types

```aria
// Wrong: Unclear what this returns
func:calculate = NIL(x, y) {  // Missing types!
    pass(x + y);
}

// Right:
func:calculate = int32(int32:x, int32:y) {
    pass(x + y);
}
```

---

## Related Topics

- [Function Syntax](function_syntax.md) - Complete syntax reference
- [Function Parameters](function_params.md) - Parameter details
- [Function Return Types](function_return_type.md) - Return type options
- [pass/fail Keywords](pass_keyword.md) - TBB result handling
- [Generic Functions](generic_functions.md) - Parameterized functions

---

**Remember**: Aria functions are explicit, type-safe, and designed for clarity. No implicit conversions, no hidden behavior.
