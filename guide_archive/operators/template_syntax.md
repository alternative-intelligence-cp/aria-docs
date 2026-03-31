# Template Syntax

**Category**: Operators → String  
**Purpose**: String templates and interpolation

---

See [String Interpolation](interpolation.md) and [Template Literal](backtick.md) for complete documentation.

---

## Quick Reference

### Interpolation

```aria
// Backtick template literals with &{} interpolation
string:msg = `Hello, &{name}!`;

// Embed expressions
string:text = `Total: &{price * quantity}`;
```

### Raw Strings

```aria
// No escape processing
string:path = `C:\Windows\System32`;
```

### Multi-line

```aria
string:template = `
    Name: &{name}
    Age: &{age}
    Email: &{email}
`;
```

---

## Related

- [String Interpolation](interpolation.md)
- [Backtick (`)](backtick.md)
- [Strings](../types/string.md)

---

**Remember**: `$` for interpolation, `` ` `` for raw strings!
