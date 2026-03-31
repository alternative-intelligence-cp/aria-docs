# User Stack Memory

**Category**: Memory Model → Specialized Allocations  
**Syntax**: `astack()`, `apush()`, `apop()`, `apeek()`  
**Purpose**: Per-scope implicit LIFO allocation for typed scratch values  
**Since**: v0.4.3

---

## What is the User Stack?

The **user stack** is a separate memory region from the call stack, allocated via `mmap` and managed by the compiler. It provides typed LIFO storage for scratch values within a function scope.

---

## Memory Layout

Each user stack consists of:
- A **values array** — `int64` slots for uniform storage
- A **tags array** — `int64` type tags per slot
- A **top pointer** — bump pointer for O(1) push/pop

Total per-slot overhead: **16 bytes** (8 value + 8 tag).

Default capacity: **256 slots = 4,096 bytes** (4 KB).

---

## Allocation Lifecycle

```
astack(N)  →  mmap allocates values + tags arrays
apush(v)   →  store value + tag at top, bump pointer
apop()     →  decrement pointer, read value + tag, validate
             → function exit  →  munmap (auto-cleanup)
```

The compiler inserts cleanup code on all return paths automatically. No `defer` needed.

---

## Relationship to Other Memory Modes

| Mode | Backing | Lifetime | Safety | Use Case |
|------|---------|----------|--------|----------|
| `stack` | Call stack (RSP) | Scope-based | Safe | Local variables |
| `gc` | Heap (GC-managed) | Automatic | Safe | Long-lived objects |
| `wild` | Heap (malloc) | Manual | Unsafe | Full control |
| `wildx` | Executable pages | Manual | Unsafe | JIT code |
| **User stack** | mmap pages | Scope-based | Safe | Typed scratch values |

The user stack is most similar to `stack` allocation — both are scope-based and safe. The difference is that the user stack provides **dynamic LIFO ordering** with **runtime type checking**, while `stack` allocation provides fixed, named variables.

---

## Performance Characteristics

- **Allocation**: Single `mmap` call per `astack()` — microsecond scale
- **Push/Pop**: O(1) — pointer bump + store/load
- **Type check**: One integer comparison per pop/peek
- **Cleanup**: Single `munmap` call on function exit
- **Memory**: Lazy page allocation via mmap — OS only commits pages as touched

---

## See Also

- [Advanced Features → User Stack](../advanced_features/user_stack.md) — Full usage guide
- [Memory Model → Stack](stack.md) — Hardware call stack
- [Memory Model → Allocation](allocation.md) — Memory allocation modes
