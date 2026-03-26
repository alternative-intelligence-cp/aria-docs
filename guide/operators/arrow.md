# Arrow Operator (->)

**Category**: Operators → Type Declaration  
**Operator**: `->`  
**Purpose**: Specify return types

---

## Syntax

```aria
fn <name>(<params>) -> <return_type>
```

---

## Description

The arrow operator `->` declares a function's return type.

---

## Basic Usage

```aria
// Returns i32
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

// Returns string
func:greet = string(string:name) {
    pass("Hello, " + name);
}
```

---

## Void Functions

```aria
// No return type needed for void
func:print_message = NIL(string:msg) {
    print(msg);
}

// Or explicit void
func:log = NIL(string:text) {
    print(text);
}
```

---

## Complex Return Types

```aria
// Array
func:get_numbers = []i32() {
    pass([1, 2, 3, 4, 5]);
}

// Optional
func:find_user = User?(int32:id) {
    when exists(id) then
        pass(get_user(id));
    else
        pass(nil);
    end
}

// Tuple
func:divide = (i32,(int32:a, int32:b)i32) {
    pass((a / b, a % b));
}
```

---

## Generic Returns

```aria
func:create = T() {
    pass(T::new());
}

func:wrap = Result<T>(T:value) {
    pass(Option::Some(value));
}
```

---

## Best Practices

### ✅ DO: Be Explicit

```aria
func:calculate = flt64() {
    pass(42.0);
}
```

### ✅ DO: Use for Clarity

```aria
// Clear contract
func:validate = bool(string:input) {
    pass(input.length() > 0);
}
```

### ❌ DON'T: Omit for Non-Void

```aria
// Wrong: Missing return type
func:compute = NIL(int32:x) {  // Should specify -> i32
    pass(x * 2);
}

// Right
func:compute = int32(int32:x) {
    pass(x * 2);
}
```

---

## Related

- [Colon (:)](colon.md) - Type specification
- [Functions](../functions/functions.md)
- [Return Types](../functions/return_types.md)

---

**Remember**: `->` declares what the function **returns**!
