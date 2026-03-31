# Hash Tables — ahash

## Overview

Per-scope hash tables mapping string keys to typed values. Heterogeneous (different
value types per key). Auto-cleanup on scope exit. Optional byte-budget capacity.

## Builtins

| Builtin | Signature | Description |
|---------|-----------|-------------|
| `ahash(cap)` | `(int64) → int64` | Create hash table with byte budget. 0 = unbounded. Returns handle. |
| `ahset(h, key, val)` | `(int64, str, T) → int32` | Set key to value. Returns 0 on success, -1 on overflow. |
| `ahget(h, key)` | `(int64, str) → T` | Get value at key. NIL if not found. Type mismatch → failsafe. |
| `ahcount(h)` | `(int64) → int64` | Number of keys in table. |
| `ahsize(h)` | `(int64) → int64` | Current size in bytes of all stored entries. |
| `ahfits(h, val)` | `(int64, T) → int64` | 1 if val fits in remaining capacity, 0 otherwise. Always 1 if unbounded. |
| `ahtype(h, key)` | `(int64, str) → int32` | Type tag of value at key, -1 if key not found. |

## Example

```aria
func:main = int32() {
    int64:ht = ahash(0);                 // unbounded hash table

    int32:rc1 = ahset(ht, "name", "Alice");
    int32:rc2 = ahset(ht, "age", 30);
    int32:rc3 = ahset(ht, "score", 98.6);

    string:name = ahget(ht, "name");
    int32:age = ahget(ht, "age");
    flt64:score = ahget(ht, "score");

    int64:count = ahcount(ht);           // 3

    println(name);
    println(age);
    exit 0;
}

func:failsafe = int32(tbb32:err) { exit 1; }
```

## Fast-Path Mode

When the Z3 solver can prove all values in a hash table are the same type at compile
time, the compiler emits optimized **fast-path** code that skips runtime type checks.
This is automatic — no user action required.

## Capacity

- `ahash(0)` — unbounded (grows as needed)
- `ahash(1024)` — 1KB byte budget; `ahset` returns -1 when full
- `ahfits(h, val)` — check before inserting into bounded tables

## Type Safety

The receiving variable's type determines the expected type on `ahget`:

```aria
int32:val = ahget(ht, "age");     // expects int32 at "age"
string:s = ahget(ht, "age");      // type mismatch → failsafe!
```

Check with `ahtype(h, key)` before getting if unsure.

## Auto-Cleanup

Hash tables are automatically freed when the scope exits.

## Related

- [astack.md](astack.md) — user stack (LIFO)
