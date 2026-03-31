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

## Related

- [types/vec.md](vec.md) — fixed-size vector types
- [collections/astack.md](../collections/astack.md) — user stack (LIFO)
- [collections/ahash.md](../collections/ahash.md) — hash tables
