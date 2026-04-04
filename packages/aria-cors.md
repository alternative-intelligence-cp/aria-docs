# aria-cors

**Version**: 0.12.1
**Category**: middleware, http, security
**License**: MIT

CORS (Cross-Origin Resource Sharing) middleware for Aria. Configure allowed
origins, methods, headers, credentials, and generate proper CORS response headers.

## API Reference

| Function | Description |
|----------|-------------|
| `cors_reset() → int64` | Create CORS configuration handle |
| `cors_allow_origin(int64, string) → int64` | Set allowed origin |
| `cors_allow_methods(int64, string) → int64` | Set allowed methods |
| `cors_allow_headers(int64, string) → int64` | Set allowed headers |
| `cors_allow_credentials(int64, int64) → int64` | Allow credentials (1/0) |
| `cors_expose_headers(int64, string) → int64` | Set exposed headers |
| `cors_max_age(int64, int64) → int64` | Set preflight cache max-age |
| `cors_is_allowed(int64, string) → int64` | Check if origin is allowed |
| `cors_is_preflight(string, string, string) → int64` | Check if request is preflight |
| `cors_headers(int64, string) → string` | Generate CORS headers for origin |
| `cors_preflight(int64, string) → string` | Generate preflight response headers |

**Type:Cors** wrapper available.

## Quick Start

```aria
use "aria_cors.aria".*;

func:main = int32() {
    int64:cfg = raw cors_reset();
    int64:_o = raw cors_allow_origin(cfg, "https://example.com");
    int64:_m = raw cors_allow_methods(cfg, "GET, POST, OPTIONS");
    int64:_h = raw cors_allow_headers(cfg, "Content-Type, Authorization");
    string:hdrs = raw cors_headers(cfg, "https://example.com");
    println(hdrs);
    exit(0i32);
};
```

## See Also

- [aria-server](aria-server.md) — HTTP server
- [aria-cookie](aria-cookie.md) — Cookie handling
