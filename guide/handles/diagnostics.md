# Handle diagnostics

The handle subsystem owns one dedicated diagnostic code,
**`ARIA-032`**, plus the type-mismatch error you get from the general
type checker when you mix incompatible handle types.

For the broader set of memory-region codes (`ARIA-014`, `ARIA-015`,
`ARIA-028`, `ARIA-029`, `ARIA-031`), see
[`memory/diagnostics.md`](../memory/diagnostics.md).

---

## `ARIA-032` — handle outlives its arena

The borrow checker has seen `HandleArena.destroy(a)` earlier in the
same function, and now sees a `HandleArena.deref(h)` or
`HandleArena.free(h)` where `h` is a handle bound to `a`.

```aria
use "handle.npk".*;

func:main = int32() {
    int64:a            = raw HandleArena.create();
    Handle<int32>:h    = raw HandleArena.alloc(a, 4i64);
    raw HandleArena.destroy(a);
    int64:p = raw HandleArena.deref(h);   // [ARIA-032]
    exit 0;
};
```

```
error[ARIA-032]: Handle 'h' outlives its arena 'a'. The arena was
destroyed earlier in this function; the handle is no longer valid.
See ARIA-032 and guide/memory/handles.md.
```

**Fix:** either drop the deref/free, move it before
`HandleArena.destroy(a)`, or restructure so the arena outlives all
its handles:

```aria
int64:a            = raw HandleArena.create();
Handle<int32>:h    = raw HandleArena.alloc(a, 4i64);
int64:p = raw HandleArena.deref(h);       // use first
raw HandleArena.free(h);                   // then free
raw HandleArena.destroy(a);                // then destroy
```

### Scope

The rule is **intra-function** only. Handles that cross function
boundaries (parameters, returns) or go through struct storage / FFI
are tracked at runtime instead — `deref` returns `0i64`. The static
rule is a safety net for the common straight-line case; it never
accepts code the runtime would reject.

### Regression tests

- `bug260_handle_outlives_arena_deref.npk` — deref after destroy.
- `bug261_handle_outlives_arena_free.npk` — free after destroy.
- `bug262_handle_other_arena_alive_pass.npk` — sibling arena.
- `bug263_handle_destroy_after_use_pass.npk` — correct teardown.

---

## Type-mismatch on `Handle<T>`

Assigning between two `Handle<T>` types with different `T` is a type
error from the general type checker:

```aria
Handle<int32>:hi = raw HandleArena.alloc(a, 4i64);
Handle<int64>:hl = hi;          // error: cannot assign Handle<int32> to Handle<int64>
```

`Handle<T>` ↔ `int64` is bidirectional and silent, so the way out is
to round-trip through `int64`:

```aria
int64:tok        = hi;
Handle<int64>:hl = tok;          // OK — explicit
```

This is the same pattern you would use for FFI; `bug257_handle_type_mismatch_rejected.npk`
is the regression test.

---

## Runtime NULL deref (not a diagnostic)

`HandleArena.deref(h)` returning `0i64` is not a compile error — it
is the *runtime* contract for stale or freed handles. Always check:

```aria
int64:p = raw HandleArena.deref(h);
if (p == 0i64) {
    exit 1;        // handle was stale; recover or bail
};
```

`bug258_handle_uaf_returns_null.npk` and
`bug259_arena_destroy_invalidates.npk` (now compile-failing under
`ARIA-032` after v0.27.9, but the runtime behaviour is unchanged)
cover this.

## See also

- [Lifetimes](lifetimes.md) — the static rule in detail.
- [Handles](handles.md) — runtime behaviour.
- [`memory/diagnostics.md`](../memory/diagnostics.md) — region codes.
- [`borrow/diagnostics.md`](../borrow/diagnostics.md) — non-region
  borrow codes.
