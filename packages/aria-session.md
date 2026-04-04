# aria-session

**Version**: 0.12.1
**Category**: middleware, http
**License**: MIT

Server-side session management for Aria. Create sessions with unique IDs,
store key-value session variables, and generate Set-Cookie headers.

## API Reference

| Function | Description |
|----------|-------------|
| `session_create(string) → int64` | Create session with ID, returns handle |
| `session_id(int64) → string` | Get session ID |
| `session_is_active(int64) → int64` | Check if session is active (1/0) |
| `session_destroy(int64) → int32` | Destroy session |
| `session_set(int64, string, string) → int32` | Set session variable |
| `session_get(int64, string) → string` | Get session variable |
| `session_has(int64, string) → int64` | Check if variable exists |
| `session_remove(int64, string) → int32` | Remove session variable |
| `session_var_count(int64) → int64` | Count of session variables |
| `session_set_cookie_name(int64, string) → int32` | Set cookie name |
| `session_get_cookie_name(int64) → string` | Get cookie name |
| `session_set_max_age(int64, int64) → int32` | Set max-age in seconds |
| `session_cookie_header(int64) → string` | Generate Set-Cookie header |
| `session_clear_cookie(int64) → string` | Generate cookie-clearing header |

**Type:Session** wrapper available.

## Quick Start

```aria
use "aria_session.aria".*;

func:main = int32() {
    int64:sess = raw session_create("sid_abc123");
    int32:_s = raw session_set(sess, "user", "Alice");
    string:user = raw session_get(sess, "user");
    string:cookie = raw session_cookie_header(sess);
    println(cookie);
    exit(0i32);
};
```

## See Also

- [aria-cookie](aria-cookie.md) — Cookie handling
- [aria-server](aria-server.md) — HTTP server
