# Memory diagnostics

This chapter lists the diagnostic codes the compiler emits for
region/lifetime/leak issues, with the smallest reproducer for each and
the canonical fix. For borrow-checker diagnostics that are not
region-specific (`ARIA-022`, `ARIA-023`, `ARIA-026`), see
[`borrow/diagnostics.md`](../borrow/diagnostics.md).

---

## `ARIA-014` — wild allocation without matching `defer free`

Every `wild` allocation must be paired with a release — the simplest
form is `defer free(...)` in the same scope.

```aria
func:bad = int32() {
    wild int32:buf = 0i32;
    pass 0i32;                 // [ARIA-014] no defer free for buf
};
```

**Fix:** add `defer free(buf);` immediately after the allocation, or
explicitly call the matching release before every exit path.

```aria
func:good = int32() {
    wild int32:buf = 0i32;
    defer free(buf);
    pass 0i32;
};
```

---

## `ARIA-015` — use of released `wild` variable

Using a `wild` value after it has been freed, moved, or otherwise
drop-tracked is rejected.

```aria
wild int32:buf = 0i32;
free(buf);
pass buf;                       // [ARIA-015] buf is released
```

**Fix:** rebuild the value (allocate again) or restructure the code so
the release happens after the last use.

---

## `ARIA-028` — stack escape (return/pass of borrow into a local)

You tried to return, `pass`, or `fail` a borrow whose host is a
binding local to the current function (or to an inner block) — the
host's stack frame is destroyed at the function (or block) boundary,
so the reference would dangle.

```aria
func:bad = $$m int32() {
    stack int32:tmp = 7i32;
    $$m int32:r = tmp;
    pass r;                    // [ARIA-028] tmp's frame ends here
};
```

The same diagnostic also fires when the host is a `gc` *binding*: the
underlying object survives on the heap, but the named binding is
local and the borrow path goes out of scope.

**Fix:** return by value, take ownership in the caller and pass a
borrow in, or accept a borrow parameter and return that.

> **Note (v0.26.6):** earlier compiler versions emitted these errors
> as `ARIA-017`; the docs cited `ARIA-027`. Both are unified under
> `ARIA-028 STACK_ESCAPE` from v0.26.6 on. The full text now
> mentions the host's stack frame and includes an inline fix hint.

---

## Reserved / deferred codes

- **`ARIA-029 GC_REF_FROM_WILD`** — reserved. Will fire when a `gc`
  binding (or field) is initialised from a borrow whose origin is in a
  `wild` allocation. Deferred to v0.27.x because it requires
  per-binding region tracking on `LifetimeContext` (currently only
  `var_depths` is recorded).
- **`ARIA-031 STACK_REF_INTO_GC_FIELD`** — reserved. Will fire when a
  `gc`-region path field is assigned a borrow whose origin is a
  `stack` binding. (Renumbered from the originally-planned `ARIA-030`,
  which is already used for the borrow-across-`await` runtime
  warning.) Deferred to v0.27.x for the same reason.

Until those land, both of the underlying *unsafe* shapes are blocked
upstream by `ARIA-028` (binding-level escape) and `ARIA-014` /
`ARIA-015` (wild lifetime), so there is no runtime regression — only a
diagnostic-quality gap.

---

## See also

- [`borrow/diagnostics.md`](../borrow/diagnostics.md) — non-region
  borrow codes (`ARIA-022`, `ARIA-023`, `ARIA-026`, `ARIA-028`, ...).
- [`stack.md`](stack.md), [`gc.md`](gc.md), [`wild.md`](wild.md) — the
  region chapters with longer-form examples.
