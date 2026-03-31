# Atomic Types — `atomic<T>`

## Overview

`atomic<T>` provides lock-free concurrent access to a wrapped value. Lock-free when
T ≤ 64 bits; locked (mutex-backed) for larger types.

## Declaration

Two forms are supported:

```aria
stack atomic<int32>:counter;                       // uninitialized stack allocation
atomic<int32>:counter = atomic_new(0i32);           // initialized with value
stack atomic<bool>:flag;                            // boolean atomic
```

**Type constraint:** `atomic<T>` requires lock-free compatible types:
`int8`–`int64`, `uint8`–`uint64`, `bool`, `tbb8`–`tbb64`.
Floats, strings, and arrays are **rejected** at compile time.

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
int32:val = counter.load();                         // read
counter.store(42i32);                               // write
int32:old = counter.swap(100i32);                   // exchange
bool:ok = counter.compare_exchange(99i32, 100i32);  // CAS
int32:prev = counter.fetch_add(1i32);               // atomic increment
int32:prev = counter.fetch_sub(1i32);               // atomic decrement
```

Weak CAS (`compare_exchange_weak`) may spuriously fail — use in a loop.

## ERR Sentinel

The TBB ERR sentinel value remains atomic — ERR propagation works through atomic operations.

## Status

Basic operations (load, store, swap, CAS, fetch_add, fetch_sub) are implemented.
Advanced memory orderings are in progress.

## Related

- [advanced_features/concurrency.md](../advanced_features/concurrency.md) — threads, channels
- [tbb.md](tbb.md) — TBB error propagation through atomics
