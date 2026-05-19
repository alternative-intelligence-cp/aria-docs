# Drop / RAII

Nitpick's **Drop** mechanism is the opt-in RAII layer that
auto-cleans-up resources at scope exit. Importing
[`stdlib/drop.npk`](https://github.com/alternative-intelligence-cp/nitpick/blob/dev-0.29.x/stdlib/drop.npk)
enables drop emission for a fixed set of resource regions —
`wild` struct bindings, `wildx` pointer bindings, `HandleArena`
handles, and `JitFn` pages — without changing the source-level
spelling of those regions.

Without the import, those bindings keep the v0.29.2-and-earlier
**explicit-cleanup contract**: you call `npk_free` / `wildx_free` /
`HandleArena.destroy` / `Jit.free` yourself, and `ARIA-014` fires
on leaks. Drop is purely additive.

## Quick reference

| With `use "drop.npk".*;`               | Auto-emit at scope end                  |
|----------------------------------------|------------------------------------------|
| `wild T:x = T{ ... };`                 | `npk_free(x)`                            |
| `wildx T->:p = wildx_alloc(N);`        | `npk_wildx_free(p)`                      |
| `int64:a = HandleArena.create();`      | `npk_handle_arena_destroy(a)`            |
| `wildx int8->:f = Jit.compile_add_i32();` | `npk_wildx_free(f)`                  |

Each region has its own opt-in sentinel struct
(`NitpickWildRaii`, `NitpickWildxRaii`,
`NitpickHandleArenaRaii`, `NitpickJitFnRaii`). The compiler keys
on the presence of each `impl:Drop:for:NitpickXxxRaii`
declaration. They all currently flip on the same `drop.npk`
import; a future cycle may let a program enable one without the
others.

## When Drop runs

Drop calls fire at every scope-exit path:

1. **Fall-through** — the block reaches its closing `}` without
   a terminator.
2. **`pass v`** — early successful exit from a function.
3. **`fail e`** — early TBB-error exit.
4. **`return Result{...}`** — early return inside a fallible
   function.

Order within a scope is **reverse declaration order**
(DROP-DEC-003). Across nested scopes, drops walk
**innermost-out**. Drops always run **before** `defer` blocks
(DROP-DEC-010).

## When Drop does *not* run

- **`exit N`** is a hard process exit. Drops are **skipped**
  (DROP-DEC-008). Same semantics as C++ `_Exit` or Rust
  `process::exit`. Reach for `pass` / `fail` if you need
  destructors to run.
- **`pass v` / `return v` where `v` is a bare identifier** —
  the binding named by `v` has its drop **skipped**; ownership
  moves to the caller (DROP-DEC-004 move semantics, bare-
  identifier only this cycle). Every other binding in scope
  still drops.
- **Bindings of unsupported shapes** — `wild T->:p = alloc(...)`
  (raw pointer-typed wild), per-`Handle<T>` auto-free, primitive
  bindings. These keep the explicit-cleanup contract.

## Comparison: Drop vs `failsafe` vs explicit free

| Mechanism      | Runs on                            | Order              | Opt-in             |
|----------------|------------------------------------|--------------------|--------------------|
| **Drop**       | fall-through, `pass`, `fail`, `return` | reverse decl    | `use "drop.npk".*;` |
| **`defer`**    | same as Drop                       | reverse `defer`    | always available   |
| **`failsafe`** | `fail` propagation in `main`       | top of `main` body | always available   |
| **Explicit free** | wherever you write it           | source order       | always available   |

Drop and `defer` are not mutually exclusive — a function may
have both. Drops run **first**, defers run **after**. Inside a
single scope, Drop touches only the four supported region
shapes; `defer` runs arbitrary user code.

## Chapters

1. [README](README.md) — this page.
2. [Surface](surface.md) — `impl:Drop:for:T`, the `drop` method
   shape, what's allowed and what isn't.
3. [Regions](regions.md) — Drop for `wild` / `wildx` / `Handle` /
   `Arena` / `JitFn`. The RAII-vs-explicit decision table.
4. [Ordering](ordering.md) — block reverse order, struct-field
   reverse order, `pass` / `fail` semantics, `failsafe`
   interaction, drop-before-defer.
5. [Pitfalls](pitfalls.md) — moves out of bindings (no drop),
   destructor failure, drop-during-drop, hard `exit`, the
   `Jit_compile_*` forcing-function gotcha.
6. [FAQ](faq.md) — RAII vs manual, why no NLL, why no Drop for
   primitives, can `drop` allocate? can it `pass`?

## See also

- [`stdlib/drop.npk`](https://github.com/alternative-intelligence-cp/nitpick/blob/dev-0.29.x/stdlib/drop.npk)
  — the opt-in source.
- [`guide/memory/wild.md`](../memory/wild.md) — explicit
  `wild` / `wildx` lifetime model that Drop layers on top of.
- [`guide/handles/`](../handles/README.md) — `HandleArena`
  lifetimes; Drop wraps `HandleArena.create()` with auto-destroy.
- [`guide/jit/`](../jit/README.md) — JIT pages; Drop wraps
  `Jit.compile_add_i32()` with auto-free.

## Validation snapshot (v0.29.7)

- **CTest:** 85/85 (84 at v0.29.6 close; +1 = `bug_tests_v0297`).
- **K core:** 151/151. **K proofs:** 11/11.
- **Bug regressions:** `bug279`–`bug310` covering surface,
  per-region codegen, early-exit drops, move semantics, and the
  before-defer ordering.
- **Decisions locked:** DROP-DEC-003 (reverse declaration order),
  DROP-DEC-004 (bare-identifier move), DROP-DEC-007 (per-region
  opt-in flags), DROP-DEC-008 (`exit` skips drops), DROP-DEC-010
  (drops before defers).
