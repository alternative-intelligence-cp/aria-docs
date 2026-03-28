# Actors

**Category**: Advanced Features → Concurrency  
**Syntax**: `Actor.spawn()`, `Actor.send()`, `Actor.stop()`  
**Purpose**: Lightweight concurrent entities with isolated state and mailboxes  
**Since**: v0.2.37

---

## Overview

Actors are **independent concurrent entities** that communicate exclusively through message passing. Each actor has:

- A **mailbox** (channel) for receiving messages
- A **behavior function** that processes one message at a time
- An **isolated thread** of execution

Actors never share mutable state — all communication happens via `send()`.

---

## Spawning Actors

```aria
use "actor.aria".*;
use "channel.aria".*;
use "thread.aria".*;

// Define a behavior: receives int64 messages, processes each one
(int64)(int64):my_handler = int64(int64:msg) {
    drop(println("Got message"));
    pass(0i64);
};

// Spawn the actor
int64:actor = aria_shim_actor_spawn(my_handler);
```

---

## Sending Messages

```aria
// Send a message to actor's mailbox
aria_shim_actor_send(actor, 42i64);
aria_shim_actor_send(actor, 100i64);
```

Messages are queued in the actor's mailbox. The behavior function processes them one at a time in FIFO order.

---

## Request/Reply Pattern

Use a shared reply channel for bidirectional communication:

```aria
// Setup reply channel
int64:reply_ch = raw(Channel.create(64i32));
aria_shim_set_reply_channel(reply_ch);

// Handler echoes messages to reply channel
(int64)(int64):echo = int64(int64:msg) {
    aria_shim_reply(msg);
    pass(0i64);
};

int64:actor = aria_shim_actor_spawn(echo);
aria_shim_actor_send(actor, 42i64);

// Wait for reply
int64:response = raw(Channel.recv(reply_ch));
// response == 42
```

---

## Lifecycle Management

```aria
// Check if actor is running
int32:alive = aria_shim_actor_is_alive(actor);
// 1 = running, 0 = stopped

// Check pending messages
int32:pending = aria_shim_actor_pending(actor);

// Stop the actor (finishes current message, stops processing)
aria_shim_actor_stop(actor);

// Destroy and free resources
aria_shim_actor_destroy(actor);
```

---

## Actor Mailbox Access

Each actor has an internal channel. You can access it directly:

```aria
int64:mailbox = aria_shim_actor_mailbox(actor);
// mailbox is a channel handle
```

---

## Multiple Actors

```aria
// Spawn a pool of workers
int64:worker1 = aria_shim_actor_spawn(handler);
int64:worker2 = aria_shim_actor_spawn(handler);
int64:worker3 = aria_shim_actor_spawn(handler);

// Distribute work
aria_shim_actor_send(worker1, 1i64);
aria_shim_actor_send(worker2, 2i64);
aria_shim_actor_send(worker3, 3i64);

// Wait for processing
raw(Thread.sleep_ms(50i64));

// Cleanup
aria_shim_actor_stop(worker1);
aria_shim_actor_stop(worker2);
aria_shim_actor_stop(worker3);
aria_shim_actor_destroy(worker1);
aria_shim_actor_destroy(worker2);
aria_shim_actor_destroy(worker3);
```

---

## Rules

1. Actors process messages **sequentially** — one at a time, no data races
2. Always `stop()` then `destroy()` actors when done
3. Messages are `int64` values (encode structured data as needed)
4. Actor behavior functions have signature `(int64)(int64)` — take message, return status
5. The reply channel pattern requires explicit setup with `aria_shim_set_reply_channel`
6. Small delays (`Thread.sleep_ms`) may be needed for actor thread startup

---

## See Also

- [Channels](channels.md) — Underlying message passing mechanism
- [Threading](threading.md) — Thread primitives
- [Concurrency](concurrency.md) — Concurrency overview
