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
2. [Tuning](tuning.md) — runtime knobs (`NPK_GC_NURSERY_SIZE`,
   `NPK_GC_OLD_GEN_THRESHOLD`, `NPK_GC_MODE`).
3. [Interop](interop.md) — GC ↔ `wild` / `wildx` invariants and
   the `npk_shadow_stack_add_root` escape hatch (v0.26.5).

> Subsequent chapters (`stack`, `gc`, `pinning`, `diagnostics`,
> `faq`) will land alongside the remaining v0.26.x slices.

## Validation snapshot (v0.26.0)

- K core tests cover all three runtime-allocated paths
  (`146_alloc_gc_pass.aria`, `147_alloc_stack_pass.aria`,
  `148_alloc_default_is_stack_pass.aria`).
- Codegen audit recorded in
  [`META/NITPICK/ROADMAP/0.26/CODEGEN_AUDIT.md`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/AUDIT_v0.25.7.md).
- Borrow checker treatment of `stack` and `gc` is identical: the region
  is a runtime fact, not a borrow fact.
