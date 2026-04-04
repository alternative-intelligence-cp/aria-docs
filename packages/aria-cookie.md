# aria-cookie

**Version**: 0.12.1
**Category**: middleware, http
**License**: MIT

HTTP cookie parsing and building for Aria. Parse `Cookie` headers into
key-value pairs, and build `Set-Cookie` headers with full attribute support.

## Features

- Parse cookie headers into queryable key-value store
- Cookie builder with domain, path, max-age, secure, httponly, samesite
- Count, lookup, and remove parsed cookies

## API Reference

| Function | Description |
|----------|-------------|
| `cookie_parse(string) → int64` | Parse Cookie header, returns handle |
| `cookie_get(int64, string) → string` | Get cookie value by name |
| `cookie_has(int64, string) → int64` | Check if cookie exists (1/0) |
| `cookie_count(int64) → int64` | Number of parsed cookies |
| `cookie_remove(int64, string) → int64` | Remove a cookie by name |
| `cookie_bld_reset() → int64` | Create new cookie builder |
| `cookie_bld_name(int64, string) → int64` | Set cookie name |
| `cookie_bld_value(int64, string) → int64` | Set cookie value |
| `cookie_bld_domain(int64, string) → int64` | Set Domain attribute |
| `cookie_bld_path(int64, string) → int64` | Set Path attribute |
| `cookie_bld_max_age(int64, int64) → int64` | Set Max-Age in seconds |
| `cookie_bld_secure(int64, int64) → int64` | Set Secure flag |
| `cookie_bld_httponly(int64, int64) → int64` | Set HttpOnly flag |
| `cookie_bld_samesite(int64, string) → int64` | Set SameSite policy |
| `cookie_bld_build(int64) → string` | Build Set-Cookie header string |

**Type:Cookie** wrapper available.

## Quick Start

```aria
use "aria_cookie.aria".*;

func:main = int32() {
    int64:jar = raw cookie_parse("session=abc123; theme=dark");
    string:sess = raw cookie_get(jar, "session");
    int64:bld = raw cookie_bld_reset();
    int64:_n = raw cookie_bld_name(bld, "token");
    int64:_v = raw cookie_bld_value(bld, "xyz");
    int64:_s = raw cookie_bld_secure(bld, 1i64);
    int64:_h = raw cookie_bld_httponly(bld, 1i64);
    string:hdr = raw cookie_bld_build(bld);
    println(hdr);
    exit(0i32);
};
```

## See Also

- [aria-session](aria-session.md) — Session management
- [aria-server](aria-server.md) — HTTP server
