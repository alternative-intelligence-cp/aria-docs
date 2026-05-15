# Wild

`wild` opts out of the borrow checker. The value lives on the
unmanaged heap (`npk_alloc`), and the user is responsible for pairing
the allocation with the matching release.

```aria
func:scratch = int32() {
    wild int32:buf = 0i32;
    defer free(buf);            // ARIA-014 requires this
    // ... use buf via FFI ...
    pass 0i32;
};
```

`wild` exists for FFI buffers and interop with C-style APIs. **Do not**
reach for it as an escape hatch from the borrow checker — that is
what [`gc`](gc.md) is for.

## What "exempt from the borrow checker" means

The checker still tracks the *allocation site* (for `ARIA-014` /
`ARIA-015`), but does not enforce mutability-XOR-aliasing on the
contents. Inside FFI you are running C-side rules; the language
trusts you, and therefore so does the optimiser.

The exemption is wired in
[`borrow_checker.cpp:1805`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/src/frontend/sema/borrow_checker.cpp#L1805):

```cpp
bool is_wild_type = stmt->isWild || stmt->typeName.find("wild") == 0;
if (is_wild_type && stmt->initializer && !stmt->is_pinned_shadow) {
    stmt->requires_drop = true;
    // ...track allocation, do not enforce borrow rules on contents...
}
```

## Diagnostics that still apply

Even though aliasing is unchecked, two leak/use-after-free guards
remain:

- **`ARIA-014`** — every `wild` allocation must have a matching
  `defer free(...)` (or other release) in the same scope. The check
  fires at scope exit.
- **`ARIA-015`** — using a `wild` variable after it has been released
  (`free`d, moved, or otherwise drop-tracked) is rejected.

See [`diagnostics.md`](diagnostics.md) for examples and fix recipes.

## `wildx` — manual heap, executable (W^X)

`wildx` is `wild` with `PROT_EXEC` (write-XOR-execute discipline). It
exists to support runtime code generation: allocate, write machine
bytes with the W mapping, `mprotect` to PROT_EXEC, then call as a
function pointer.

```aria
// Typical wildx usage: call npk_alloc_exec() directly, write machine
// bytes, mprotect to PROT_EXEC, then call as a function pointer.
```

In practice you call `npk_alloc_exec()` directly from user code rather
than declaring a `wildx` variable; the keyword is parsed for symmetry.

## Interop with `gc`

The two heaps are **disjoint**: the GC marker never follows a `wild`
pointer (verified end-to-end by `bug230` against the runtime accessor
`npk_gc_is_heap_pointer_i32`), and a `wild` slot does not root anything
the GC can see (`bug231` confirms a `gc` binding survives 5000 wild
alloc/free cycles around explicit safepoints).

If you genuinely need a `wild` slot to root a `gc` reference, the
escape hatch is `npk_shadow_stack_add_root`. Full discussion in
[`interop.md`](interop.md).

## ABI gotcha

When exposing C functions that return a region predicate, **never** use
a 1-byte return type (`_Bool`, `uint8_t`) and read it as `int32` from
Nitpick. The x86-64 SysV ABI leaves the upper EAX bits undefined for
sub-word returns, and the Nitpick extern call will see garbage.
Always widen at the C side — see the v0.26.5 fix that introduced
`npk_gc_is_heap_pointer_i32` (full `int32_t` return) alongside the
legacy `_Bool`-returning variant.

## Validation

- CTest: `bug_tests_v0265` (heap partition + churn coexistence).
- Existing borrow-checker leak/UAF coverage: `ARIA-014` / `ARIA-015`
  fixtures predate v0.26.x.
