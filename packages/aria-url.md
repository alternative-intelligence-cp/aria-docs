# aria-url

**Version**: 0.12.0
**Category**: networking, url
**License**: MIT

URL parsing, encoding, and decoding for Aria. Decomposes URLs into
scheme, host, port, path, query, and fragment components.

## API Reference

| Function | Description |
|----------|-------------|
| `parse(string) → int32` | Parse a URL string |
| `get_scheme() → string` | Get URL scheme (http, https, etc.) |
| `get_host() → string` | Get hostname |
| `get_port() → int32` | Get port number |
| `get_path() → string` | Get URL path |
| `get_query() → string` | Get query string |
| `get_fragment() → string` | Get fragment identifier |
| `encode(string) → string` | URL-encode a string |
| `decode(string) → string` | URL-decode a string |

## Quick Start

```aria
use "aria_url.aria".*;

func:main = int32() {
    int32:_p = raw parse("https://example.com:8080/api?q=test#top");
    string:host = raw get_host();
    int32:port = raw get_port();
    string:path = raw get_path();
    println(host);
    exit(0i32);
};
```

## See Also

- [aria-http](aria-http.md) — HTTP client
- [aria-dns](aria-dns.md) — DNS resolution
