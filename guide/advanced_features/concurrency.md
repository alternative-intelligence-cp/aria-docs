# Concurrency

## Async Functions

```aria
async func:fetch = string(string:url) {
    pass data;
}
```

## Threads

The runtime has POSIX thread support (thread.cpp ~898 LOC) + atomics (~1,301 LOC).
Stdlib wrappers provide higher-level abstractions.

## Channels

Typed message-passing between threads:

```aria
// Channel creation and send/receive patterns
// (stdlib wrapper API)
```

## Atomics

See [types/atomic.md](../types/atomic.md) for `atomic<T>` usage.

## Sync Primitives

- `sync` — synchronization primitive marker
- `atomic` — atomic operation marker
- Mutexes, condition variables via stdlib

## Related

- [types/atomic.md](../types/atomic.md) — atomic<T> type
- [functions/async.md](../functions/async.md) — async function syntax
