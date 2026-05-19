# Pitfalls

The Drop mechanism is opt-in and conservative, but there are
sharp edges. This page is the honest list.

## 1. Hard `exit` skips drops

```nitpick
use "drop.npk".*;

func:demo = NIL() {
    wildx int8->:p = wildx_alloc(1024i64);
    exit 0;   // p is LEAKED — the kernel reclaims on process death
};
```

DROP-DEC-008. `exit N` translates to a `_Exit`-style hard
process exit; no destructors run. If you need cleanup, use
`pass` / `fail` to unwind back to `main` and exit from there.

The kernel reclaims the address-space allocation, so for
in-memory `wild`/`wildx`/`Handle` resources this leak is
harmless on process termination. For resources that survive
process death (files, sockets, locks held in shared memory),
explicit cleanup before `exit` is the only correct option.

## 2. Bare-identifier `pass` skips that binding's drop

```nitpick
use "drop.npk".*;

func:make_buf = wildx int8->() {
    wildx int8->:p = wildx_alloc(64i64);
    pass p;   // p's drop is SKIPPED — ownership moves to caller
};
```

DROP-DEC-004. This is the **forcing-function** behaviour that
makes factory functions work — every `Jit.compile_*` helper in
the stdlib relies on it. Without bare-identifier move,
`Jit.compile_add_i32()` would auto-free its returned page and
the JIT helper would UAF in every caller.

The pitfall is the inverse: if you write `pass other_expr;`
(not a bare identifier — e.g. `pass make_alias(p);`), the move
does **not** trigger and `p` still drops at scope end. If you
intended to transfer ownership, the call may double-free or
UAF downstream.

Rule of thumb: write the binding name plainly to transfer
ownership. Anything fancier and the conservative default
(drop everything) kicks in.

## 3. Mixing explicit free with auto-drop

```nitpick
use "drop.npk".*;

func:demo = NIL() {
    wildx int8->:p = wildx_alloc(64i64);
    raw wildx_free(p);   // manual free
    // ...auto-drop ALSO emits npk_wildx_free(p) at scope end → double-free
    pass;
};
```

In v0.29.x there is **no compile-time diagnostic** for the
manual-free + auto-drop collision. The successor cycle adds an
`ARIA-022` extension for it. Until then: pick one model per
binding.

If you genuinely need to free early, do not import
`drop.npk` in the file that holds the binding, and manage the
lifetime by hand.

## 4. Destructor failure is undefined this cycle

```nitpick
impl:Drop:for:MyType = {
    func:drop = NIL($m MyType:self) {
        fail 1i32;   // UNDEFINED BEHAVIOUR in v0.29.x
    };
};
```

DROP-DEC-009. A `drop` method that fails has no defined
semantics this cycle. The compiler does not statically reject
`fail` inside a `drop` body; it just does not specify what
happens. Future cycles will pick a policy (terminate the
process, propagate to the enclosing `failsafe`, swallow
silently — TBD). The four built-in `NitpickXxxRaii` sentinels
all use `pass;` and so are unaffected.

## 5. Drop-during-drop is allowed but unscheduled

If a type's `drop` calls into another type's `drop` (e.g. by
freeing a `wild` struct whose drop body in turn frees a child
binding), Nitpick runs both. There is no special detection or
re-entry guard — the per-thread drop stack just unwinds. This
is well-defined for the v0.29.x built-in regions because their
sentinel bodies do nothing user-visible; for user types,
DROP-DEC-009 (above) is the relevant unconstraint.

## 6. `main` cannot use `pass` or `fail`

Nitpick's `main` is exit-code-only: `pass` and `fail` inside
`main` are rejected by sema. To exercise the early-exit drop
paths, write a helper function and call it from `main`:

```nitpick
func:do_work = int32() {
    wildx int8->:p = wildx_alloc(16i64);
    pass 0i32;          // drops fire here
};

func:main = int32() {
    Result<int32>:r = do_work();
    if (r.is_error) { exit 1; }
    exit 0;
};
```

The check `r.is_error` is a `bool` — compare directly, **not**
`r.is_error != 0i8`.

## 7. `return v;` of a bare identifier is **rejected**

```nitpick
func:make = wildx int8->() {
    wildx int8->:p = wildx_alloc(64i64);
    return p;   // ERROR — "'return' cannot return a bare value"
};
```

Inside fallible functions, `return` requires an explicit
`Result{val: ..., err: ..., is_error: ...}` literal. To return
an owned value with move semantics, use `pass v;` instead.

This is a Nitpick syntax rule, not a Drop rule, but it bites
people writing factory functions. The good news: `pass v;`
does the right thing.

## 8. The `Jit_compile_*` recipe (the forcing-function gotcha)

When you add a new region to Drop, the very first thing to
re-run is `bug_tests_v0296`. The `Jit_compile_add_i32` helper
in `stdlib/jit.npk` does `pass page;` where `page` is a
`wildx` RAII binding. If the new region change breaks
bare-identifier move semantics (DROP-DEC-004) — even
indirectly — every JIT smoke test will UAF.

This came up live in v0.29.7: drops on `pass` were wired
before move semantics were threaded through, and every JIT
test crashed with exit 139. The fix was to thread
`moved_var_name` through `emitDropsForScope`.

## 9. The `gc` region has no Drop

There is no `impl:Drop` for `gc T:x = ...` bindings. The GC
owns those values; you free them by dropping the last reference
and letting the collector reap. Importing `drop.npk` has no
effect on `gc` bindings.

## 10. Region recognizer keys on the **initializer shape**

The recognizer for each region looks at the RHS of the binding,
not just the type:

- `wild T:x = T{ ... };` → auto-drop (struct-literal init).
- `wild T:x = make_holder();` → **not auto-dropped** (call init).
- `wildx int8->:p = wildx_alloc(N);` → auto-drop.
- `wildx int8->:p = some_helper();` → **not auto-dropped**.

This is by design: the recognizer needs to know who allocated
the resource. A function-returned `wildx` page has already had
its drop transferred via DROP-DEC-004; auto-dropping at the
callee's binding would double-free. The trade-off is that you
sometimes need to rebind via a recognizable pattern, or accept
explicit free for that binding.

## 11. The trait name **must** be `Drop`

```nitpick
impl:Cleanup:for:T = { /* ... */ };   // not Drop — registered as a regular trait
```

Only `impl:Drop:for:T` flips Drop semantics. The check is a
literal string match in sema.

## 12. v0.29.x is **opt-in everywhere or nothing**

There is no per-file or per-binding `use "drop.npk"` scope
distinction — the import flips compile-unit-wide flags. If you
mix a file that imports `drop.npk` with a file that does not
in the same program, both files get drops if any one of them
imports. This is a v0.29.x limitation; future cycles may make
the flags per-file.
