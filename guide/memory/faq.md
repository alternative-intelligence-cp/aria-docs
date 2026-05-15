# Memory FAQ

Short answers to recurring questions. For longer-form treatment see the
chapter linked at the end of each entry.

---

### What region does a bare local declaration use?

The **stack** region, same as the explicit `stack` keyword. The earlier
"non-primitives are GC by default" lore was wrong; only the explicit
`gc` keyword routes through `npk_gc_alloc`. See [`regions.md`](regions.md).

---

### When should I pick `gc` over `stack`?

When the value's lifetime needs to outlast the current frame: returning
it, storing it in another `gc` structure, putting it in a queue, or
participating in a cyclic graph. Otherwise, prefer `stack`. See
[`gc.md`](gc.md).

---

### Is `gc` safe across `npk_gc_safepoint()` calls?

Yes. Every `gc` binding is auto-pinned and shadow-stack rooted at
allocation (v0.26.3.2+). You don't need to add a `#` pin yourself for
this case; the binding name keeps the object reachable and the
auto-pin keeps its address stable. See [`gc.md`](gc.md) and
[`pinning.md`](pinning.md).

---

### Why does the borrow checker reject `pass $$m local;`?

That's `ARIA-028 STACK_ESCAPE`. The host's stack frame is destroyed
when the function returns, so the borrow would dangle. Return by
value, take ownership in the caller, or accept a borrow parameter and
return that. See [`diagnostics.md`](diagnostics.md#aria-028--stack-escape-returnpass-of-borrow-into-a-local).

---

### Can a borrow into a `gc` binding escape the function?

No — `ARIA-028` is conservative and rejects this even though the
underlying object survives on the heap. The named binding goes out of
scope, and the borrow path is rooted in the binding name. The
path-sensitive relaxation that would allow this is deferred. See
[`stack.md`](stack.md).

---

### Does the GC follow `wild` pointers?

No. The GC heap and the libc-malloc-backed `wild` heap are disjoint —
verified end-to-end by `bug230` against `npk_gc_is_heap_pointer_i32`.
A `wild` slot does not root anything the GC can see, either; if you
need that, use `npk_shadow_stack_add_root` as documented in
[`interop.md`](interop.md).

---

### How do I tune the GC?

Three env-vars read at GC init: `NPK_GC_NURSERY_SIZE`,
`NPK_GC_OLD_GEN_THRESHOLD`, `NPK_GC_MODE`. Resolution order: explicit
arg → env-var → compiled-in default. Full table and verification
recipe in [`tuning.md`](tuning.md).

---

### Is there a way to opt out of the borrow checker?

`wild` does that for the contents of the allocation (the leak/UAF
guards `ARIA-014` / `ARIA-015` still apply at the allocation site).
`wild` is intended for FFI buffers and interop with C-style APIs, not
as a general escape hatch. For owned values that need to outlive the
frame, use `gc`. See [`wild.md`](wild.md).

---

### Does `#x` extend lifetime?

No. `#x` only promises **address stability** — the underlying storage
will not move for the lifetime of the alias. Lifetime is a separate
concern handled by the borrow checker (and the GC reachability graph
for `gc` bindings). See [`pinning.md`](pinning.md).

---

### What does `npk_shadow_stack_add_root` do, and when do I need it?

It registers an arbitrary memory location as a GC root, so the marker
will trace whatever pointer lives at that address. The narrow case it
exists for: a `wild` slot must hold a `gc` reference that the GC
needs to see. Almost no user code should need this; the auto-pin +
auto-root behaviour for `gc` bindings handles the common cases. See
[`interop.md`](interop.md).

---

### Why `ARIA-028` and not `ARIA-017` or `ARIA-027`?

History. The compiler tagged stack-escape errors as `ARIA-017` from
v0.26.1 through v0.26.5; the docs promised `ARIA-027` for the same
concept but the code never used that tag. v0.26.6 unified everything
under `ARIA-028 STACK_ESCAPE` with polished message text. See
[`diagnostics.md`](diagnostics.md#aria-028--stack-escape-returnpass-of-borrow-into-a-local).
