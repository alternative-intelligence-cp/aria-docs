# Channels

**Category**: Advanced Features → Concurrency  
**Syntax**: `Channel.create()`, `Channel.send()`, `Channel.recv()`  
**Purpose**: Thread-safe message passing between concurrent tasks  
**Since**: v0.2.37

---

## Overview

Channels are the primary mechanism for **safe, lock-free communication** between threads. A channel is a typed FIFO queue that supports blocking send/receive operations. Aria provides buffered, unbuffered (rendezvous), and oneshot channel modes.

---

## Creating Channels

```aria
use "channel.aria".*;

// Buffered channel (capacity = 16 messages)
int64:ch = raw(Channel.create(16i32));

// Unbuffered (rendezvous) — sender blocks until receiver is ready
int64:rendezvous = raw(Channel.create_unbuffered());

// Oneshot — exactly one message, auto-closes after
int64:once = raw(Channel.create_oneshot());
```

---

## Send and Receive

```aria
// Send a value (blocks if channel is full)
raw(Channel.send(ch, 42i64));

// Receive a value (blocks if channel is empty)
int64:value = raw(Channel.recv(ch));

// Non-blocking try_send (returns -1 if full)
int32:ok = raw(Channel.try_send(ch, 99i64));

// Non-blocking try_recv (returns -1 if empty)
int64:maybe = raw(Channel.try_recv(ch));
```

---

## Producer/Consumer Pattern

```aria
use "channel.aria".*;
use "thread.aria".*;

(int64)(int64):producer = int64(int64:ch) {
    raw(Channel.send(ch, 1i64));
    raw(Channel.send(ch, 2i64));
    raw(Channel.send(ch, 3i64));
    raw(Channel.close(ch));
    pass(0i64);
};

func:main = int32() {
    int64:ch = raw(Channel.create(8i32));
    int64:tid = raw(Thread.spawn(producer, ch));

    int64:sum = 0i64;
    for (int32:i = 0i32; i < 3i32; i += 1i32) {
        int64:v = raw(Channel.recv(ch));
        sum += v;
    };
    raw(Thread.join(tid));
    raw(Channel.destroy(ch));
    // sum == 6
    pass(0i32);
};
```

---

## Channel Modes

| Mode | Create Function | Behavior |
|------|----------------|----------|
| Buffered | `Channel.create(capacity)` | Async up to capacity, then blocks |
| Unbuffered | `Channel.create_unbuffered()` | Sender+receiver must rendezvous |
| Oneshot | `Channel.create_oneshot()` | One message, auto-closes |

Query the mode:
```aria
int32:mode = raw(Channel.get_mode(ch));
// 0 = buffered, 1 = unbuffered, 2 = oneshot
```

---

## Backpressure

Buffered channels enforce backpressure. When full, `send()` blocks and `try_send()` returns `-1`:

```aria
int64:ch = raw(Channel.create(4i32));
raw(Channel.send(ch, 1i64));
raw(Channel.send(ch, 2i64));
raw(Channel.send(ch, 3i64));
raw(Channel.send(ch, 4i64));

// Channel full — this would block:
// raw(Channel.send(ch, 5i64));

// Non-blocking check:
int32:result = raw(Channel.try_send(ch, 5i64));  // returns -1
```

---

## Select (Multi-Channel Wait)

Wait on two channels, receive from whichever is ready first:

```aria
int64:result = raw(Channel.select2(ch_a, ch_b));
```

---

## Channel Utilities

```aria
int32:n = raw(Channel.count(ch));        // Messages currently in channel
int32:cap = raw(Channel.get_capacity(ch));  // Total capacity
raw(Channel.close(ch));                   // Close (no more sends)
raw(Channel.destroy(ch));                 // Free resources
```

---

## High-Level Patterns (aria-channel package)

The `aria-channel` package provides composable patterns:

```aria
use "aria_channel.aria".*;

// Fan-out: distribute one channel to N workers
int64:fanout = FanOut.create(input_ch, 4i32, 16i32);
int64:worker0_ch = FanOut.get_output(fanout, 0i32);

// Fan-in: merge N channels into one
int64:fanin = FanIn.create2(ch_a, ch_b, 32i32);
int64:merged = FanIn.get_output(fanin);

// Pipeline: chain processing stages
int64:stage = Pipeline.stage(in_ch, out_ch, transform_func);
```

---

## Rules

1. Always `destroy()` channels when done to free resources
2. `close()` signals no more sends — receivers still drain remaining messages
3. Sending on a closed channel is undefined behavior
4. Channel handles are `int64` values (opaque handles)
5. Values sent through channels are `int64` — use encoding for other types

---

## See Also

- [Actors](actors.md) — Higher-level actor model built on channels
- [Threading](threading.md) — Thread creation and management
- [Concurrency](concurrency.md) — Overview of concurrency models
