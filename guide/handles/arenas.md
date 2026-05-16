# Arenas

A **handle arena** is a generation-checked bump allocator. You create
one, allocate into it any number of times, and tear the whole thing
down with one call. Every handle minted by an arena is invalidated
when the arena is destroyed.

## Surface

```aria
use "handle.npk".*;

int64:a = raw HandleArena.create();
// ... allocate, deref, free into `a` ...
raw HandleArena.destroy(a);
```

`HandleArena.create()` returns an arena id as `int64` (≥ 0). Store
the id wherever you would store a `gc` binding — it is a plain value.

`HandleArena.destroy(arena)` bumps the generation counter of every
live slot. After this call, all outstanding `Handle<T>` values that
came from `arena` either:

- Fail the static borrow-checker rule
  ([`ARIA-032`](../memory/diagnostics.md#aria-032)), or
- (If the static check is bypassed — e.g. the handle was passed
  through FFI) deref to `0i64` at runtime.

## Capacity

Arenas grow on demand. You do not size them up front. The runtime
maps slot indices to a `calloc`-backed slot table protected by a
mutex; the table grows when slots run out.

Slot indices saturate at `UINT32_MAX` — once a slot has been
re-allocated `2³²` times it is permanently retired (the generation
counter does not wrap). This is the trade-off that makes
generation-checked deref sound: collision is impossible.

## One big arena, or many small ones?

Prefer one arena per **natural lifetime boundary**:

- Per HTTP request, per parse, per game frame, per compilation unit.
- Anywhere you would otherwise reach for a sub-allocator.

Avoid:

- A process-wide arena that lives forever — you lose the bulk-free
  benefit and the slot table grows without bound.
- One arena per allocation — that is just `wild`, with extra steps.

A common pattern is *sibling arenas* in the same scope: one for the
hot data, one for the scratch data, destroyed independently.

```aria
int64:hot     = raw HandleArena.create();
int64:scratch = raw HandleArena.create();
// ... work on both ...
raw HandleArena.destroy(scratch);   // free scratch early
// ... continue using hot ...
raw HandleArena.destroy(hot);
```

`bug262_handle_other_arena_alive_pass.npk` locks in that destroying
one arena does not poison handles bound to a sibling arena.

## Teardown order

The canonical correct teardown is:

```aria
int64:a            = raw HandleArena.create();
Handle<int32>:h    = raw HandleArena.alloc(a, 4i64);
int64:p            = raw HandleArena.deref(h);     // use the handle
raw HandleArena.free(h);                           // free the slot
raw HandleArena.destroy(a);                        // destroy the arena
```

`bug263_handle_destroy_after_use_pass.npk` confirms this order is
accepted; the borrow checker does not false-positive on it.

## What an arena is **not**

- It is not a region in the borrow-checker sense — handles do not
  participate in `$$i` / `$$m` borrow rules. A handle is a value,
  not a borrow.
- It is not a `gc` heap — the GC does not scan arenas, and arenas do
  not move allocations. The pointer you get back from `deref` is
  stable until the slot is freed.
- It is not thread-safe in the sense of "share one handle between
  threads". The runtime serialises arena operations with a mutex,
  but the type system treats handles as thread-local; concurrent use
  of the same handle from multiple threads is out of scope for the
  v0.27.x cycle.

## See also

- [Handles](handles.md) — what `Handle<T>` actually is.
- [Lifetimes](lifetimes.md) — the compile-time outlives rule.
- [FFI](ffi.md) — what to do with the pointer returned from `deref`.
