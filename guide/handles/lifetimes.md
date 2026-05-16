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

- **Cross-function flow.** Passing a handle into a callee that
  destroys its arena, or returning a handle whose arena went out of
  scope in the caller. The intra-function rule is sound; the
  cross-function extension is deferred.
- **Indirect aliasing through structs / arrays.** If you stash a
  handle in a struct field and then destroy its arena, the rule
  does not currently trace that.
- **FFI round-trips.** A handle that left for C and came back is
  not tracked.

In all of those bypass cases the **runtime** generation check still
catches the misuse: deref returns `0i64`. The static rule is a
strict superset of safety — it never accepts code the runtime would
reject; it just refuses some code earlier.

## Interaction with `raw` / `drop`

The borrow checker peeks through `raw(...)` and `drop(...)` wrappers
on the initializer expression — `Handle<T>:h = raw HandleArena.alloc(...)`
is recognised the same as `Handle<T>:h = HandleArena.alloc(...)`.

## Validation

- `bug260_handle_outlives_arena_deref.npk` — destroy then deref →
  compile-fail with `ARIA-032`.
- `bug261_handle_outlives_arena_free.npk`  — destroy then free  →
  compile-fail with `ARIA-032`.
- `bug262_handle_other_arena_alive_pass.npk` — sibling arena alive
  → compiles and exits 0.
- `bug263_handle_destroy_after_use_pass.npk` — correct teardown
  order → compiles and exits 0.

## See also

- [Diagnostics](diagnostics.md) — full `ARIA-032` reference.
- [Arenas](arenas.md) — arena lifecycle.
