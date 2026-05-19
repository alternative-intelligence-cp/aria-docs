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

**Alternative (v0.29.3+):** import `drop.npk` and use a struct
binding to opt the binding into RAII auto-free, which silences
`ARIA-014` for that shape. See the [`guide/drop/`](../drop/README.md)
cookbook.

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

`ARIA-029` and `ARIA-031` were reserved here through v0.26.x and
landed in v0.27.2 / v0.27.3 respectively — see the dedicated sections
above. `ARIA-032` (handle outlives arena) landed in v0.27.9.

No region codes are currently in the "reserved" state.

---

## `ARIA-029` — GC reference from `wild` (v0.27.2)

A `gc` binding (or `gc` field) is being initialised from a borrow
whose origin is a `wild` allocation. The type checker emits a hint
suggesting the pin operator `#x`, since the underlying shape would
otherwise leak a wild-region reference into a GC root.

```aria
wild int32:w = 5i32;
gc int32@:p  = @w;            // [ARIA-029] consider `gc int32@:p = #w;`
```

**Fix:** pin the wild value (`#w`), or restructure so the GC binding
owns its value rather than referencing through `wild` storage.

The borrow-checker rejection in `checkVarDecl` is defense-in-depth;
the type checker rejects this shape first in surface code.

---

## `ARIA-031` — stack reference into a GC field (v0.27.3)

Assigning the address of a `stack` binding into a field reached
through a `gc` path would let the GC observe a soon-to-be-dangling
stack pointer.

```aria
gc Node:n        = Node{ next: @local_stack_node };   // [ARIA-031]
```

**Fix:** allocate the referenced node in the `gc` region, or pin it
(`#local_stack_node`) so the borrow checker enforces address
stability — and accept that the stack lifetime still bounds the
reference's validity.

---

## `ARIA-032` — handle outlives its arena (v0.27.9)

Within a function body, you destroyed an arena and then derefed (or
freed) a `Handle<T>` bound to that arena.

```aria
int64:a            = raw HandleArena.create();
Handle<int32>:h    = raw HandleArena.alloc(a, 4i64);
raw HandleArena.destroy(a);
int64:p = raw HandleArena.deref(h);     // [ARIA-032]
```

**Fix:** move the deref/free before the `HandleArena.destroy(a)`
call, or drop it entirely (a bulk-destroy obviates per-slot frees).

The rule has grown across v0.28.x:

- **v0.27.9** — intra-function only: destroy + later deref/free in
  the same function body.
- **v0.28.3** — cross-function Phase 1: a callee that calls
  `HandleArena.destroy(p)` on a parameter `p` is auto-discovered;
  the call site fires `ARIA-032` if the matching argument is
  re-used after the call.
- **v0.28.4 / v0.28.4.1** — cross-function Phase 2: returning a
  handle bound to a local arena (bare or stashed in a struct
  field) fires `ARIA-032` at the `pass` / `return` site.
- **v0.28.5** — FFI passthrough: passing a `Handle<T>` directly
  to an `extern` function emits a **warning** with the
  `@cast<int64>(h)` fix suggestion. The cast is also the way to
  silence the warning intentionally; once you cross the C
  boundary the static rule cannot follow you and the runtime
  generation check is your only safety net.

For cases the static rule still does not catch (handles stashed
in GC objects, handles round-tripped through opaque C state,
indirect aliasing through arrays), the runtime generation check
still applies: `deref` returns `0i64`. See
[`handles/lifetimes.md`](../handles/lifetimes.md) for the full
rule and [`handles/diagnostics.md`](../handles/diagnostics.md)
for the canonical examples.

---

## See also

- [`borrow/diagnostics.md`](../borrow/diagnostics.md) — non-region
  borrow codes (`ARIA-022`, `ARIA-023`, `ARIA-026`, `ARIA-028`, ...).
- [`stack.md`](stack.md), [`gc.md`](gc.md), [`wild.md`](wild.md) — the
  region chapters with longer-form examples.
- [`../handles/README.md`](../handles/README.md) — the handle
  cookbook (`ARIA-032`, runtime UAF behaviour).
