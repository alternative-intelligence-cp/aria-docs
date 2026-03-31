# Colon Operator (:)

**Category**: Operators → Type Declaration  
**Operator**: `:`  
**Purpose**: Specify variable types

---

## Syntax

```aria
<variable>: <type>
<variable>: <type> = <value>
```

---

## Description

The colon operator `:` declares a variable's type.

---

## Basic Usage

```aria
// Type declaration
x: i32;

// With initialization
y: i32 = 42;

// Type inference (no colon)
z := 42;  // Type inferred as i32
```

---

## Function Parameters

```aria
func:greet = NIL(string:name, int32:age) {
    print("Hello " + name + ", age " + age);
}
```

---

## Return Types

```aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## Struct Fields

```aria
struct User {
    name: string,
    age: i32,
    email: string
}
```

---

## Type Annotations

```aria
// Explicit type
numbers: []i32 = [1, 2, 3];

// Map type
users: Map<string, User> = Map::new();

// Optional type
maybe: i32? = nil;
```

---

## Best Practices

### ✅ DO: Be Explicit When Unclear

```aria
// Clear intent
config: Config = load_config();

// Ambiguous without type
data := load();  // What type?
```

### ✅ DO: Use for Function Signatures

```aria
func:calculate = flt64(flt64:input, int32:precision) {
    ...
}
```

### ❌ DON'T: Over-Annotate

```aria
// Unnecessary
x: i32 = 42;  // Type obvious

// Better
x := 42;  // Type inferred
```

---

## Related

- [Assignment (=)](assign.md)
- [Arrow (->)](arrow.md) - Return types
- [Type Inference](../types/type_inference.md)

---

**Remember**: `:` declares **types**, `=` assigns **values**!
