# Pinning

The `#` operator pins a value: it asserts that the address of the
underlying storage will not change for the lifetime of the pin alias.
Pinning matters most for [`gc`](gc.md) bindings, where the collector
can otherwise evacuate objects between safepoints.

```aria
gc Holder:h = Holder{value: 1234i32};
wild Holder->:p = #h;
// p is a stable pointer to h, safe to hand to FFI.
```

## Per-region behaviour

| Region            | What `#x` means                                                   |
|-------------------|-------------------------------------------------------------------|
| Default / `stack` | No-op — the value is trivially pinned to its frame.               |
| `gc`              | Pure SSA pointer pass-through — the binding was already pinned + rooted at allocation (v0.26.3.2+). |
| `wild` / `wildx`  | Not needed — there is no collector to move the value.             |

The `#` operator does **not** lower to a per-use `npk_gc_pin` call,
because every `gc` binding is auto-pinned at allocation and rooted on
the shadow stack. Pinning is therefore an aliasing/typing affordance,
not a runtime side effect.

## Pin-derived alias rules

A pointer derived from a pin (`wild T->:p = #h;`) inherits the pin's
read-only contract for the borrow checker. The alias may not be used
to:

- Mutate through it as if it were a fresh `$$m` borrow.
- Escape into a callee that takes ownership.
- Be re-bound to a different host.

These rules are tracked by the borrow checker via
`pin_derived_aliases` in `LifetimeContext` and enforced at every
assignment / call site that touches the alias.

## Surviving safepoints

The whole point of pinning a `gc` value is to keep its address stable
across implicit and explicit safepoints. The regression tests
exercise exactly this:

```aria
gc Holder:h = Holder{value: 1234i32};
wild Holder->:p = #h;
loop (i in 0i32..5000i32) {
    wild int32:scratch = 0i32;
    npk_gc_safepoint();
};
pass p->value - 1234i32;            // 0 — p still points at h
```

The K model proves the borrow-checker side of this in
`149_pin_gc_pass.aria` and
`150_pin_gc_mut_borrow_failsafe.aria`. The runtime side is covered by
bug224 / bug225.

## Where pinning is **not** the answer

- **"My borrow keeps escaping"** — that's [`stack`](stack.md) and the
  borrow checker, not pinning. Pin doesn't extend lifetime; it only
  promises address stability.
- **"I want to share between threads"** — pinning has nothing to say
  about concurrency. Use the appropriate synchronisation primitive.
- **"I want a manually-managed buffer"** — that is [`wild`](wild.md).

## Validation

- K core: `149_pin_gc_pass.aria`, `150_pin_gc_mut_borrow_failsafe.aria`.
- K proofs: `pin-address-stable-proofs.k` (v0.27.4) — three claims
  formalising that pin-derived pointer addresses are stable across
  `npk_gc_safepoint` calls (PIN-DEC-004 part 1).
- CTest: `bug_tests_v0263{0..3}` (pin baseline, allocation qualifiers,
  auto-pin + shadow-stack root, MEM-010 reinstatement).
- Audit: `META/NITPICK/ROADMAP/done/0.26/0.26.3.x/PIN_LOWERING_AUDIT.md`.

## Concurrency note (v0.27.4 → v0.28.6.1)

The pin proof models a single mutator interleaved with the GC. The
concurrent-collector stress harness (originally tracked as
`bug244`) was deferred out of v0.27.4 — the proof covers the
algorithmic guarantee; the multi-thread harness is about exercising
the runtime implementation.

v0.28.6 lands that harness:
`tests/runtime/test_pin_concurrent_v0286.cpp` — 4 mutator threads,
5000 iterations each, of `alloc(64)` → write magic → `pin` →
`npk_gc_safepoint` → verify address-stable and magic-roundtrip
→ `unpin`. The pin protocol is exercised 20000 times under
genuine multi-mutator `gc_mutex` contention; the assertion is
that pin-derived addresses remain stable through every safepoint
(PIN-DEC-004 part 2, the runtime confirmation of part 1's K proof).

v0.28.6.1 fixes a latent deadlock the stress harness uncovered:
`GCCoroAllocator::init_gc()` used to call `npk_gc_init()` from
its constructor, and the allocator is lazy-constructed from inside
`minor_gc()` / `major_gc()` while `gc_mutex` is already held —
nested `npk_gc_init` then re-acquired the same non-recursive
mutex and self-deadlocked on the first minor GC. The fix removes
the nested init call; one-time init is the caller's responsibility
(already the convention everywhere else).
`tests/runtime/test_pin_gc_pressure_v02861.cpp` is the regression:
4 threads × 2000 iters with a forced 64 KB nursery so a minor GC
*must* fire under the pin protocol; it asserts
`stats.num_minor_collections > 0` to keep the test from
fast-pathing past the bug.
