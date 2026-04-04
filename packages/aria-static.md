# aria-static

**Version**: 0.12.1
**Category**: middleware, http, filesystem
**License**: MIT

Static file serving middleware for Aria. Resolve URL paths to filesystem paths,
detect MIME types, and serve file content with proper headers.

## API Reference

| Function | Description |
|----------|-------------|
| `static_reset() → int64` | Create static file server handle |
| `static_set_root(int64, string) → int32` | Set document root path |
| `static_get_root(int64) → string` | Get document root path |
| `static_set_index(int64, string) → int32` | Set default index file |
| `static_detect_mime(string) → string` | Detect MIME type from file path |
| `static_resolve(int64, string) → int32` | Resolve URL path to file |
| `static_resolved_path(int64) → string` | Get resolved filesystem path |
| `static_mime_type(int64) → string` | Get MIME type of resolved file |
| `static_error(int64) → string` | Get error message |
| `static_content(int64) → string` | Get file content |
| `static_length(int64) → int64` | Get content length |

**Type:Static** wrapper available.

## Quick Start

```aria
use "aria_static.aria".*;

func:main = int32() {
    int64:srv = raw static_reset();
    int32:_r = raw static_set_root(srv, "/var/www/html");
    int32:_i = raw static_set_index(srv, "index.html");
    string:mime = raw static_detect_mime("style.css");
    println(mime);
    exit(0i32);
};
```

## See Also

- [aria-server](aria-server.md) — HTTP server
- [aria-body-parser](aria-body-parser.md) — Request body parsing
