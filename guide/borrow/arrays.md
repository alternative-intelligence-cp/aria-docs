# Array Element Borrows

Arrays are borrowed through their elements. The checker distinguishes
between *literal-index* borrows (path is statically known) and
*dynamic-index* borrows (path collapses to "the whole array").

## Literal index — path is precise

```aria
int32[4]:arr = [10, 20, 30, 40];
$$m int32:first = arr[0];     // path arr[0]
$$m int32:third = arr[2];     // path arr[2] — disjoint, OK
first = 99;                   // arr[0] = 99 after release
```

Two `$$m` loans on different literal indices coexist — they are disjoint
paths. (K test `107`.)

## Same literal index conflicts

```aria
$$m int32:r1 = arr[0];
$$m int32:r2 = arr[0];         // ARIA-023
```

## Dynamic index — collapses to `arr[*]`

When the index is not known at compile time the checker tracks the loan as
*"the whole array"*:

```aria
int32:i = 1;
$$i int32:elem = arr[i];       // path arr[*]
exit elem;                     // 20
```

A single dynamic immutable borrow is fine. (K test `110`.)
A single dynamic mutable borrow with writeback is fine. (K test `111`.)

## Two dynamic borrows on the same array conflict

Because both collapse to `arr[*]`, the checker cannot prove disjointness
even if the indices happen to differ at runtime:

```aria
int32:i = 0;
int32:j = 1;
$$m int32:a = arr[i];
$$m int32:b = arr[j];          // ARIA-023: arr[*] vs arr[*]
```

(K test `112`.)

## Dynamic borrow + literal borrow on the same array

The literal index `[k]` is not provably disjoint from `[*]`, so this is
also rejected:

```aria
$$m int32:dyn = arr[i];
$$m int32:lit = arr[0];        // ARIA-023: arr[*] vs arr[0]
```

A dynamic borrow on one array combined with any borrow on a *different*
array is fine — different hosts cannot alias. (K test `113`.)

## Assigning to a borrowed slot

```aria
$$m int32:r = arr[0];
arr[0] = 99;                  // ARIA-026: assignment through borrowed slot
```

(K test `109`.)

## Bounds

Out-of-bounds access is *not* checked by the borrow checker — that is the
runtime / KLEE story. See [`verification/klee.md`](../verification/klee.md).
