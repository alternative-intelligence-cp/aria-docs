# Handles FAQ

## Why not just use a pointer?

A `wild` pointer is sound only as long as you remember to `free`
exactly once. A `Handle<T>` is sound even if you forget — the
generation check on deref turns "use after free" into a `0i64`
return, not undefined behaviour. You also get cheap bulk-free via
`HandleArena.destroy`, which a raw pointer cannot do.

If your allocation has the shape "one of these, kept around forever
or until the program exits", a `wild` pointer is fine. Once you have
*many* of them, all tied to a known scope, handles win.

## What's the overhead vs `wild`?

Per `alloc`: one mutex lock, one slot-table lookup (constant for the
allocated capacity), and the underlying `calloc`. Per `deref`: one
mutex lock and one generation comparison. The cost is small but
non-zero compared to a raw pointer dereference.

In return you get the generation check, the bulk-free, and the
static [`ARIA-032`](../memory/diagnostics.md#aria-032) rule. For
anything that handles untrusted lifetimes (request handlers, plugin
callbacks, anything that can be released out of order), the trade
is worth it.

## Can I store a `Handle<T>` in a `gc` value?

Yes. `Handle<T>` lowers to `int64`; it's a pure value and the GC
ignores it. Storing it in a `gc` struct is fine. The handle does not
keep the arena alive — destroying the arena while a `gc`-stored
handle still references it is allowed (and will result in
runtime-NULL deref, since the static rule cannot trace across
storage).

## Can I share a handle between threads?

The runtime serialises arena operations under a mutex, so concurrent
`alloc` / `free` / `deref` calls do not corrupt internal state. But
the type system treats handles as thread-local: there is no
happens-before guarantee on the *contents* of a handle's slot
between threads, and the `ARIA-032` static rule does not consider
threads. For the v0.27.x cycle, treat handles as thread-local.

A future cycle may add atomic generation counters and a `wild`-style
"share carefully" path; for now, hand the *data*, not the handle,
across threads.

## What's the difference between `HandleArena.free(h)` and `HandleArena.destroy(a)`?

- `free(h)` releases **one slot** and bumps its generation. Any
  outstanding copies of that specific handle become stale; the
  arena and other handles in it are unaffected.
- `destroy(a)` tears down **the whole arena**: every slot's
  generation is bumped, the slot table is released. All outstanding
  handles minted from `a` become stale at once.

In bulk-free workflows (per-request arenas, per-frame arenas), you
typically skip `free` entirely and just call `destroy` at the end of
the scope.

## What happens if I `deref` a NULL handle?

Returns `0i64`. The runtime treats generation `0` as NULL and
short-circuits before the slot lookup.

## What about saturation?

A single slot reused `2³²` times retires permanently (it is never
allocated from again). The arena keeps growing its slot table; this
is one of the few resource costs that does not bound back to the
arena lifetime. If you have a workload that turns over a single
arena trillions of times, prefer destroying and recreating the
arena rather than reusing it forever.

## How is this different from the `wildx` W^X allocator?

`wildx` is about **executable** memory with a write→execute lifecycle
(`alloc` → `seal` → `free`). It has nothing to do with handles or
generation checking. The two systems can coexist; they target
different problems.

## Why is `Handle<T>` not parametric the way `optional<T>` is?

The `T` parameter is currently a *type-checker tag* — it prevents
silent cross-type assignment between `Handle<int32>` and
`Handle<int64>` (`bug257`) but does not generate per-`T` code. The
runtime is `int64`-only; alloc takes a `size` in bytes. Richer
generic dispatch (e.g. `HandleArena.alloc<T>(a)` inferring `size_of(T)`)
is on the wishlist for a future cycle.

## I want lifetimes that cross function calls.

That is the deferred extension noted in [lifetimes](lifetimes.md).
For now, either:

- Keep the entire arena lifetime inside one function (recommended).
- Rely on the runtime NULL-on-stale check for any cross-function
  misuse — sound, just not caught at compile time.

## See also

- [README](README.md), [Arenas](arenas.md), [Handles](handles.md),
  [Lifetimes](lifetimes.md), [FFI](ffi.md), [Diagnostics](diagnostics.md).
