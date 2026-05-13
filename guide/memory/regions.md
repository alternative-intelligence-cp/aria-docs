# Regions

Nitpick has five memory regions. Two are stack-shaped (`alloca`), one is
managed by the GC, and two are unmanaged (`wild`, `wildx`). The borrow
checker is **region-agnostic**: it tracks aliasing the same way for
`stack` and `gc` values, and ignores `wild`/`wildx` entirely.

---

## Default (no keyword)

A bare local declaration uses the **stack** region.

```aria
func:main = int32() {
    int32:x = 5;        // stack alloca, lives for the function frame
    Foo:f = Foo{a: 1};  // stack alloca of the struct value
    exit x + f.a;
};
```

This matches what the codegen actually emits today
(`src/backend/ir/codegen_stmt.cpp:602`). The earlier "non-primitives are
GC by default" lore was wrong; only the explicit `gc` keyword routes
through `npk_gc_alloc`.

## `stack` — explicit stack allocation

`stack` is a documentation-grade keyword: it produces the same LLVM
`alloca` as the bare form, but makes the intent obvious at the call
site, and gives the borrow checker an explicit anchor for "must not
escape this frame" diagnostics in future slices.

```aria
func:make_pair = int32(int32:a, int32:b) {
    stack int32:lhs = a;
    stack int32:rhs = b;
    exit lhs + rhs;
};
```

**Lifetime:** the value is destroyed when the surrounding function
returns. Returning or `pass`-ing a borrow into a `stack` local is a
hard error (`ARIA-017`):

```text
error: [ARIA-017] Cannot return 'r' — it borrows from local variable 'x'
       which will be destroyed when the function returns
```

This applies transitively: a borrow chained through another local
(`$$m int32:r2 = r1;`) is rejected the same way. The diagnostic is
deliberately conservative — even borrows into `gc` *bindings* are
rejected, because the named handle goes out of scope. Path-sensitive
relaxation is deferred to a later cycle.

## `gc` — explicit GC heap allocation

`gc` puts the value on the GC heap. It is then automatically reclaimed
when no live reference can reach it.

```aria
func:make_node = int32() {
    gc Node:n = Node{value: 7, next: 0};
    exit n.value;   // 'n' becomes unreachable here; GC will reclaim it
};
```

Use `gc` when:

- The value's lifetime needs to outlast the current frame (returning it,
  storing it in another `gc` structure, putting it in a queue).
- The value participates in a cyclic graph (linked lists, trees with
  back-pointers, message passing).
- You want the runtime to manage the memory rather than threading
  ownership manually.

The runtime is generational mark-sweep; see the upcoming `tuning.md`
chapter for knobs.

## `wild` — manual heap allocation

`wild` opts out of the borrow checker. The value lives on the unmanaged
heap, and the user is responsible for pairing the allocation with the
matching release.

```aria
func:scratch = int32() {
    wild int32:buf = 0;
    // ... use buf via FFI, free it explicitly via npk_free or equivalent
    exit 0;
};
```

`wild` is intended for FFI buffers and interop with C-style APIs. Don't
reach for it as an escape hatch from the borrow checker — that is what
`gc` is for.

## `wildx` — manual heap, executable (W^X)

`wildx` is `wild` with `PROT_EXEC` (write-XOR-execute discipline). It
exists to support runtime code generation. In practice, you call
`npk_alloc_exec()` directly from user code rather than declaring a
`wildx` variable; the keyword is parsed for symmetry.

```aria
// Typical wildx usage: call npk_alloc_exec() directly, write machine
// bytes, mprotect to PROT_EXEC, then call as a function pointer.
```

---

## Borrow rules across regions

The borrow checker treats `stack` and `gc` identically:

```aria
func:main = int32() {
    gc int32:x = 10;
    stack int32:y = 20;

    $$m int32:rx = x;     // OK
    $$m int32:ry = y;     // OK — independent host
    exit rx + ry;
};
```

`wild` and `wildx` declarations skip the borrow checker entirely
(`src/frontend/sema/borrow_checker.cpp:1805`). Aliasing rules are not
enforced; that is the price of opting out.

## Pinning across regions

The pin operator `#` works on all four borrow-checked regions
(`stack`, default, `gc`). For `stack` it is conceptually a no-op (the
value is already trivially pinned to its frame); for `gc` it tells the
collector to exclude the value from evacuation, which keeps its address
stable across `npk_gc_safepoint()` calls.

`wild` / `wildx` values do not need pinning — there is no collector to
move them.

## What's coming

- **v0.26.1**: ✅ stack escape detection regression suite (`ARIA-017`
  already covers `stack`/chained/field-borrow paths; bug205–209).
- **v0.26.2**: ✅ GC stress smoke (`bug210` 50k small allocs,
  `bug211` 5k large allocs, `bug212` survivor holder + 30k churn) and
  `npk_gc_alloc` IR ABI fix — codegen now declares/calls
  `(i64 size, i16 type_id=0)` matching the runtime, eliminating a
  latent garbage-`type_id` bug on x86-64 SysV.
- **v0.26.3**: ✅ borrow + GC safepoint interaction validated
  (`bug213`/`bug214` `$m` borrow of `gc` survives explicit safepoint and
  churn-driven implicit minor GC; `bug215` confirms `ARIA-023` still
  rejects conflicting `$m` borrows of `gc` bindings). MEM-010 pin syntax
  `#x` lowering deferred — runtime is in place but the compiler does
  not yet emit `npk_gc_pin`/`npk_gc_unpin`.
- **v0.26.4**: tuning knobs (`NPK_GC_NURSERY_SIZE`, etc.).
- **v0.26.5**: `wild` / `wildx` interop with GC tracing.
- **v0.26.6**: diagnostics polish.
- **v0.26.7**: cycle close — full 9-chapter cookbook, audit.
