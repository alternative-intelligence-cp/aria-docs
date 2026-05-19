# FAQ

Short answers to recurring questions about Drop. For the long
form, follow the chapter links.

## When should I import `drop.npk`?

When every binding of the supported region shapes in your
program follows simple stack-like lifetime — allocated in a
function, freed at scope exit. That's almost every short-lived
allocation. If your lifetime model is more complex (long-lived,
conditionally freed, transferred across module boundaries in
non-trivial ways), keep the explicit-free contract.

## RAII vs manual `npk_free` — which is "the Nitpick way"?

Both are. Drop is **opt-in additive**; manual free is **always
available**. The compiler does not force a choice. The
trade-off:

- **Drop**: shorter code, less typing, harder to leak.
- **Manual**: explicit control, no surprises about *when* a
  free happens, works in patterns Drop's recognizer rejects.

In the stdlib, most internal helpers stay manual because they
already had explicit-free contracts before v0.29.x and the
cycle is intentionally non-breaking. New code is encouraged
to import `drop.npk`.

## Why no Non-Lexical Lifetimes (NLL)?

Drops fire at **scope end**, not at "last use". This is a
deliberate v0.29.x design choice:

- Last-use detection requires control-flow-graph liveness
  analysis at codegen time — expensive and complex.
- Scope-based is **predictable**: you can tell by reading the
  source where each drop fires.
- Borrow-checker rules (ARIA-014 / 022 / 015) already cover
  the "use after free" case for non-RAII paths. Letting NLL
  drop bindings early would interact with those rules in
  ways we wanted to bake into a separate cycle.

If you need a binding to die early, introduce a sub-block:

```nitpick
{
    wildx int8->:scratch = wildx_alloc(64i64);
    // ...use scratch...
}   // scratch drops here
// ...continue without scratch...
```

## Why no Drop for primitives?

Primitives (`int32`, `bool`, `flt64`, …) have no resource to
release. They live on the stack frame and the frame
disappears at function return. There is nothing to dispatch.

Sema rejects `impl:Drop:for:int32` with the "type does not
have a struct decl" check, by design.

## Can a `drop` method allocate?

It can — the compiler does not statically forbid it — but
**don't**. A `drop` method that allocates risks running the
allocator inside the cleanup of a previous allocation, which
in pathological cases recurses through the `drop_stack_`. The
v0.29.x built-in sentinels all have empty bodies (`pass;`).
For user types (surface-only this cycle), keep `drop`
side-effect-free or restricted to releasing other owned
resources.

## Can a `drop` method `pass` or `fail`?

DROP-DEC-009: destructor failure is **undefined behaviour**
this cycle. `pass;` (no value) is fine and is what the
built-in sentinels use. `pass v;` and `fail e;` have
unspecified semantics.

## Why does `Jit.compile_add_i32()` return a `wildx int8->` without freeing it?

Because of DROP-DEC-004 (bare-identifier move). Inside
`Jit_compile_add_i32`, the function does:

```nitpick
wildx int8->:page = wildx_alloc(64i64);
// ...write bytes, seal...
pass page;   // drop for `page` is SKIPPED, ownership moves to caller
```

The caller binds the returned page to its own variable, which
becomes the new RAII binding (re-recognized via the JitFn
shape). Without bare-identifier move, the page would be freed
inside `Jit_compile_add_i32` and the caller would receive a
dangling pointer.

This is also the entire reason any factory function in Nitpick
works under RAII.

## What's the difference between `defer` and `drop`?

| Aspect           | `defer`                  | Drop                        |
|------------------|--------------------------|-----------------------------|
| Surface          | `defer { ... }`          | `use "drop.npk".*;`         |
| Per-binding      | no — block-scoped        | yes — one drop per binding  |
| What runs        | arbitrary user code      | hand-emitted free for region |
| Ordering         | reverse `defer` order    | reverse declaration order   |
| Vs. each other   | runs **after** Drop      | runs **before** defers      |
| Skipped by `exit`| yes                      | yes (DROP-DEC-008)           |

Use `defer` for ad-hoc cleanup (closing a file, logging a
metric). Use Drop for owned resources in the four supported
regions. Mix freely when you want both.

## Does Drop replace `failsafe`?

No. `failsafe` is the top-level `fail`-propagation hook in
`main`. Drop runs the destructor at the failing scope; the
TBB-error then propagates up. If it bubbles all the way out of
the call chain without being captured into a `Result`,
`failsafe` is the last stop before process exit. They compose.

## Why is there a separate flag for each region?

DROP-DEC-007. The four flags
(`wild_raii_enabled`, `wildx_raii_enabled`,
`handle_arena_raii_enabled`, `jit_fn_raii_enabled`) are
independent **internally**, even though `drop.npk` flips all
four with one import. This leaves room for a future cycle to
ship per-region opt-in (e.g. `use "drop_handle.npk".*;`)
without changing the IR-generator gates.

## What happens if I forget `use "drop.npk".*;`?

Nothing automatic. The bindings work exactly as in v0.29.2
and earlier — you call `npk_free` / `wildx_free` /
`HandleArena.destroy` / `Jit.free` yourself, and `ARIA-014`
fires at scope exit if you forget.

## Does Drop work across FFI?

No. The `drop.npk` recognizers see only Nitpick-emitted
allocator calls. A pointer that crosses an `extern` boundary
is opaque; Drop does not auto-emit cleanup for resources
allocated by C code, nor does it interfere with explicit
C-side cleanup. The FFI contract is unchanged.

## Will Drop ever get NLL / move-for-arbitrary-expressions / per-binding opt-out?

Probably yes, in separate cycles. The v0.29.x cycle deliberately
shipped the minimum useful surface and left these gaps:

- **NLL** — predicate on liveness; needs CFG analysis.
- **General-expression move** — needs borrow-checker integration
  to be safe.
- **Per-binding `raw` opt-out** — needs the corresponding
  `ARIA-022` extension for double-free diagnostics first.
- **User-defined `drop` method auto-dispatch** — needs the
  registry hooked into the IR generator's scope-exit emit.
- **Drop-during-Drop policy** + **destructor failure policy** —
  needs design work and K-semantics modeling.

See [`META/NITPICK/ROADMAP/0.29/PLAN.md`](https://github.com/alternative-intelligence-cp/nitpick/blob/dev-0.29.x/META/NITPICK/ROADMAP/0.29/PLAN.md)
for the v0.29.x scope envelope.

## Where do I look for examples?

- `tests/bugs/bug285`–`bug310` — every code path Drop touches.
- `stdlib/drop.npk` — the canonical opt-in source.
- `stdlib/jit.npk` — the canonical RAII-aware factory.
- This guide's [`regions.md`](regions.md) — the four binding
  shapes side by side.
