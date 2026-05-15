# GC

The `gc` keyword routes a value through `npk_gc_alloc` onto the
generational mark-sweep GC heap. The runtime tracks reachability and
reclaims the value when no live reference can reach it.

```aria
func:make_node = int32() {
    gc Node:n = Node{value: 7i32, next: 0};
    int32:v = n.value;
    pass v;     // n becomes unreachable here; GC will reclaim it
};
```

## When to use `gc`

- The value's lifetime must outlast the current frame (returning it,
  storing it in another `gc` structure, putting it in a queue).
- The value participates in a cyclic graph (linked lists, trees with
  back-pointers, message passing).
- You want the runtime to manage the memory rather than threading
  ownership manually.

For everything else, prefer [`stack`](stack.md). For C-style FFI use
[`wild`](wild.md).

## What happens at allocation (v0.26.3+)

Every `gc` binding is, at allocation time:

1. Allocated through `npk_gc_alloc` (correct ABI: `(i64 size, i16
   type_id=0)`, fixed in v0.26.2).
2. **Auto-pinned** via `npk_gc_pin` — the underlying object is excluded
   from evacuation, so its address is stable across collection cycles.
3. **Shadow-stack rooted** via a lazy `npk_shadow_stack_push_frame` and
   a per-binding `<name>.rootslot` alloca, with a matching
   `pop_frame` on every function exit.

This means a `gc` binding is safe to use across `npk_gc_safepoint()`
calls, including implicit safepoints inside other allocations, without
any user-side bookkeeping.

```aria
func:hold = int32() {
    gc Holder:h = Holder{value: 1234i32};
    loop (i in 0i32..5000i32) {
        wild int32:scratch = 0i32;     // wild allocs trigger safepoints
        npk_gc_safepoint();
    };
    pass h.value - 1234i32;            // 0 — h survived 5000 cycles
};
```

## Borrow rules

The borrow checker treats `gc` exactly like `stack`. `$$i` / `$$m`
borrows obey the same mutability-XOR-aliasing rule, the same scope
discipline, and the same escape rules:

```aria
func:demo = int32() {
    gc int32:x = 10i32;
    $$m int32:r = x;
    int32:val = r + 1i32;
    pass val;
};
```

The region is a **runtime fact**, not a borrow fact. The K model picks
this up in `146_alloc_gc_pass.aria` and the pin tests
`149_pin_gc_pass.aria` / `150_pin_gc_mut_borrow_failsafe.aria`.

## The `#` pin alias

`#x` on a `gc` binding produces a wild pointer alias whose address is
stable. Because the binding is already pinned + rooted at allocation,
`#x` lowers to a pure SSA-pointer pass-through (no per-use
`npk_gc_pin` call):

```aria
gc Holder:h = Holder{value: 1234i32};
wild Holder->:p = #h;
// p remains a valid pointer to h across safepoints.
```

See [`pinning.md`](pinning.md) for the full pin-derived alias rules.

## Tuning

The GC reads three env-vars at init time
(`NPK_GC_NURSERY_SIZE`, `NPK_GC_OLD_GEN_THRESHOLD`, `NPK_GC_MODE`).
Resolution order: explicit args to `npk_gc_init()` →
env-vars → compiled-in defaults. Full guide: [`tuning.md`](tuning.md).

## Validation

- K core: `146_alloc_gc_pass.aria`, `149_pin_gc_pass.aria`,
  `150_pin_gc_mut_borrow_failsafe.aria`.
- CTest: `bug_tests_v0262` (50k small + 5k large + 30k churn),
  `bug_tests_v0263{0..3}` (pin × gc), `bug_tests_v0264` (tuning),
  `bug_tests_v0265` (wild interop).
