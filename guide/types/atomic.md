# Atomic Types — `atomic<T>`

## Overview

`atomic<T>` provides lock-free concurrent access to a wrapped value. Lock-free when
T ≤ 64 bits; locked (mutex-backed) for larger types.

## Declaration

```aria
atomic<int64>:counter = atomic_new(0);
```

## Memory Ordering

Default ordering is **Sequential Consistency** (SeqCst) — the safest option.
Weaker orderings require `unsafe { }`:

| Ordering | Safety | Use Case |
|----------|--------|----------|
| SeqCst | Safe (default) | General purpose |
| Acquire/Release | Requires unsafe | Producer/consumer patterns |
| Relaxed | Requires unsafe | Counters where ordering doesn't matter |

## Operations

```aria
int64:val = counter.load();              // read
counter.store(42);                        // write
int64:old = counter.swap(99);            // exchange
bool:ok = counter.compare_exchange(99, 100);  // CAS
int64:prev = counter.fetch_add(1);       // atomic increment
int64:prev = counter.fetch_sub(1);       // atomic decrement
```

Weak CAS (`compare_exchange_weak`) may spuriously fail — use in a loop.

## ERR Sentinel

The TBB ERR sentinel value remains atomic — ERR propagation works through atomic operations.

## Status

Partial implementation: basic operations complete, advanced orderings in progress.

## Related

- [advanced_features/concurrency.md](../advanced_features/concurrency.md) — threads, channels
- [tbb.md](tbb.md) — TBB error propagation through atomics
