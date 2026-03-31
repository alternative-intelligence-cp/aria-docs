# Handle<T> — Safe Arena Pointers

## Overview

`Handle<T>` is a generational index for arena-allocated data. It prevents
use-after-free (CWE-416) by tracking generation counters.

## Structure

```
Handle<T> = { uint64:index, uint32:generation }
// 12 bytes data + 4 padding = 16 bytes, 8-byte aligned
```

## Usage

```aria
// Create arena and allocate
Handle<Point>:h = arena.alloc(Point{x: 1.0, y: 2.0});

// Access (checks generation — returns ERR if stale)
Point:p = arena.get(h) ? default_point;

// Update
arena.set(h, Point{x: 3.0, y: 4.0});

// Free (increments generation — future gets return ERR)
arena.free(h);

// Grow arena (increments ALL generation counters)
arena.grow();
```

## Why Not Raw Pointers?

| Problem | Raw Pointer | Handle<T> |
|---------|-------------|-----------|
| Use-after-free | Undefined behavior | ERR (generation mismatch) |
| Dangling reference | Silent corruption | Detected at access time |
| Double free | Undefined behavior | ERR on second free |

## Related

- [wild.md](wild.md) — unmanaged memory (when Handles aren't appropriate)
- [overview.md](overview.md) — memory model overview
