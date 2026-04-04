# aria-server

**Version**: 0.12.0
**Category**: networking, http, server
**License**: MIT

HTTP server framework for Aria. Bind, listen, accept clients, parse HTTP
requests, and send typed responses with custom headers.

## Features

- TCP listener with configurable backlog
- Full HTTP request parsing: method, path, query, headers, body
- Response helpers: plain text, custom headers, typed content
- Peer address and port access
- Raw request data access

## API Reference

| Function | Description |
|----------|-------------|
| `srv_listen(string, int64, int64) → int64` | Bind and listen on addr:port |
| `srv_accept(int64) → int64` | Accept a client connection |
| `srv_close(int64) → int64` | Close server socket |
| `srv_close_client(int64) → int64` | Close client connection |
| `srv_error() → string` | Get last error message |
| `srv_read_req(int64) → int64` | Read and parse HTTP request |
| `srv_req_method() → string` | Get request method (GET, POST, etc.) |
| `srv_req_path() → string` | Get request path |
| `srv_req_query() → string` | Get query string |
| `srv_req_body() → string` | Get request body |
| `srv_req_body_len() → int64` | Get body length |
| `srv_req_hdr(string) → string` | Get header value by name |
| `srv_req_hdr_count() → int64` | Get number of headers |
| `srv_respond(int64, int64, string) → int64` | Send plain text response |
| `srv_respond_hdrs(int64, int64, string, string) → int64` | Send response with headers |
| `srv_respond_typed(int64, int64, string, string) → int64` | Send response with content type |
| `srv_peer_addr() → string` | Get client IP address |
| `srv_peer_port() → int64` | Get client port |

**Type:Server** wrapper with full method set (see source).

## Quick Start

```aria
use "aria_server.aria".*;

func:main = int32() {
    int64:fd = raw srv_listen("0.0.0.0", 8080i64, 10i64);
    int64:client = raw srv_accept(fd);
    int64:_r = raw srv_read_req(client);
    string:method = raw srv_req_method();
    string:path = raw srv_req_path();
    int64:_s = raw srv_respond(client, 200i64, "Hello from Aria!");
    int64:_c = raw srv_close_client(client);
    int64:_d = raw srv_close(fd);
    exit(0i32);
};
```

## Dependencies

Requires C shim (`libaria_server_shim.so`).

## See Also

- [aria-socket](aria-socket.md) — Low-level sockets
- [aria-cors](aria-cors.md) — CORS middleware
- [aria-session](aria-session.md) — Session management
