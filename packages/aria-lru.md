# aria-lru

**Version**: 0.12.3
**Category**: data-structures, cache
**License**: MIT

LRU (Least Recently Used) cache for Aria. Pure Aria implementation using
ahash with clock-based access tracking. Automatic eviction of least recently
used entries when capacity is reached.

## Features

- O(1) get/put via ahash
- Clock-counter-based access time tracking
- Automatic LRU eviction on capacity overflow
- Tracks last evicted key

## API Reference

| Function | Description |
|----------|-------------|
| `lru_create(int64) → int64` | Create LRU cache with given capacity |
| `lru_put(int64, string, string) → int32` | Insert or update key-value pair |
| `lru_get(int64, string) → string` | Get value (bumps access time), "" on miss |
| `lru_has(int64, string) → int64` | Check if key exists (1/0) |
| `lru_count(int64) → string` | Current entry count |
| `lru_capacity(int64) → string` | Maximum capacity |
| `lru_last_evicted(int64) → string` | Key of last evicted entry |

**Type:LRU** wrapper with methods: `init`, `put`, `get`, `has`, `cnt`, `cap`, `evicted`.

## Quick Start

```aria
use "aria_lru.aria".*;

func:main = int32() {
    int64:cache = raw lru_create(3i64);
    int32:_p1 = raw lru_put(cache, "a", "100");
    int32:_p2 = raw lru_put(cache, "b", "200");
    int32:_p3 = raw lru_put(cache, "c", "300");
    // Cache full — next put evicts least recently used
    string:val = raw lru_get(cache, "a");
    int32:_p4 = raw lru_put(cache, "d", "400");
    string:evicted = raw lru_last_evicted(cache);
    println(evicted);
    exit(0i32);
};
```

## See Also

- [aria-retry](aria-retry.md) — Retry with backoff
- [aria-glob](aria-glob.md) — Glob pattern matching
