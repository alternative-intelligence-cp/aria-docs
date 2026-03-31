# Standard Library — String Utilities

## Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `string_length(s)` | `(string) → int64` | Character count (UTF-8 aware) |
| `string_concat(a, b)` | `(string, string) → string` | Concatenate two strings |
| `string_contains(s, sub)` | `(string, string) → bool` | Check if substring exists |

## Usage

```aria
use "string_utils.aria".*;

string:name = "Hello, World!";
int64:len = raw string_length(name);      // 13
bool:has = raw string_contains(name, "World");  // true
```

## Related

- [types/string.md](../types/string.md) — string type details
- [operators/string_ops.md](../operators/string_ops.md) — string operators
