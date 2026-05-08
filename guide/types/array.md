# Array — `array`

## Overview

Standard contiguous arrays with compile-time sizing.

## Declaration

```aria
int32[5]:numbers;
numbers[0] = 10i32;
numbers[1] = 20i32;
numbers[2] = 30i32;
numbers[3] = 40i32;
numbers[4] = 50i32;
```

## Access

```aria
int32:first = numbers[0];     // zero-indexed
int32:last = numbers[4];
if (numbers[2] != 30i32) {
    // handle error
}
```

## Slicing

```aria
int64[5]:nums;
int64[]:slice = nums[0..4];    // inclusive range
int64[]:slice2 = nums[0...2];  // exclusive range
```

## Multi-Dimensional Arrays (v0.19.0)

Nitpick supports 2-D (and higher) arrays with nested bracket syntax:

```aria
int32[2][2]:sq = [[1, 2], [3, 4]];
int32:top_right = sq[0][1];    // 2

int32[3][4]:tbl;
int32:r = 1;
int32:c = 3;
tbl[r][c] = 42;
int32:val = tbl[r][c];         // 42
```

The first bracket is the outer (row) dimension; the second is the inner
(column) dimension. Access uses `arr[row][col]` for 2-D arrays.

## Dynamic-Index Borrows (v0.19.0)

Array elements can be borrowed with `$$i` (immutable) or `$$m` (mutable) using
a variable as the index. The borrow checker treats any variable-index borrow as
targeting the entire array (path `[*]`) since the slot is not known at
compile time:

```aria
int32[3]:arr = [10, 20, 30];
int32:i = 1;

$$i int32:elem = arr[i];   // immutable borrow — read only
// arr is locked for mutation while elem is live

$$m int32:slot = arr[i];   // mutable borrow — write-back on release
slot = 99;
// arr[1] is now 99 after slot goes out of scope
```

**Conflict rules:**

- Two `$$m` borrows to the same array via any variable index are rejected
  (both map to `[*]` — conservatively non-disjoint).
- A literal-index borrow and a variable-index borrow on the same array are
  rejected (ARIA-023 — `[k]` and `[*]` cannot be proven disjoint).
- Borrows on *different* arrays (even within the same struct) are fine.

```aria
// COMPILE ERROR — both borrow the same array's [*] path
$$m int32:a = arr[i];
$$m int32:b = arr[j];   // ARIA-023

// COMPILE ERROR — literal arr[0] conflicts with dynamic arr[i]
$$m int32:lit = arr[0];
$$m int32:dyn = arr[i]; // ARIA-023
```

## Arrays-in-Structs Borrow Interaction (v0.19.0)

When a struct contains an array field, split borrows work across disjoint
paths — a scalar field and an array element are distinct paths:

```aria
struct:Pair = {
    int32:a;
    int32[4]:buf;
};

Pair:p;
p.a = 10;
p.buf[1] = 2;

$$m int32:ra = p.a;       // borrows path p.a
$$m int32:rb = p.buf[1];  // borrows path p.buf[1] — disjoint from p.a
ra = 3;
rb = 5;
// p.a == 3, p.buf[1] == 5 after both borrows release
```

## Related

- [types/vec.md](vec.md) — fixed-size vector types
- [collections/astack.md](../collections/astack.md) — user stack (LIFO)
- [collections/ahash.md](../collections/ahash.md) — hash tables
- [memory_model/borrow.md](../memory_model/borrow.md) — borrow semantics
