# Handles

A **handle** is an opaque 64-bit token (`Handle<T>`) that names an
allocation inside a generational arena. Handles give you:

- **Region-style cheap allocation** — bump into an arena, free the
  whole arena at once with `HandleArena.destroy`.
- **Stable identity across moves** — handles are pure values; you can
  copy them, store them in structs, hand them to FFI, without changing
  what they refer to.
- **Generation-checked dereference** — once an arena is destroyed (or
  a slot is freed), every outstanding handle to it dereferences to
  `0i64` (NULL). No undefined behaviour, no use-after-free.
- **A compile-time outlives rule** — within a function body, derefing
  a handle whose arena was destroyed is a static error
  ([`ARIA-032`](../memory/diagnostics.md#aria-032)).

## When to reach for a handle vs `gc` / `wild`

| You want…                                            | Use       |
|------------------------------------------------------|-----------|
| Lots of short-lived allocations, all freed together  | `Handle`  |
| One thing collected when nothing references it       | `gc`      |
| Explicit `alloc` / `defer free` with a known shape   | `wild`    |
| Executable memory with W^X lifecycle                 | `wildx`   |

Handles sit between `gc` (zero ceremony, arbitrary lifetime) and
`wild` (full manual control). They are the right answer when you have
a *bulk* of allocations whose lifetime matches a known scope (a
request, a parse, a frame) and you would otherwise be writing your
own free list.

## Chapters

1. [README](README.md) — this page.
2. [Arenas](arenas.md) — `HandleArena.create` / `destroy`, capacity,
   when to use one big arena vs many small ones.
3. [Handles](handles.md) — `Handle<T>`, allocation, deref, free, the
   generation counter, runtime UAF behaviour.
4. [Lifetimes](lifetimes.md) — the borrow-checker rule that catches
   handle-outlives-arena at compile time (`ARIA-032`).
5. [FFI](ffi.md) — handing a handle's underlying pointer across the C
   boundary without losing the safety net.
6. [Diagnostics](diagnostics.md) — every handle-related error /
   warning, smallest reproducer, canonical fix.
7. [FAQ](faq.md) — recurring questions about handles.

## See also

- [`guide/drop/`](../drop/README.md) — the v0.29.5 opt-in RAII
  layer that auto-emits `npk_handle_arena_destroy(a)` for
  `int64:a = HandleArena.create();` bindings at scope end.

## Validation

- Runtime: `tests/runtime/test_handle_v0277.cpp` (CTest
  `runtime_handle_v0277`).
- Typed surface: `bug256`–`bug259` (CTest `bug_tests_v0278`).
- Outlives rule: `bug260`–`bug263` (CTest `bug_tests_v0279`).
