# Threading

**Category**: Advanced Features → Concurrency  
**Purpose**: Parallel execution using OS threads

---

## Overview

Threads enable **true parallelism** by running code on multiple CPU cores simultaneously.

---

## Basic Thread

```aria
use std.thread.Thread;

func:worker = NIL() {
    print("Hello from thread!");
}

func:main = NIL() {
    thread: Thread = Thread.spawn(worker);
    thread.join();  // Wait for completion
}
```

---

## Thread with Arguments

```aria
use std.thread.Thread;

func:worker = NIL(int32:id, string:message) {
    print(`Thread &{id}: &{message}`);
}

func:main = NIL() {
    thread: Thread = Thread.spawn(|| {
        worker(1, "Hello");
    });
    thread.join();
}
```

---

## Multiple Threads

```aria
use std.thread.Thread;

func:main = NIL() {
    threads: []Thread = [];
    
    till(3, 1) {
        thread: Thread = Thread.spawn(|| {
            print("Worker $($)");
        });
        threads.push(thread);
    }
    
    // Wait for all threads
    till(threads.length - 1, 1) {
        threads[$].join();
    }
}
```

---

## Thread Return Values

```aria
use std.thread.Thread;

func:compute = int32(int32:n) {
    pass(n * n);
}

func:main = NIL() {
    thread: Thread<i32> = Thread.spawn(|| {
        pass(compute(10));
    });
    
    Result: i32 = thread.join();
    print(`Result: &{result}`);  // Result: 100
}
```

---

## Thread Synchronization

### Mutex

```aria
use std.sync.Mutex;
use std.thread.Thread;

counter: Mutex<i32> = Mutex.new(0);

func:worker = NIL() {
    till(999, 1) {
        lock = counter.lock();
        *lock += 1;
        // Mutex automatically unlocked when lock goes out of scope
    }
}

func:main = NIL() {
    threads: []Thread = [];
    till(3, 1) {
        threads.push(Thread.spawn(worker));
    }
    
    till(threads.length - 1, 1) {
        threads[$].join();
    }
    
    print("Counter: &{counter.lock()}");  // Counter: 4000
}
```

---

### Channels

```aria
use std.sync.Channel;
use std.thread.Thread;

func:producer = NIL(Sender<i32>:tx) {
    till(9, 1) {
        tx.send($);
    }
}

func:consumer = NIL(Receiver<i32>:rx) {
    while value = rx.recv() {
        print(`Received: &{value}`);
    }
}

func:main = NIL() {
    channel: Channel<i32> = Channel.new();
    
    t1: Thread = Thread.spawn(|| producer(channel.sender()));
    t2: Thread = Thread.spawn(|| consumer(channel.receiver()));
    
    t1.join();
    channel.close();
    t2.join();
}
```

---

## Common Patterns

### Worker Pool

```aria
use std.sync.Channel;
use std.thread.Thread;

struct WorkerPool<T, R> {
    workers: []Thread,
    task_queue: Channel<T>,
    result_queue: Channel<R>,
}

impl<T, R> WorkerPool<T, R> {
    pub func:new = R)(int32:num_workers, fn(T:work_fn)-> WorkerPool {
        task_queue: Channel<T> = Channel.new();
        result_queue: Channel<R> = Channel.new();
        workers: []Thread = [];
        
        till(num_workers - 1, 1) {
            rx = task_queue.receiver();
            tx = result_queue.sender();
            
            worker: Thread = Thread.spawn(|| {
                while task = rx.recv() {
                    Result: R = work_fn(task);
                    tx.send(result);
                }
            });
            workers.push(worker);
        }
        
        return WorkerPool {
            workers: workers,
            task_queue: task_queue,
            result_queue: result_queue,
        };
    }
    
    pub func:submit = NIL(T:task) {
        self.task_queue.send(task);
    }
    
    pub func:get_result = ?R() {
        pass(self.result_queue.recv());
    }
    
    pub func:shutdown = NIL() {
        self.task_queue.close();
        till(self.workers.length - 1, 1) {
            self.workers[$].join();
        }
    }
}

// Usage
func:process_data = int32(int32:data) {
    pass(data * 2);
}

func:main = NIL() {
    pool: WorkerPool<i32, i32> = WorkerPool.new(4, process_data);
    
    // Submit tasks
    till(99, 1) {
        pool.submit($);
    }
    
    // Collect results
    till(99, 1) {
        Result: i32 = pool.get_result()?;
        print(result);
    }
    
    pool.shutdown();
}
```

---

### Producer-Consumer

```aria
use std.sync.Channel;
use std.thread.Thread;

func:producer = NIL(Sender<string>:tx) {
    till(9, 1) {
        message: string = "Message $($)";
        tx.send(message);
        Thread.sleep(100);  // Simulate work
    }
}

func:consumer = NIL(int32:id, Receiver<string>:rx) {
    while message = rx.recv() {
        print(`Consumer &{id} got: &{message}`);
        Thread.sleep(50);  // Simulate processing
    }
}

func:main = NIL() {
    channel: Channel<string> = Channel.new();
    
    // One producer
    p: Thread = Thread.spawn(|| producer(channel.sender()));
    
    // Multiple consumers
    consumers: []Thread = [];
    till(2, 1) {
        rx = channel.receiver();
        consumer: Thread = Thread.spawn(|| consumer($, rx));
        consumers.push(consumer);
    }
    
    p.join();
    channel.close();
    
    till(consumers.length - 1, 1) {
        consumers[$].join();
    }
}
```

---

### Parallel Map

```aria
use std.thread.Thread;

func:parallel_map = R,([]T:items, fn(T:f)num_threads: i32) -> []R {
    results: []R = [];
    chunk_size: i32 = items.len() / num_threads;
    threads: []Thread<[]R> = [];
    
    till(num_threads - 1, 1) {
        i = $;
        start: i32 = i * chunk_size;
        end: i32 = if i == num_threads - 1 {
            items.len()
        } else {
            (i + 1) * chunk_size
        };
        
        chunk: []T = items[start..end];
        
        thread: Thread<[]R> = Thread.spawn(|| {
            chunk_results: []R = [];
            till(chunk.length - 1, 1) {
                chunk_results.push(f(chunk[$]));
            }
            pass(chunk_results);
        });
        threads.push(thread);
    }
    
    till(threads.length - 1, 1) {
        chunk_results: []R = threads[$].join();
        results.extend(chunk_results);
    }
    
    pass(results);
}

// Usage
func:square = int32(int32:x) {
    pass(x * x);
}

func:main = NIL() {
    numbers: []i32 = [1, 2, 3, 4, 5, 6, 7, 8];
    squared: []i32 = parallel_map(numbers, square, 4);
    print(squared);
}
```

---

## Thread-Local Storage

```aria
use std.thread.ThreadLocal;

thread_id: ThreadLocal<i32> = ThreadLocal.new();

func:worker = NIL(int32:id) {
    thread_id.set(id);
    
    // Each thread has its own copy
    my_id: i32 = thread_id.get();
    print(`Thread &{my_id}`);
}
```

---

## Best Practices

### ✅ DO: Use for CPU-Bound Work

```aria
func:parallel_compute = flt64([]f64:data) {
    threads: []Thread<f64> = [];
    chunks = data.chunks(1000);
    
    till(chunks.length - 1, 1) {
        thread: Thread<f64> = Thread.spawn(|| {
            pass(expensive_computation(chunks[$]));
        });
        threads.push(thread);
    }
    
    sum: f64 = 0.0;
    till(threads.length - 1, 1) {
        sum += threads[$].join();
    }
    pass(sum);
}
```

### ✅ DO: Use Synchronization Primitives

```aria
use std.sync.Mutex;

shared_data: Mutex<HashMap<string, i32>> = Mutex.new(HashMap.new());

func:update_shared = NIL() {
    lock = shared_data.lock();
    lock.insert("key", 42);
    // Automatically unlocked
}
```

### ⚠️ WARNING: Avoid Data Races

```aria
// ❌ Bad - data race!
counter: i32 = 0;

func:increment = NIL() {
    counter += 1;  // Not thread-safe!
}

// ✅ Good - use atomics or mutex
use std.sync.Atomic;
counter: Atomic<i32> = Atomic.new(0);

func:increment = NIL() {
    counter.fetch_add(1);  // Thread-safe
}
```

### ❌ DON'T: Create Too Many Threads

```aria
// ❌ Bad - too many threads
till(9999, 1) {
    Thread.spawn(worker);
}

// ✅ Good - use thread pool
pool: WorkerPool = WorkerPool.new(8);  // Match CPU cores
till(9999, 1) {
    pool.submit(work);
}
```

---

## Related

- [atomics](atomics.md) - Atomic operations
- [async](async.md) - Async/await
- [concurrency](concurrency.md) - Concurrency overview

---

**Remember**: Threads provide **true parallelism** but require **synchronization** to avoid races!
