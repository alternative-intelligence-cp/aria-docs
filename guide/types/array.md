# Array — `array`

## Overview

Standard contiguous arrays with compile-time or runtime sizing.

## Declaration

```aria
[]int32:numbers = [1, 2, 3, 4, 5];
[]string:names = ["Alice", "Bob", "Carol"];
```

## Access

```aria
int32:first = numbers[0];     // zero-indexed
int32:last = numbers[4];
```

## Slicing

```aria
[]int32:sub = numbers[1..3];  // inclusive range: [2, 3, 4]
```

## Related

- [types/vec.md](vec.md) — fixed-size vector types
- [collections/astack.md](../collections/astack.md) — user stack (LIFO)
- [collections/ahash.md](../collections/ahash.md) — hash tables
