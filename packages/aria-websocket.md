# aria-websocket

**Version**: 0.12.2
**Category**: networking, protocol
**License**: MIT

WebSocket protocol layer for Aria. Pure Aria implementation — manages
connection state, handshake generation, frame type classification, and
close negotiation. Rebuilt from FFI to pure Aria in v0.12.2.

## API Reference

| Function | Description |
|----------|-------------|
| `ws_create(string, string) → int64` | Create WebSocket handle (host, path) |
| `ws_handshake_request(int64) → string` | Generate HTTP upgrade request |
| `ws_check_accept(int64, string) → int64` | Validate server accept response |
| `ws_set_state(int64, string) → int32` | Set connection state |
| `ws_state(int64) → string` | Get current connection state |
| `ws_host(int64) → string` | Get host |
| `ws_path(int64) → string` | Get path |
| `ws_frame_type_name(int64) → string` | Get frame type name by opcode |
| `ws_is_control_frame(int64) → int64` | Check if opcode is control frame |
| `ws_is_data_frame(int64) → int64` | Check if opcode is data frame |
| `ws_set_close(int64, string, string) → int32` | Set close code and reason |
| `ws_close_code(int64) → string` | Get close code |
| `ws_close_reason(int64) → string` | Get close reason |
| `ws_set_protocol(int64, string) → int32` | Set sub-protocol |
| `ws_protocol(int64) → string` | Get sub-protocol |
| `ws_set_extensions(int64, string) → int32` | Set extensions |
| `ws_extensions(int64) → string` | Get extensions |

**Type:WebSocket** wrapper available.

## Quick Start

```aria
use "aria_websocket.aria".*;

func:main = int32() {
    int64:ws = raw ws_create("echo.websocket.org", "/");
    string:req = raw ws_handshake_request(ws);
    println(req);
    string:state = raw ws_state(ws);
    string:fname = raw ws_frame_type_name(1i64);
    println(fname);
    exit(0i32);
};
```

## See Also

- [aria-socket](aria-socket.md) — Raw sockets for transport
- [aria-http](aria-http.md) — HTTP client
