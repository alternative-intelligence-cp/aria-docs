# aria-http

**Version**: 0.12.0
**Category**: networking, http
**License**: MIT

HTTP client for Aria. Provides GET, POST, PUT, DELETE, PATCH, and HEAD methods
with configurable headers, timeouts, redirects, and URL encoding.

## Features

- Full HTTP method support: GET, POST, PUT, DELETE, PATCH, HEAD
- Custom request headers with `http_set_header`
- Response status, body, and headers access
- Configurable timeout and redirect following
- URL encoding via `http_url_encode`

## API Reference

| Function | Description |
|----------|-------------|
| `http_init() → int32` | Initialize the HTTP client |
| `http_cleanup() → NIL` | Clean up resources |
| `http_set_header(string) → NIL` | Add a custom request header |
| `http_clear_headers() → NIL` | Remove all custom headers |
| `http_set_timeout(int32) → NIL` | Set timeout in seconds |
| `http_follow_redirects(int32) → NIL` | Enable/disable redirect following |
| `http_set_user_agent(string) → NIL` | Set the User-Agent header |
| `http_get(string) → int32` | Perform GET request |
| `http_post(string, string) → int32` | Perform POST request with body |
| `http_put(string, string) → int32` | Perform PUT request with body |
| `http_delete(string) → int32` | Perform DELETE request |
| `http_patch(string, string) → int32` | Perform PATCH request with body |
| `http_head(string) → int32` | Perform HEAD request |
| `http_status() → int32` | Get response status code |
| `http_body() → string` | Get response body |
| `http_body_length() → int32` | Get response body length |
| `http_response_headers() → string` | Get raw response headers |
| `http_error() → string` | Get error message |
| `http_url_encode(string) → string` | URL-encode a string |

## Quick Start

```aria
use "aria_http.aria".*;

func:main = int32() {
    int32:_i = raw http_init();
    drop http_set_header("Accept: application/json");
    int32:_r = raw http_get("https://api.example.com/data");
    int32:status = raw http_status();
    string:body = raw http_body();
    println(body);
    drop http_cleanup();
    exit(0i32);
};
```

## Dependencies

Requires C shim (`libaria_http_shim.so`) linked with libcurl.

## See Also

- [aria-url](aria-url.md) — URL parsing and encoding
- [aria-dns](aria-dns.md) — DNS resolution
