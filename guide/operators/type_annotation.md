# Type Annotation (:)

**Category**: Operators → Types  
**Operator**: `:`  
**Purpose**: Specify types

---

See [Colon (:)](colon.md) for complete documentation.

---

## Quick Reference

```aria
// Variable type
x: i32 = 42;

// Function parameter
func:process = NIL(string:data) { ... }

// Return type (uses ->)
func:calculate = int32() { ... }

// Struct field
struct User {
    name: string,
    age: i32
}
```

---

**Remember**: `:` declares **types**, `->` for **return types**!
