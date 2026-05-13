# Borrow Diagnostics

This is the cheat sheet for the borrow-checker diagnostic codes. Every
borrow rejection from the v0.25.x compiler maps to one of these.

## `ARIA-023` — conflicting borrow

You tried to introduce a loan that overlaps an existing live loan on the
same host (or a covering path).

Common shapes:

```aria
$$i int32:r = a;
$$m int32:m = a;             // ARIA-023: $$m while $$i is live
```

```aria
$$m int32:r1 = pair.a;
$$m int32:r2 = pair.a;       // ARIA-023: same path twice as $$m
```

```aria
$$m Leaf:rl = box.leaf;
$$m int32:rx = box.leaf.x;   // ARIA-023: rl covers box.leaf.x
```

```aria
$$m int32:a = arr[i];
$$m int32:b = arr[j];        // ARIA-023: arr[*] vs arr[*]
```

**Fix:** narrow the path so the two loans become disjoint, scope-release
the older loan, or convert one of the `$$m`s to `$$i` if read-only access
is enough.

## `ARIA-026` — assignment through borrowed path

You tried to assign directly to a host (or path) that has a live loan:

```aria
$$m int32:r = pair.a;
pair.a = 99;                 // ARIA-026
```

```aria
$$m int32:r = arr[0];
arr[0] = 99;                 // ARIA-026
```

**Fix:** write through the borrow (`r = 99`), or release the loan first by
letting the borrowing binding leave scope.

## `ARIA-022` — borrow of moved value

The host was moved (e.g. into a callee that takes ownership) and is no
longer available to borrow.

```aria
some_take(pair);             // pair moved
$$i Pair:r = pair;           // ARIA-022
```

**Fix:** clone the value before the move, or have the callee take a
borrow instead of ownership.

## `ARIA-027` — escape of local borrow

You tried to return or pass out a borrow whose host is a local — its
lifetime ends with the function:

```aria
func:bad = $$i int32() {
    int32:tmp = 7;
    pass $$i tmp;            // ARIA-027 — tmp dies at function end
};
```

**Fix:** return by value, or have the caller own the storage and pass a
borrow in.

## Reading the messages

The compiler prints the *path* of the conflicting loan, the source location
of both loans, and the kind of each loan (`$$i` / `$$m`). The first line is
the new (rejected) loan; the "previous loan here" line is the existing one.
If the path printed contains `[*]`, you are in dynamic-index territory —
see [arrays.md](arrays.md).

## Where to dig deeper

- The rule set lives in `src/frontend/sema/borrow_checker.cpp`.
- The K formalisation is in `k-semantics/aria.k` (cells `<borrow-vars>`,
  `<borrow-aliases>`, `<imm-borrow-hosts>`, `<mut-borrow-hosts>`).
- The regression suite is in `tests/bugs/bug040..bug204_*.npk`.
