# aria-socket

**Version**: 0.12.0
**Category**: networking, sockets
**License**: MIT

Low-level TCP and UDP socket operations for Aria. Provides connect, listen,
accept, send, receive, and non-blocking I/O through a C shim.

## Features

- TCP client connections and server listeners
- UDP socket creation, binding, send/receive
- Non-blocking I/O with `socket_set_nonblocking`
- Poll-based read readiness check
- Configurable timeouts
- String and raw data send/receive

## API Reference

| Function | Description |
|----------|-------------|
| `socket_tcp_connect(string, int32) → int32` | Connect to host:port, returns fd |
| `socket_tcp_listen(string, int32, int32) → int32` | Bind and listen on addr:port |
| `socket_tcp_accept(int32) → int32` | Accept incoming connection |
| `socket_udp_create() → int32` | Create UDP socket |
| `socket_udp_bind(int32, string, int32) → int32` | Bind UDP socket to addr:port |
| `socket_udp_sendto(int32, string, int32, string, int32) → int32` | Send UDP datagram |
| `socket_udp_recvfrom(int32) → string` | Receive UDP datagram |
| `socket_send(int32, string, int32) → int32` | Send raw bytes |
| `socket_send_str(int32, string) → int32` | Send string data |
| `socket_recv(int32, int32) → string` | Receive up to N bytes |
| `socket_recv_length() → int64` | Get last receive length |
| `socket_set_nonblocking(int32) → int32` | Set non-blocking mode |
| `socket_poll_read(int32, int32) → int32` | Poll for read readiness |
| `socket_close(int32) → NIL` | Close socket fd |
| `socket_set_timeout(int32, int32) → int32` | Set socket timeout |
| `socket_error() → string` | Get last error message |

## Quick Start

```aria
use "aria_socket.aria".*;

func:main = int32() {
    int32:fd = raw socket_tcp_connect("example.com", 80i32);
    int32:_s = raw socket_send_str(fd, "GET / HTTP/1.0\r\nHost: example.com\r\n\r\n");
    string:resp = raw socket_recv(fd, 4096i32);
    println(resp);
    drop socket_close(fd);
    exit(0i32);
};
```

## Dependencies

Requires C shim (`libaria_socket_shim.so`).

## See Also

- [aria-server](aria-server.md) — HTTP server framework
- [aria-http](aria-http.md) — High-level HTTP client
