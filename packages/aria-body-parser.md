# aria-body-parser

**Version**: 0.12.1
**Category**: middleware, http
**License**: MIT

HTTP request body parser for Aria. Detects content type, parses URL-encoded
form data, and provides field access by name or index.

## API Reference

| Function | Description |
|----------|-------------|
| `body_detect(string) → string` | Detect body type from Content-Type |
| `body_parse(string, string) → int64` | Parse body with content type, returns handle |
| `body_parse_urlencoded(int64, string) → int64` | Parse URL-encoded body |
| `body_field(int64, string) → string` | Get field value by name |
| `body_field_count(int64) → int64` | Number of parsed fields |
| `body_field_key(int64, int64) → string` | Get field key by index |
| `body_field_value(int64, int64) → string` | Get field value by index |
| `body_has_field(int64, string) → int64` | Check if field exists (1/0) |
| `body_raw(int64) → string` | Get raw body content |
| `body_raw_length(int64) → int64` | Get raw body length |
| `body_content_type(int64) → string` | Get detected content type |
| `body_error(int64) → string` | Get parse error message |

**Type:BodyParser** wrapper available.

## Quick Start

```aria
use "aria_body_parser.aria".*;

func:main = int32() {
    int64:bp = raw body_parse("application/x-www-form-urlencoded", "name=Alice&age=30");
    string:name = raw body_field(bp, "name");
    string:age = raw body_field(bp, "age");
    println(name);
    exit(0i32);
};
```

## See Also

- [aria-server](aria-server.md) — HTTP server
- [aria-cookie](aria-cookie.md) — Cookie parsing
