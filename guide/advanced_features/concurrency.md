# Concurrency

## Overview

Aria's concurrency model provides four layers of abstraction:

| Layer | API | Use Case |
|-------|-----|----------|
| Threads | `aria_shim_thread_*` | Direct POSIX thread control |
| Thread Pool | `aria_shim_pool_*` | Task-parallel work distribution |
| Channels | `aria_shim_channel_*` | Typed message-passing between threads |
| Actors | `aria_shim_actor_*` | Self-contained message-processing entities |

All concurrency primitives are accessed via `extern` declarations to the runtime shim layer.

---

## 1. Threads

### Spawning a Thread

```aria
extern func:aria_shim_thread_spawn = int64(int64:func_ptr, int64:arg);
extern func:aria_shim_thread_join = int32(int64:handle);
extern func:aria_shim_thread_detach = int32(int64:handle);
extern func:aria_shim_thread_yield = NIL();
extern func:aria_shim_thread_sleep_ms = NIL(int64:ms);
extern func:aria_shim_thread_hardware_concurrency = int32();
extern func:aria_shim_thread_current_id = int64();

func:worker = int64(int64:arg) {
    // Worker body — arg is user-provided data
    pass arg * 2i64;
}

func:main = int32() {
    int64:handle = aria_shim_thread_spawn(@worker, 42i64);
    drop aria_shim_thread_join(handle);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

---

## 2. Thread Pool

```aria
extern func:aria_shim_pool_create = int64(int32:num_workers);
extern func:aria_shim_pool_submit = int32(int64:handle, int64:func_ptr, int64:arg);
extern func:aria_shim_pool_shutdown = int32(int64:handle);
extern func:aria_shim_pool_wait_idle = int32(int64:handle);
extern func:aria_shim_pool_active_tasks = int64(int64:handle);
extern func:aria_shim_pool_pending_tasks = int64(int64:handle);

func:task = int64(int64:arg) {
    pass arg + 1i64;
}

func:main = int32() {
    int32:cores = aria_shim_thread_hardware_concurrency();
    int64:pool = aria_shim_pool_create(cores);

    // Submit work
    loop(0i64, 100i64, 1i64) {
        drop aria_shim_pool_submit(pool, @task, $);
    }

    // Wait for all tasks to complete
    drop aria_shim_pool_wait_idle(pool);
    drop aria_shim_pool_shutdown(pool);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

---

## 3. Channels

Typed message-passing between threads. Three channel modes:

| Mode | Creation | Behavior |
|------|----------|----------|
| Buffered | `channel_create(capacity)` | Send blocks when full, recv blocks when empty |
| Unbuffered | `channel_create_unbuffered()` | Send blocks until receiver is ready (rendezvous) |
| Oneshot | `channel_create_oneshot()` | Single send, single recv, then auto-closes |

```aria
extern func:aria_shim_channel_create = int64(int32:capacity);
extern func:aria_shim_channel_send = int32(int64:handle, int64:value);
extern func:aria_shim_channel_recv = int64(int64:handle);
extern func:aria_shim_channel_try_send = int32(int64:handle, int64:value);
extern func:aria_shim_channel_try_recv = int64(int64:handle);
extern func:aria_shim_channel_close = int32(int64:handle);
extern func:aria_shim_channel_destroy = int32(int64:handle);
extern func:aria_shim_channel_count = int32(int64:handle);
extern func:aria_shim_channel_is_closed = int32(int64:handle);

func:producer = int64(int64:ch) {
    loop(1i64, 10i64, 1i64) {
        drop aria_shim_channel_send(ch, $);
    }
    drop aria_shim_channel_close(ch);
    pass 0i64;
}

func:consumer = int64(int64:ch) {
    int64:sum = 0i64;
    while (aria_shim_channel_is_closed(ch) == 0i32) {
        int64:val = aria_shim_channel_recv(ch);
        sum = sum + val;
    }
    pass sum;
}

func:main = int32() {
    int64:ch = aria_shim_channel_create(16i32);
    int64:prod = aria_shim_thread_spawn(@producer, ch);
    int64:cons = aria_shim_thread_spawn(@consumer, ch);

    drop aria_shim_thread_join(prod);
    drop aria_shim_thread_join(cons);
    drop aria_shim_channel_destroy(ch);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

### Channel Select

Wait on multiple channels simultaneously:

```aria
extern func:aria_shim_channel_select2 = int32(int64:ch0, int64:ch1, int64:timeout_ms);

// Returns index of ready channel (0 or 1), or -1 on timeout
int32:ready = aria_shim_channel_select2(ch_a, ch_b, 5000i64);
```

---

## 4. Actors

Self-contained entities that process messages from a mailbox (built on channels + threads):

```aria
extern func:aria_shim_actor_spawn = int64(int64:behavior_func);
extern func:aria_shim_actor_send = int32(int64:handle, int64:message);
extern func:aria_shim_actor_stop = int32(int64:handle);
extern func:aria_shim_actor_destroy = int32(int64:handle);
extern func:aria_shim_actor_is_alive = int32(int64:handle);
extern func:aria_shim_actor_pending = int32(int64:handle);
```

---

## 5. Synchronization Primitives

### Mutexes

```aria
extern func:aria_shim_mutex_create = int64(int32:type);
extern func:aria_shim_mutex_lock = int32(int64:handle);
extern func:aria_shim_mutex_trylock = int32(int64:handle);
extern func:aria_shim_mutex_unlock = int32(int64:handle);
extern func:aria_shim_mutex_destroy = int32(int64:handle);
```

### Condition Variables

```aria
extern func:aria_shim_condvar_create = int64();
extern func:aria_shim_condvar_wait = int32(int64:cv, int64:mutex);
extern func:aria_shim_condvar_timedwait = int32(int64:cv, int64:mutex, int64:timeout_ns);
extern func:aria_shim_condvar_signal = int32(int64:handle);
extern func:aria_shim_condvar_broadcast = int32(int64:handle);
extern func:aria_shim_condvar_destroy = int32(int64:handle);
```

### Read-Write Locks

```aria
extern func:aria_shim_rwlock_create = int64();
extern func:aria_shim_rwlock_rdlock = int32(int64:handle);
extern func:aria_shim_rwlock_wrlock = int32(int64:handle);
extern func:aria_shim_rwlock_unlock = int32(int64:handle);
extern func:aria_shim_rwlock_destroy = int32(int64:handle);
```

---

## 6. Atomics

Low-level atomic operations with explicit memory ordering:

```aria
extern func:aria_shim_atomic_int64_create = int64(int64:initial);
extern func:aria_shim_atomic_int64_load = int64(int64:handle, int32:order);
extern func:aria_shim_atomic_int64_store = int32(int64:handle, int64:value, int32:order);
extern func:aria_shim_atomic_int64_fetch_add = int64(int64:handle, int64:value, int32:order);
extern func:aria_shim_atomic_int64_exchange = int64(int64:handle, int64:value, int32:order);
extern func:aria_shim_atomic_int64_destroy = int32(int64:handle);
```

Memory ordering constants (pass as `int32:order`):

| Keyword | Value | Guarantee |
|---------|-------|-----------|
| `relaxed` | 0 | No synchronization |
| `acquire` | 1 | Reads after this see prior writes |
| `release` | 2 | Writes before this visible to acquirers |
| `acq_rel` | 3 | Combined acquire + release |
| `seq_cst` | 4 | Sequentially consistent (strongest) |

---

## Async/Await

The `async` keyword declares functions that can suspend and resume:

```aria
async func:fetch_data = string(string:url) {
    // Async operations
    pass data;
}
```

Use `await` to suspend until the async operation completes:

```aria
string:result = await fetch_data("https://example.com");
```

See [functions/async.md](../functions/async.md) for full async documentation.

---

## Related

- [functions/async.md](../functions/async.md) — async function syntax
- [verification.md](verification.md) — `--verify-concurrency` for data race detection
- [safety_layers.md](safety_layers.md) — safety levels for concurrent code
