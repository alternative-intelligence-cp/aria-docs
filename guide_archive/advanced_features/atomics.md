# Atomic Operations

**Category**: Advanced Features → Concurrency  
**Purpose**: Thread-safe operations without locks

---

## Overview

Atomics provide **lock-free** thread-safe operations on shared data.

---

## Basic Atomic

```aria
use std.sync.Atomic;

counter: Atomic<i32> = Atomic.new(0);

// Thread-safe increment
counter.fetch_add(1);
```

---

## Atomic Types

```aria
use std.sync.Atomic;

atomic_int: Atomic<i32> = Atomic.new(0);
atomic_uint: Atomic<u32> = Atomic.new(0);
atomic_bool: Atomic<bool> = Atomic.new(false);
atomic_ptr: Atomic<*void> = Atomic.new(NULL);
```

---

## Atomic Operations

### Load and Store

```aria
atomic: Atomic<i32> = Atomic.new(42);

// Load value
value: i32 = atomic.load();

// Store value
atomic.store(100);
```

---

### Fetch and Modify

```aria
atomic: Atomic<i32> = Atomic.new(10);

old: i32 = atomic.fetch_add(5);   // old = 10, atomic = 15
old: i32 = atomic.fetch_sub(3);   // old = 15, atomic = 12
old: i32 = atomic.fetch_and(0xF); // Bitwise AND
old: i32 = atomic.fetch_or(0x10); // Bitwise OR
old: i32 = atomic.fetch_xor(0x5); // Bitwise XOR
```

---

### Compare and Swap (CAS)

```aria
atomic: Atomic<i32> = Atomic.new(10);

// Compare and swap
success: bool = atomic.compare_and_swap(
    10,   // Expected value
    20    // New value
);

if success {
    print("Swapped!");  // atomic is now 20
} else {
    print("Failed - value was not 10");
}
```

---

### Swap

```aria
atomic: Atomic<i32> = Atomic.new(42);

// Atomically swap and return old value
old: i32 = atomic.swap(100);  // old = 42, atomic = 100
```

---

## Memory Ordering

```aria
// Relaxed - no ordering guarantees
atomic.store_relaxed(value);
value = atomic.load_relaxed();

// Acquire/Release - synchronization
atomic.store_release(value);
value = atomic.load_acquire();

// Sequential - strongest ordering
atomic.store_seq_cst(value);
value = atomic.load_seq_cst();
```

---

## Common Patterns

### Thread-Safe Counter

```aria
use std.sync.Atomic;

struct Counter {
    value: Atomic<i32>,
}

impl Counter {
    pub func:new = Counter() {
        return Counter {
            value: Atomic.new(0),
        };
    }
    
    pub func:increment = NIL() {
        self.value.fetch_add(1);
    }
    
    pub func:get = int32() {
        pass(self.value.load());
    }
}

// Use from multiple threads
counter: Counter = Counter.new();

func:worker = NIL() {
    till(999, 1) {
        counter.increment();  // Thread-safe!
    }
}
```

---

### Atomic Flag

```aria
use std.sync.Atomic;

running: Atomic<bool> = Atomic.new(true);

func:worker = NIL() {
    while running.load() {
        // Do work
    }
}

func:stop = NIL() {
    running.store(false);  // Signal all workers to stop
}
```

---

### Lock-Free Stack

```aria
use std.sync.Atomic;

struct Node<T> {
    value: T,
    next: *Node<T>,
}

struct Stack<T> {
    head: Atomic<*Node<T>>,
}

impl<T> Stack<T> {
    pub func:push = NIL(T:value) {
        node: *Node<T> = alloc(Node {
            value: value,
            next: NULL,
        });
        
        loop {
            old_head: *Node<T> = self.head.load();
            node.next = old_head;
            
            if self.head.compare_and_swap(old_head, node) {
                break;  // Success
            }
            // Retry if another thread modified head
        }
    }
    
    pub func:pop = ?T() {
        loop {
            old_head: *Node<T> = self.head.load();
            if old_head == NULL {
                pass(None);
            }
            
            new_head: *Node<T> = old_head.next;
            
            if self.head.compare_and_swap(old_head, new_head) {
                value: T = old_head.value;
                free(old_head);
                pass(Some(value));
            }
            // Retry if another thread modified head
        }
    }
}
```

---

### Spin Lock

```aria
use std.sync.Atomic;

struct SpinLock {
    locked: Atomic<bool>,
}

impl SpinLock {
    pub func:new = SpinLock() {
        return SpinLock {
            locked: Atomic.new(false),
        };
    }
    
    pub func:lock = NIL() {
        while !self.locked.compare_and_swap(false, true) {
            // Spin until we acquire the lock
            yield_cpu();
        }
    }
    
    pub func:unlock = NIL() {
        self.locked.store(false);
    }
}
```

---

## Best Practices

### ✅ DO: Use for Simple Counters

```aria
hits: Atomic<i32> = Atomic.new(0);

func:record_hit = NIL() {
    hits.fetch_add(1);  // ✅ Thread-safe, lock-free
}
```

### ✅ DO: Use for Flags

```aria
shutdown: Atomic<bool> = Atomic.new(false);

func:check_shutdown = bool() {
    pass(shutdown.load());
}
```

### ⚠️ WARNING: Understand Memory Ordering

```aria
// Relaxed - fast but no ordering
atomic.store_relaxed(value);  // ⚠️ May be reordered

// SeqCst - slow but safe
atomic.store_seq_cst(value);  // ✅ Ordered
```

### ❌ DON'T: Use for Complex Data Structures

```aria
// ❌ Bad - use Mutex instead
atomic_map: Atomic<HashMap>;  // Complex type

// ✅ Good - use proper synchronization
mutex_map: Mutex<HashMap>;
```

---

## Performance Characteristics

```aria
// Atomic operations are lock-free but not always wait-free

// Single operation: very fast
counter.fetch_add(1);  // Nanoseconds

// CAS loop: depends on contention
while !atomic.compare_and_swap(old, new) {
    // May retry many times under high contention
}
```

---

## Related

- [threading](threading.md) - Threading
- [concurrency](concurrency.md) - Concurrency overview

---

**Remember**: Atomics provide **lock-free** synchronization for **simple** data types!
