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

## Concurrency note (v0.27.4)

The pin proof models a single mutator interleaved with the GC. A
concurrent-collector stress test (originally `bug244`) was deferred
out of v0.27.4 — the proof covers the algorithmic guarantee; the
multi-thread stress harness would only exercise the runtime
implementation and is tracked as a future cycle item.
