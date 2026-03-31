# String Interpolation (&{})

**Category**: Operators → String  
**Operator**: `&{expr}` in template literals  
**Purpose**: Embed expressions in backtick strings

---

See [String Interpolation](interpolation.md) for complete documentation.

---

## Quick Reference

```aria
// Backtick template literals support &{} interpolation
string:name = "Alice";
string:msg = `Hello, &{name}!`;

// Expression
int32:x = 10;
string:text = `Value: &{x + 5}`;

// Method calls
string:info = `Status: &{user.status()}`;
```

---

**Remember**: Use backtick strings with `&{expression}` for interpolation. Double-quoted strings do not interpolate.
