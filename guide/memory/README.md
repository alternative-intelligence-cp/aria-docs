# Memory regions

Nitpick gives you explicit control over **where** a value lives, separate
from **how** the borrow checker tracks aliasing. This guide explains the
five memory regions, when to use each, and how they interact with the
borrow checker, the `#` pin operator, and `wild`/`wildx` interop.

## Quick reference

| Region    | Keyword     | Backing                | Lifetime                          |
|-----------|-------------|------------------------|-----------------------------------|
| Default   | (none)      | LLVM `alloca`          | function frame                    |
| Stack     | `stack`     | LLVM `alloca`          | function frame                    |
| GC heap   | `gc`        | `npk_gc_alloc`         | until unreachable, then collected |
| Wild heap | `wild`      | `npk_alloc` (manual)   | user-managed                      |
| Wildx     | `wildx`     | `npk_alloc_exec` (W^X) | user-managed                      |

## Chapters

1. [Regions](regions.md) — the five regions in detail, with examples.
2. [Stack](stack.md) — the default region: when, why, and what escapes it.
3. [GC](gc.md) — `gc` heap allocation, generational mark-sweep,
   safepoints, auto-pinning, shadow-stack rooting.
4. [Wild](wild.md) — `wild` / `wildx` manual heap, FFI, and the
   leak/use-after-free diagnostics that catch the obvious mistakes.
5. [Pinning](pinning.md) — `#x`, what it does per region, and the
   pin-derived alias rules.
6. [Tuning](tuning.md) — runtime knobs (`NPK_GC_NURSERY_SIZE`,
   `NPK_GC_OLD_GEN_THRESHOLD`, `NPK_GC_MODE`).
7. [Interop](interop.md) — GC ↔ `wild` / `wildx` invariants and
   the `npk_shadow_stack_add_root` escape hatch.
8. [Diagnostics](diagnostics.md) — the memory-region diagnostic codes
   (`ARIA-014`, `ARIA-015`, `ARIA-028`, `ARIA-029`, `ARIA-031`,
   `ARIA-032`) — what they mean and how to fix the most common cases.
9. [FAQ](faq.md) — short answers to recurring questions about regions,
   the borrow checker, the GC, and FFI.

For the **handle** subsystem (generation-checked arena allocation,
`Handle<T>`, `ARIA-032`), see the separate
[`guide/handles/`](../handles/README.md) cookbook. For **runtime
code generation** on top of `wildx`, see the
[`guide/jit/`](../jit/README.md) cookbook. For the **opt-in RAII**
layer that auto-frees `wild` / `wildx` / `HandleArena` / `JitFn`
bindings at scope exit, see the [`guide/drop/`](../drop/README.md)
cookbook (v0.29.x).

## Validation snapshot (v0.28.8 — cycle close)

- **CTest:** 78/78 (70 at v0.27.10 close; +8 across v0.28.x:
  `bug_tests_v0281`–`v0285`, `bug_tests_v02841`, and two runtime
  stress entries `bug_tests_v0286` / `test_pin_gc_pressure_v02861`).
- **K core:** 151/151 (unchanged across v0.28.x — no new K core
  tests this cycle).
- **K proofs:** 11/11 (unchanged — v0.28.6 is runtime stress, not
  a new proof).
- **Bug regressions:** bug205–bug278 (+15 across v0.28.x:
  bug264–bug278 spanning JIT smoke, `jit.npk` helper, cross-
  function `ARIA-032` Phases 1–2 + FFI passthrough).
- **New surface (v0.28.x):** `stdlib/jit.npk` (`Type:Jit`), the
  v0.28.1 runtime helpers (`npk_jit_install_add_i32`,
  `npk_jit_call_i32_i32`), and the [JIT cookbook](../jit/README.md).
- Codegen audit recorded in
  [`META/NITPICK/ROADMAP/0.26/CODEGEN_AUDIT.md`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/AUDIT_v0.25.7.md).
- Borrow checker treatment of `stack` and `gc` is identical: the region
  is a runtime fact, not a borrow fact.
