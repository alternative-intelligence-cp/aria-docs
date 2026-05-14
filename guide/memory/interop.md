# GC ↔ wild interop

`wild` / `wildx` and `gc` are deliberately separate worlds. The borrow
checker ignores `wild`; the GC ignores `wild`. This chapter explains
what that means in practice, what the runtime guarantees, and the one
escape hatch when you need a `wild` slot to root a `gc` reference.

---

## The two invariants

**Invariant 1 — the GC marker never follows a `wild` pointer.**
The mark phase walks the shadow stack and, for each rooted slot, asks
the runtime whether the pointer it sees is inside the GC heap
(`is_heap_pointer`, [`src/runtime/gc/gc.cpp:1024`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/src/runtime/gc/gc.cpp#L1024)).
Pointers outside the nursery and old-generation ranges are skipped
unconditionally. A `wild` allocation can therefore contain bytes that
*look* like a GC pointer; the marker will not be fooled.

**Invariant 2 — a `wild` slot does not root anything by itself.**
`wild` allocations live on the libc heap (`npk_alloc` ⇒ `malloc`).
They are never registered as GC roots and never appear on the shadow
stack. Storing a `gc` reference into a `wild` buffer and dropping the
named handle is a use-after-free waiting to happen — the GC has no way
to know the reference exists and will reclaim it on the next cycle.

These two invariants together mean you can mix `gc Holder:h` and
`wild ?->:scratch` in the same scope without one corrupting the
other.

## Verifying the partition

The runtime exposes
[`npk_gc_is_heap_pointer_i32`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/include/runtime/gc.h)
(declared in [`include/runtime/gc.h`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/include/runtime/gc.h))
as the canonical predicate. It returns `1` for any address inside the
GC nursery or old-gen and `0` for everything else, including stack
addresses, static data, and `wild` allocations.

```aria
extern func:npk_alloc = wild ?*(int64:size);
extern func:npk_free = void(wild ?*:ptr);
extern func:npk_gc_is_heap_pointer_i32 = int32(wild ?*:ptr);

struct:Holder = { int32:value; };

func:main = int32() {
    gc Holder:h = Holder{value: 7i32};
    wild Holder->:hp = #h;
    int32:gc_in = npk_gc_is_heap_pointer_i32(hp);   // 1

    wild ?->:wp = npk_alloc(64i64);
    int32:wild_in = npk_gc_is_heap_pointer_i32(wp); // 0
    npk_free(wp);

    if (gc_in == 1i32) {
        if (wild_in == 0i32) { exit 0; }
    }
    exit 1;
};
```

This is exactly the assertion in `bug230_gc_wild_heap_partition.npk`.

> **ABI note.** The legacy `npk_gc_is_heap_pointer` returns C `_Bool`,
> whose upper EAX bits are undefined under the System V x86-64 ABI.
> Reading that value through a Nitpick `int32` extern produces
> garbage. Always use the `_i32` variant from Nitpick — it is exactly
> the same function with a fully-extended return value.

## Mixing safely

`gc` and `wild` may freely coexist in the same scope. The wild
allocator runs through libc `malloc`, which never returns addresses
inside the GC's mmap'd nursery, and the GC never touches addresses
outside its own ranges. A churn loop that allocates and frees `wild`
buffers around `npk_gc_safepoint()` calls does not perturb GC
reachability:

```aria
extern func:npk_gc_safepoint = void();
extern func:npk_alloc = wild ?*(int64:size);
extern func:npk_free = void(wild ?*:ptr);

func:main = int32() {
    gc Holder:h = Holder{value: 555i32};
    loop(0i32, 5000i32, 1i32) {
        wild ?->:wp = npk_alloc(128i64);
        npk_free(wp);
        npk_gc_safepoint();   // h must still resolve to 555 afterwards
    }
    if (h.value == 555i32) { exit 0; }
    exit 1;
};
```

`bug231_gc_wild_coexist_under_churn.npk` is precisely this pattern.

## When you actually need a `wild` slot to root a `gc` reference

There is a narrow, deliberate escape hatch:
[`npk_shadow_stack_add_root`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/include/runtime/gc.h).
You hand it the address of a slot that contains a `gc` pointer, and
the marker visits that slot on every collection cycle until you
unregister it. The slot itself can live anywhere — including inside a
`wild` buffer.

This is rare. Use it only when:

- You are building a long-lived data structure in `wild` memory
  (e.g. a JIT scratchpad) that has to hold a reference back into `gc`
  state.
- That reference is the only thing keeping the GC object alive.

The alternative — storing the same reference in a `gc` field — is
always preferred when feasible. Explicit registration is an unsafe
operation: forgetting to unregister leaks the slot reference into
every future collection, and unregistering early is a use-after-free.

## What the GC does not protect

- **`wild` ↔ `wild` aliasing.** The borrow checker is off; if two
  `wild` pointers alias, that is on you.
- **`wild` writes to live `gc` objects.** If you cast a `gc` reference
  to `wild ?->` and write to it through the `wild` view, you are
  bypassing the write barrier. The GC will not see the new edge.
  Don't do this.
- **`wildx` execution of GC bytes.** `wildx` is for code; do not
  point a `wildx` mapping at a `gc` allocation.

For the corresponding diagnostics work, see the upcoming
`diagnostics.md` chapter (v0.26.6).

## Validation snapshot (v0.26.5)

- `bug230_gc_wild_heap_partition.npk` — exits 0 when the runtime
  partitions `gc` and `wild` addresses correctly.
- `bug231_gc_wild_coexist_under_churn.npk` — exits 0 when a `gc`
  binding survives 5 000 wild alloc/free cycles interleaved with
  `npk_gc_safepoint()`.
- New runtime export `npk_gc_is_heap_pointer_i32` — ABI-safe (full
  `int32_t` return) variant of the existing `_Bool`-returning
  predicate. Use this from Nitpick code.
- CTest entry `bug_tests_v0265` (label
  `memory;gc;interop;MEM-013;v0.26.5`).
