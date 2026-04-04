# Aria Packages Reference

This directory documents the official Aria packages available in `aria-packages`.

## v0.2.8 Packages (Game Development Track)

| Package | Description |
|---------|-------------|
| [aria-raylib](aria-raylib.md) | raylib 2D/audio/input bindings — the core game dev library |
| [aria-tetris](aria-tetris.md) | Full Tetris clone — reference game implementation |
| [aria-gml](aria-gml.md) | GameMaker Language compatibility layer over raylib |
| [aria-opengl](aria-opengl.md) | OpenGL 3.3 Core — custom shaders and GPU-accelerated 3D |

## v0.12.0 Packages (Networking)

| Package | Description |
|---------|-------------|
| [aria-http](aria-http.md) | HTTP client — GET/POST/PUT/DELETE with headers, timeouts |
| [aria-dns](aria-dns.md) | DNS resolution — forward, reverse, validation |
| [aria-socket](aria-socket.md) | TCP/UDP sockets — connect, listen, send, receive |
| [aria-server](aria-server.md) | HTTP server — request parsing, response helpers |
| [aria-url](aria-url.md) | URL parsing — scheme, host, port, path, query, fragment |

## v0.12.1 Packages (Middleware)

| Package | Description |
|---------|-------------|
| [aria-cookie](aria-cookie.md) | HTTP cookies — parsing and Set-Cookie builder |
| [aria-cors](aria-cors.md) | CORS middleware — origin, method, header configuration |
| [aria-body-parser](aria-body-parser.md) | Request body parsing — URL-encoded, field access |
| [aria-session](aria-session.md) | Session management — variables, cookie headers |
| [aria-static](aria-static.md) | Static file serving — MIME detection, path resolution |
| [aria-rate-limit](aria-rate-limit.md) | Token-bucket rate limiting — HTTP headers |

## v0.12.2 Packages (Protocol & Terminal)

| Package | Description |
|---------|-------------|
| [aria-ftp](aria-ftp.md) | FTP command builder and reply parser |
| [aria-smtp](aria-smtp.md) | SMTP command builder, email composer, reply parser |
| [aria-websocket](aria-websocket.md) | WebSocket protocol — handshake, state, frames |
| [aria-display](aria-display.md) | Terminal display — cursor, colors, attributes, dimensions |
| [aria-input](aria-input.md) | Key mapping — bitmask button state, press/release tracking |

## v0.12.3 Packages (Utility)

| Package | Description |
|---------|-------------|
| [aria-lru](aria-lru.md) | LRU cache — clock-based eviction, O(1) get/put |
| [aria-glob](aria-glob.md) | Glob pattern matching — *, **, ? wildcards for file paths |
| [aria-retry](aria-retry.md) | Retry with exponential backoff — configurable cap |

## All Packages

See individual files in this directory for full documentation of each package.

## Quick Start

```bash
# Clone the packages repo
git clone https://github.com/aria-lang/aria-packages
cd aria-packages/packages/aria-gml/shim
make

# Compile a program using aria-gml
ariac my_game.aria -o my_game \
  -L packages/aria-gml/shim \
  -laria_gml_shim -lraylib -lm
```
