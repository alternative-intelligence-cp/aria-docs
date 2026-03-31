# String Operators

## Concatenation

The `+` operator concatenates strings:

```aria
string:full = ("Hello" + ", " + "world!");
string:name = (first + " " + last);
```

Long chains (7+ operands) work correctly.

## Interpolation — Template Literals

Use backtick strings with `&{expression}` for interpolation:

```aria
string:msg = `Hello, &{name}! You are &{age} years old.`;
string:calc = `Result: &{a + b}`;
int32:x = 42;
println(`x = &{x}`);
```

## Comparison

Strings support `==` and `!=`:

```aria
if (name == "Alice") {
    println("Found Alice");
}
```

## Comments (related syntax)

| Syntax | Type |
|--------|------|
| `//` | Line comment |
| `/* ... */` | Block comment |

## Escape Sequences (in `"..."` strings)

| Escape | Character |
|--------|-----------|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Backslash |
| `\"` | Double quote |

Raw strings (`` `...` `` without `&{}`) do not process escapes.
