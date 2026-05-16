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
[`guide/handles/`](../handles/README.md) cookbook.

## Validation snapshot (v0.27.10 — cycle close)

- **CTest:** 70/70 (was 53/53 at v0.25.7 close, 108/108 at v0.26.7
  close; the count dropped from 108 because v0.26.x rolled many
  per-slice bug runners into umbrella entries, then v0.27.x added
  back per-slice entries with finer-grained labels).
- **K core:** 151/151 (was 145 at v0.26.x start; +6 across
  `146`–`151`).
- **K proofs:** 11/11 (was 10; +1 `pin-address-stable-proofs.k`).
- **Bug regressions:** bug205–bug263 (+59 across MEM and HANDLE
  series).
- Codegen audit recorded in
  [`META/NITPICK/ROADMAP/0.26/CODEGEN_AUDIT.md`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/AUDIT_v0.25.7.md).
- Borrow checker treatment of `stack` and `gc` is identical: the region
  is a runtime fact, not a borrow fact.
