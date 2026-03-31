# String Interpolation (&{})

**Category**: Operators → String  
**Operator**: `&{expr}` in template literals  
**Purpose**: Embed expressions in backtick-delimited template strings

---

## Syntax

```aria
`text &{expression} more text`
```

Interpolation uses `&{` ... `}` inside **backtick** template literals only.
Double-quoted strings (`"..."`) do **not** support interpolation.

---

## Description

Template literals (backtick strings) support string interpolation via `&{expression}`.
Any expression can be placed inside `&{}` — the result is converted to a string
and spliced into the surrounding text.

---

## Basic Usage

```aria
string:name = "Alice";
int32:age = 25;

// Simple interpolation
string:msg = `Hello, &{name}!`;
// "Hello, Alice!"

// With expressions
string:info = `Age: &{age + 1}`;
// "Age: 26"
```

---

## Expression Interpolation

```aria
int32:x = 10;
int32:y = 20;

string:result = `&{x} + &{y} = &{x + y}`;
// "10 + 20 = 30"
```

---

## Method Calls

```aria
User:user = get_user();

string:msg = `User: &{user.name()}, Status: &{user.status()}`;
```

---

## Best Practices

### ✅ DO: Use for Readability

```aria
// Clear — backtick template literal
string:msg = `User &{name} has &{count} items`;

// Also valid — concatenation with double-quoted strings
string:msg = "User " + name + " has " + to_string(count) + " items";
```

### ✅ DO: Use for Complex Expressions

```aria
string:text = `Result: &{calculate(x, y)}`;
```

### ❌ DON'T: Use Double Quotes for Interpolation

```aria
// ❌ WRONG — double quotes have no interpolation
string:msg = "Hello &{name}";  // Literal text, not interpolated

// ✅ CORRECT — backtick template literal
string:msg = `Hello &{name}`;
```

---

## Related

- [Strings](../types/string.md)
- [Template Syntax](template_syntax.md)
- [Backtick (`)](backtick.md)
- [String Concatenation](add.md)

---

**Remember**: `$` in strings **interpolates** expressions!
