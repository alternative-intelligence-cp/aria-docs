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

### Surface lifecycle (v0.27.5+)

The W^X state machine is exposed through three pointer-keyed builtins:

```aria
wildx int8->:page = wildx_alloc(64);   // UNINITIALIZED -> WRITABLE
// ... write machine bytes into `page` ...
fixed int32:rc1 = wildx_seal(page);    // WRITABLE -> EXECUTABLE
// ... call into the page (e.g. via an extern jit_call helper) ...
fixed int32:rc2 = wildx_free(page);    // EXECUTABLE -> FREED (munmap)
```

Each pointer is registered in a process-wide
`std::unordered_map<void*, WildXGuard>` keyed by the writable address.
The guards own the FNV-1a code hash, ASLR jitter base, and quota
accounting; surface code never sees the struct directly.

The runtime enforces:

- **One-way seal** — `wildx_seal` only accepts pages in the WRITABLE
  state. Calling it twice returns `-1` (regression: `bug246`).
- **Hardware W^X** — once sealed, the page is `PROT_EXEC` only;
  writes trap with `SIGSEGV` from the MMU.
- **Single free** — `wildx_free` removes the registry entry on the
  first call. A second call (or a foreign `wild alloc()` pointer)
  returns `-1` because the lookup misses.

The borrow checker registers `wildx_free` as a deallocator, so the
same wild-leak (`ARIA-014`) and double-free (`ARIA-022`) diagnostics
that protect plain `wild` apply to the new builtins (regressions:
`bug247` double-free, `bug248` leak). Compile-time rejection is
preferred over runtime rejection; the runtime rc=-1 path exists as a
defense-in-depth backstop for code that smuggles pointers through
function boundaries the borrow checker cannot see across.

### Legacy form

The pre-v0.27.5 shape `wildx int8->:b = alloc(N); free(b);` still
parses and compiles, but it routes through plain `npk_alloc` (no
W^X mapping). New code should prefer the `wildx_alloc` / `wildx_seal` /
`wildx_free` triple to actually reach `npk_alloc_exec`.

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
