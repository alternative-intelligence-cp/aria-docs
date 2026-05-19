# Handle lifetimes (`ARIA-032`)

A handle is only valid as long as the arena that minted it lives.
Within a single function body, the borrow checker enforces this
statically: derefing or freeing a handle whose arena was destroyed
earlier in the same function is a compile error tagged
[`ARIA-032`](../memory/diagnostics.md#aria-032).

## The rule

```aria
use "handle.npk".*;

func:bad = int32() {
    int64:a            = raw HandleArena.create();
    Handle<int32>:h    = raw HandleArena.alloc(a, 4i64);
    raw HandleArena.destroy(a);              // arena gone
    int64:p = raw HandleArena.deref(h);      // [ARIA-032]
    exit 0;
};
```

```
error[ARIA-032]: Handle 'h' outlives its arena 'a'. The arena was
destroyed earlier in this function; the handle is no longer valid.
See ARIA-032 and guide/memory/handles.md.
```

`free` of a stale handle is also rejected:

```aria
raw HandleArena.destroy(a);
raw HandleArena.free(h);                     // [ARIA-032]
```

## What the rule catches

The borrow checker records two things per function:

- For every `Handle<T>:h = ... HandleArena.alloc(arena, ...)`, a
  binding from `h` → the arena variable's name.
- For every `HandleArena.destroy(a)`, the name `a` joins a
  "destroyed in this function" set.

`HandleArena.deref(h)` and `HandleArena.free(h)` look up `h`'s bound
arena; if it is in the destroyed set, the rule fires.

The rule does **not** false-positive on correct teardown:

```aria
int64:a         = raw HandleArena.create();
Handle<int32>:h = raw HandleArena.alloc(a, 4i64);
int64:p         = raw HandleArena.deref(h);    // OK — a still alive
raw HandleArena.free(h);                       // OK — a still alive
raw HandleArena.destroy(a);                    // last
exit 0;
```

`bug263_handle_destroy_after_use_pass.npk` is the regression test.

Sibling arenas are independent:

```aria
int64:a            = raw HandleArena.create();
int64:b            = raw HandleArena.create();
Handle<int32>:hb   = raw HandleArena.alloc(b, 4i64);
raw HandleArena.destroy(a);                    // destroy a
int64:p = raw HandleArena.deref(hb);           // OK — hb is bound to b, not a
```

`bug262_handle_other_arena_alive_pass.npk` covers this.

## What the rule does **not** catch (intentional)

The v0.28.x slices extended the rule across function boundaries
for the common shapes:

- **Callee destroys a parameter** — v0.28.3 auto-discovers any
  callee whose body calls `HandleArena.destroy(p)` on a parameter
  `p`, and fires `ARIA-032` at the call site when the matching
  argument is re-used after the call (regressions: `bug267`
  compile-fail, `bug268` positive sibling-arena).
- **Returning a handle bound to a local arena** — v0.28.4 catches
  bare returns (`pass h` where `h` was allocated in an arena
  created in this function); v0.28.4.1 extends this to struct
  fields (`pass HBox{h:h}`). Both fire `ARIA-032` at the
  `pass` / `return` site.
- **`Handle<T>` directly into `extern`** — v0.28.5 emits a
  **warning** with the suggested `@cast<int64>(h)` fix. Casting
  silences the warning *and* documents the intent that the
  runtime generation check is now the only safety net.

What still bypasses the static rule (by design, for now):

- **Transitive cross-function flow** — destroy-via-helper-via-helper
  is Phase-1-only (single-hop, regression: `bug269` documents the
  boundary).
- **Handles stashed in GC objects** — if the handle is reachable
  only through a `gc` reference and the destroying call is in
  another function, the static rule loses the thread.
- **Round-trips through opaque C state** — a handle that left
  for C and came back is not tracked.
- **Indirect aliasing through arrays** — a handle stored in an
  array slot is not currently traced through.
- **`Handle<T>` imported from another module** — the borrow
  checker only walks the main `ProgramNode.declarations` when
  collecting cross-function summaries, so `extern` handles
  imported via `use "handle.npk".*` are not visible. Locally
  re-declare the relevant extern (see `bug277`) to get the
  warning; broader fix deferred.

In all of those bypass cases the **runtime** generation check
still catches the misuse: deref returns `0i64`. The static rule
is a strict superset of safety — it never accepts code the
runtime would reject; it just refuses some code earlier.

## Interaction with `raw` / `drop`

The borrow checker peeks through `raw(...)` and `drop(...)` wrappers
on the initializer expression — `Handle<T>:h = raw HandleArena.alloc(...)`
is recognised the same as `Handle<T>:h = HandleArena.alloc(...)`.

Since v0.29.5, importing `drop.npk` opts arena bindings into
auto-destroy: `int64:a = HandleArena.create();` auto-emits
`npk_handle_arena_destroy(a)` at scope end. The outlives rule
still applies — bindings whose handles escape from a RAII-managed
arena trigger `ARIA-032` the same way. See the
[`guide/drop/`](../drop/README.md) cookbook (especially
[`regions.md`](../drop/regions.md) for the recognizer rules and
[`pitfalls.md`](../drop/pitfalls.md) for the manual + auto
double-free trap).

## Validation

- `bug260_handle_outlives_arena_deref.npk` — destroy then deref →
  compile-fail with `ARIA-032`.
- `bug261_handle_outlives_arena_free.npk`  — destroy then free  →
  compile-fail with `ARIA-032`.
- `bug262_handle_other_arena_alive_pass.npk` — sibling arena alive
  → compiles and exits 0.
- `bug263_handle_destroy_after_use_pass.npk` — correct teardown
  order → compiles and exits 0.
- `bug267`–`bug269` (v0.28.3) — cross-function Phase 1: callee
  destroys a parameter; sibling-arena handle still compiles; the
  transitive-through-wrapper case is documented as the Phase 1
  boundary.
- `bug270`–`bug272` (v0.28.4) — cross-function Phase 2 part A:
  returning a handle whose arena is local; positive arena-as-param
  control; inline `pass raw HandleArena.alloc(localArena, ..)`.
- `bug273`–`bug275` (v0.28.4.1) — Phase 2 part B: struct-binding
  return / inline struct literal.
- `bug276`–`bug278` (v0.28.5) — FFI passthrough warning, cast
  silences, end-to-end round trip via `@cast<int64>`.

## See also

- [Diagnostics](diagnostics.md) — full `ARIA-032` reference.
- [Arenas](arenas.md) — arena lifecycle.
