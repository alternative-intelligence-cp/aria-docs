# Concurrency Verification

## Overview

With `--verify-concurrency`, the compiler detects data races and deadlocks at
compile time. Introduced in v0.14.2, this is the most advanced verification domain
in Aria.

## Data Race Detection

A data race occurs when two threads access the same shared variable, at least one
access is a write, and there is no synchronization between them.

The compiler detects shared variables (globals accessed from functions used as
thread entry points) and verifies that writes are properly guarded.

### Unprotected Access (Disproven)

```aria
int32:counter = 0i32;

func:unsafe_writer = int64(int64:arg) {
    counter = counter + 1i32;    // Disproven: unprotected write to shared variable
    pass 0i64;
};
```

### Mutex-Protected Access (Proven)

```aria
int32:counter = 0i32;
extern func:aria_shim_mutex_lock = int32(int64:mtx);
extern func:aria_shim_mutex_unlock = int32(int64:mtx);

func:safe_writer = int64(int64:arg) {
    drop aria_shim_mutex_lock(global_mtx);
    counter = counter + 1i32;           // Proven: mutex-protected
    drop aria_shim_mutex_unlock(global_mtx);
    pass 0i64;
};
```

## Lock Region Recognition

The solver recognizes these synchronization primitives:

| Primitive | Lock Call | Unlock Call |
|-----------|-----------|-------------|
| Mutex | `aria_shim_mutex_lock` | `aria_shim_mutex_unlock` |
| RWLock (write) | `aria_shim_rwlock_wrlock` | `aria_shim_rwlock_unlock` |
| RWLock (read) | `aria_shim_rwlock_rdlock` | `aria_shim_rwlock_unlock` |

A variable access is "protected" if it occurs between a matching lock/unlock pair
on any of these primitives.

## Read-Write Lock Support

The solver distinguishes between read and write locks:

```aria
// Multiple readers — safe
func:reader = int64(int64:arg) {
    drop aria_shim_rwlock_rdlock(global_rwlock);
    int32:val = shared_data;              // Proven: read-lock protected
    drop aria_shim_rwlock_unlock(global_rwlock);
    pass 0i64;
};

// Single writer — safe
func:writer = int64(int64:arg) {
    drop aria_shim_rwlock_wrlock(global_rwlock);
    shared_data = 42i32;                   // Proven: write-lock protected
    drop aria_shim_rwlock_unlock(global_rwlock);
    pass 0i64;
};
```

## Atomic Exemptions

Variables accessed through atomic operations are exempt from race detection:

```aria
extern func:aria_shim_atomic_store = NIL(int64:ptr, int32:val);
extern func:aria_shim_atomic_load = int32(int64:ptr);

func:atomic_writer = int64(int64:arg) {
    drop aria_shim_atomic_store(counter_ptr, 42i32);  // Exempt: atomic
    pass 0i64;
};
```

## Channel-Based Transfer

Variables sent through channels transfer ownership. The solver recognizes channel
send/receive patterns as synchronization:

```aria
extern func:aria_shim_channel_send = int32(int64:ch, int64:val);
extern func:aria_shim_channel_recv = int64(int64:ch);

func:producer = int64(int64:arg) {
    int32:data = 42i32;
    drop aria_shim_channel_send(chan, data);   // Ownership transferred
    pass 0i64;
};
```

## Deadlock Detection

The solver tracks lock ordering across functions. Inconsistent ordering is flagged:

```aria
// UNSAFE — cyclic lock ordering
func:lock_ab = NIL() { lock(mtx_a); lock(mtx_b); unlock(mtx_b); unlock(mtx_a); };
func:lock_ba = NIL() { lock(mtx_b); lock(mtx_a); unlock(mtx_a); unlock(mtx_b); };
// Disproven: potential deadlock — A→B then B→A

// SAFE — consistent ordering
func:lock_ab2 = NIL() { lock(mtx_a); lock(mtx_b); unlock(mtx_b); unlock(mtx_a); };
func:lock_ab3 = NIL() { lock(mtx_a); lock(mtx_b); unlock(mtx_b); unlock(mtx_a); };
// Proven: deadlock-free (consistent A→B ordering)
```

## Understanding Results

`--verify-concurrency --prove-report` shows per-access verdicts:

```
[race] Proven:  'counter' in safe_writer() — mutex-protected
[race] Disproven: 'counter' in unsafe_writer() — unprotected write
[race] Exempt: 'counter' in atomic_writer() — atomic operation
```

## Common Patterns That Pass

1. **Lock-all-unlock-all** — acquire all locks, do work, release in reverse order
2. **Single-owner** — variable only accessed from one thread
3. **Read-only sharing** — multiple readers, no writers (rwlock or immutable)
4. **Channel transfer** — ownership moves between threads via channels
5. **Atomic operations** — lock-free access to simple values

## Avoiding False Positives

The solver is conservative — it may flag code that is actually safe if it can't
recognize the synchronization pattern. Common causes:

1. **Custom synchronization** — spinlocks, barriers, or condition variables not
   recognized as lock/unlock pairs
2. **Indirect locking** — locking through a wrapper function the solver doesn't inline
3. **Phase-based access** — thread A writes during phase 1, thread B reads during
   phase 2 (no lock needed, but solver sees both accesses)

**Workaround:** Use the recognized primitives (mutex, rwlock, atomic, channel) or
restructure code so synchronization is visible at the function level.

## Next

- [Troubleshooting](06_troubleshooting.md) — interpreting Unknown results
- [Why Verification](01_why_verification.md) — back to basics
