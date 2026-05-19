# Regions

Drop covers four resource regions in v0.29.x. Each region has
its own per-binding-shape recognizer in the IR generator; each
flips on the same `use "drop.npk".*;` import. This page
walks the per-region behaviour and the RAII-vs-explicit
decision.

## The four supported binding shapes

| Region        | Binding shape                                 | Auto-emit                          | Slice    |
|---------------|-----------------------------------------------|-------------------------------------|----------|
| `wild` struct | `wild T:x = T{ ... };`                        | `npk_free(x)`                       | v0.29.3  |
| `wildx`       | `wildx T->:p = wildx_alloc(N);`               | `npk_wildx_free(p)`                 | v0.29.4  |
| `HandleArena` | `int64:a = HandleArena.create();`             | `npk_handle_arena_destroy(a)`       | v0.29.5  |
| `JitFn`       | `wildx int8->:f = Jit.compile_add_i32();`     | `npk_wildx_free(f)`                 | v0.29.6  |

In every case the compiler:

1. Sees the binding pattern (RHS shape matters — the recognizer
   peels `gc` / `pin` wrappers and prefix-matches the
   allocator call name).
2. Pushes a `DropEntry{kind, varName, alloca, typeName}` onto
   the current scope's drop stack.
3. At every exit path for that scope, walks the stack in
   reverse declaration order and emits the matching free.

Borrow-checker integration (where applicable) mirrors the
codegen recognizer so `ARIA-014` (leak) does not fire on
RAII-managed bindings.

## `wild T:x = T{ ... };` (v0.29.3)

```nitpick
use "drop.npk".*;

struct:Holder = { value: int64, };

func:demo = NIL() {
    wild Holder:h = Holder{ value: 42i64 };
    // h is auto-freed at scope end via npk_free(h).
    pass;
};
```

- The recognizer keys on `wild T:x = T{...}` where T is a
  struct (not a pointer-typed `wild`).
- `wild T->:p = alloc(...)` (raw pointer-typed wild) is **not
  on the surface** this cycle — that's the deferred v0.29.3b
  slice. Manual `npk_free(p)` is still required there.

## `wildx T->:p = wildx_alloc(N);` (v0.29.4)

```nitpick
use "drop.npk".*;

func:demo = NIL() {
    wildx int8->:page = wildx_alloc(64i64);
    // Write bytes, optionally seal, then return.
    // Auto-emits npk_wildx_free(page) at scope end.
    pass;
};
```

- Works whether or not `wildx_seal` is called. The runtime
  state-machine accepts a free on either WRITABLE or SEALED
  pages.
- A binding initialised by `Jit.compile_*` flows through the
  v0.29.6 JitFn recognizer instead (separate flag — see below).

## `int64:a = HandleArena.create();` (v0.29.5)

```nitpick
use "drop.npk".*;
use "handle.npk".*;

func:demo = NIL() {
    int64:arena = HandleArena.create();
    int64:h = HandleArena.alloc(arena, 32i64);
    // Auto-emits npk_handle_arena_destroy(arena) at scope end.
    // Per-handle auto-free for `h` is NOT emitted —
    // handles stay on the explicit-HandleArena.free contract.
    pass;
};
```

- Destroying the arena bumps every slot's generation, so
  outstanding `Handle<T>` values become stale automatically.
  No per-handle free is needed for correctness.
- `ARIA-032` (handle outlives arena) continues to fire on
  escape attempts from a RAII-managed arena.

## `wildx int8->:f = Jit.compile_add_i32();` (v0.29.6)

```nitpick
use "drop.npk".*;
use "jit.npk".*;

func:demo = int32() {
    wildx int8->:f = Jit.compile_add_i32();
    int32:r = Jit.call_i32_i32(f, 2i32, 3i32);
    // Auto-emits npk_wildx_free(f) at scope end.
    pass r;
};
```

- The flag is **independent** from the generic `wildx` flag
  (DROP-DEC-007 — a program may enable JIT auto-free without
  enabling blanket `wildx` RAII, even though `drop.npk`
  currently flips both at once).
- The recognizer prefix-matches `Jit_compile_*` so future
  signatures (`compile_mul_i32`, …) inherit RAII without code
  changes.

## The RAII-vs-explicit decision

| Situation                                              | Choose       |
|--------------------------------------------------------|--------------|
| Function-local binding, single owner, simple lifetime  | **Drop**     |
| Binding escapes via `pass v` to caller                 | **Drop** + bare-identifier move (DROP-DEC-004) |
| Binding stored in a `gc` struct, transferred by value  | explicit free at the new owner's end |
| Long-lived process resource (started once at boot)     | explicit free in a teardown helper, or never |
| Conditional release based on runtime state             | explicit free + don't import `drop.npk` for that binding's type |
| Need to free **before** scope end (e.g. release pressure mid-function) | explicit free — but be aware v0.29.x will then double-free at scope end (diagnostic coming in successor cycle) |

The honest rule: if you are not sure, start with Drop. The
diagnostic for "you also wrote an explicit free" is not in
v0.29.x yet, so the one place to be careful is mixing both for
the same binding.

## Borrow-checker visibility

Drops are emitted by the IR generator. The borrow checker has
its own parallel recognizers (mirroring the codegen gates) so
that bindings under RAII do not trip `ARIA-014` for "missing
free". The two recognizers must agree — every slice that adds
a new region pattern to codegen also extends the borrow
checker.

`ARIA-022` (double-free) and `ARIA-015` (use-after-free) on
the underlying pointer continue to fire as before. Drop does
not weaken them.

## Validation

- `bug_tests_v0293` — wild struct RAII (bug285–bug288).
- `bug_tests_v0294` — wildx RAII (bug289–bug293).
- `bug_tests_v0295` — HandleArena RAII (bug294–bug296).
- `bug_tests_v0296` — JitFn RAII (bug297–bug300).
- All four region recognizers exercised by `bug_tests_v0297`
  early-exit fixtures (bug301–bug310).
