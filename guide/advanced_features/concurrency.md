# Concurrency

**Category**: Advanced Features → Concurrency  
**Purpose**: Execute multiple tasks simultaneously

---

## Overview

Concurrency enables **parallel execution** for better performance and responsiveness.

---

## Concurrency Models

### Threading

```aria
use std.thread;

func:worker = NIL(int32:id) {
    print(`Worker &{id} running`);
}

thread1: Thread = Thread.spawn(worker, 1);
thread2: Thread = Thread.spawn(worker, 2);

thread1.join();
thread2.join();
```

---

### Async/Await

```aria
async func:fetch_data = Result<Data>(string:url) {
    response: Response = await http.get(url)?;
    data: Data = await response.json()?;
    pass(Ok(data));
}

async func:main = NIL() {
    data: Data = await fetch_data("https://api.example.com/data")?;
    print(data);
}
```

---

### Coroutines

```aria
fn* generator() -> i32 {
    yield 1;
    yield 2;
    yield 3;
}

values = generator().collect();
till(values.length - 1, 1) {
    print(values[$]);  // Prints 1, 2, 3
}
```

---

## Thread Safety

### Atomic Operations

```aria
use std.sync.Atomic;

counter: Atomic<i32> = Atomic.new(0);

func:increment = NIL() {
    counter.fetch_add(1);  // Thread-safe
}
```

---

### Mutex

```aria
use std.sync.Mutex;

data: Mutex<i32> = Mutex.new(0);

func:update = NIL() {
    lock: MutexGuard = data.lock();
    *lock += 1;
    // Automatically unlocks when lock goes out of scope
}
```

---

### Channels

```aria
use std.sync.Channel;

channel: Channel<i32> = Channel.new();

// Producer
func:producer = NIL() {
    till(9, 1) {
        channel.send($);
    }
}

// Consumer
func:consumer = NIL() {
    while true {
        value: i32 = channel.recv()?;
        print(`Received: &{value}`);
    }
}
```

---

## Common Patterns

### Producer-Consumer

```aria
use std.sync.{Channel, Thread};

func:producer = NIL(Channel<i32>:ch) {
    till(99, 1) {
        ch.send($);
    }
}

func:consumer = NIL(Channel<i32>:ch) {
    while true {
        match ch.recv() {
            Ok(value) => print(value),
            Err(_) => break,
        }
    }
}

channel: Channel<i32> = Channel.new();
prod: Thread = Thread.spawn(producer, channel.clone());
cons: Thread = Thread.spawn(consumer, channel);

prod.join();
cons.join();
```

---

### Worker Pool

```aria
use std.sync.{Channel, Thread};

struct WorkerPool {
    workers: []Thread,
    sender: Channel<Task>,
}

impl WorkerPool {
    pub func:new = WorkerPool(int32:size) {
        channel: Channel<Task> = Channel.new();
        workers: []Thread = [];
        
        till(size - 1, 1) {
            worker: Thread = Thread.spawn(worker_thread, channel.clone());
            workers.push(worker);
        }
        
        return WorkerPool {
            workers: workers,
            sender: channel,
        };
    }
    
    pub func:execute = NIL(Task:task) {
        self.sender.send(task);
    }
}
```

---

### Async Request Handling

```aria
async func:handle_request = Response(Request:req) {
    // Multiple concurrent operations
    user_task = fetch_user(req.user_id);
    posts_task = fetch_posts(req.user_id);
    
    // Wait for both
    user: User = await user_task?;
    posts: []Post = await posts_task?;
    
    pass(Response.ok(render(user, posts)));
}
```

---

## Synchronization Primitives

### RwLock (Reader-Writer Lock)

```aria
use std.sync.RwLock;

data: RwLock<HashMap<string, i32>> = RwLock.new(HashMap.new());

// Multiple readers
func:read_data = ?i32(string:key) {
    reader: ReadGuard = data.read();
    pass(reader.get(key));
}

// Exclusive writer
func:write_data = NIL(string:key, int32:value) {
    writer: WriteGuard = data.write();
    writer.insert(key, value);
}
```

---

### Barrier

```aria
use std.sync.Barrier;

barrier: Barrier = Barrier.new(3);  // 3 threads

func:worker = NIL(int32:id) {
    print(`Worker &{id}: Phase 1`);
    barrier.wait();  // Wait for all threads
    
    print(`Worker &{id}: Phase 2`);
    barrier.wait();
    
    print(`Worker &{id}: Done`);
}
```

---

### Semaphore

```aria
use std.sync.Semaphore;

semaphore: Semaphore = Semaphore.new(3);  // Max 3 concurrent

func:limited_resource = NIL() {
    semaphore.acquire();
    // Only 3 threads can be here at once
    process_data();
    semaphore.release();
}
```

---

## Best Practices

### ✅ DO: Use Channels for Communication

```aria
// Good - communicate via channels
channel: Channel<Message> = Channel.new();
thread1.send_channel(channel.clone());
thread2.send_channel(channel);
```

### ✅ DO: Avoid Shared Mutable State

```aria
// Good - each thread owns its data
func:worker = NIL(Data:data) {
    // Process independently
}

// Bad - shared mutable state
global_data: Data;  // Dangerous!
```

### ✅ DO: Use RAII for Locks

```aria
{
    lock: MutexGuard = mutex.lock();
    // Critical section
}  // Lock automatically released here
```

### ⚠️ WARNING: Avoid Deadlocks

```aria
// Deadlock scenario
mutex1.lock();
mutex2.lock();  // Thread A

mutex2.lock();
mutex1.lock();  // Thread B - DEADLOCK!

// Solution: Always lock in same order
```

---

## Related

- [async](async.md) - Async keyword
- [async_await](async_await.md) - Async/await pattern
- [threading](threading.md) - Threading
- [atomics](atomics.md) - Atomic operations
- [coroutines](coroutines.md) - Coroutines

---

**Remember**: Concurrency enables **parallelism** but requires **careful synchronization**!
