# Surface

This page describes the source-level surface for Drop: what
you write in a `.npk` file to opt in, what shapes are accepted
by the parser and sema, and what is intentionally *not* on the
surface this cycle.

## Opt-in: import `drop.npk`

```nitpick
use "drop.npk".*;
```

That's it. There is no per-binding annotation. The import
brings four sentinel structs and their `Drop` impls into scope;
the compiler keys on the presence of those impls in the
program's Drop registry and flips per-region flags
(`wild_raii_enabled`, `wildx_raii_enabled`,
`handle_arena_raii_enabled`, `jit_fn_raii_enabled`).

Without the import, the four flags stay off and the compiler
emits no automatic drops. Existing programs are unaffected.

## Defining a `Drop` impl

```nitpick
struct:Holder = {
    int64:value,
};

impl:Drop:for:Holder = {
    func:drop = NIL($m Holder:self) {
        pass;
    };
};
```

Requirements (enforced by sema in v0.29.1):

- The trait name **must** be `Drop`. No other trait can be
  declared with this shape.
- The type must already exist (no lazy fabrication via the
  primitive-type path — sema rejects unknown type names with
  a clean error).
- Exactly one method, named `drop`.
- Signature: returns `NIL`, takes a single `$m T:self` borrow.
- Duplicate `impl:Drop:for:T` declarations are rejected.

The compiler registers `T_drop` (mangled) in the Drop registry.

## What the body can do this cycle

The four `NitpickXxxRaii` sentinel `drop` methods in
`stdlib/drop.npk` all have a body of just `pass;`. That is
deliberate: the compiler does **not** call the sentinel `drop`
method directly. The sentinel exists only to flip the per-region
flag; the actual cleanup is the hand-emitted
`npk_free` / `npk_wildx_free` / `npk_handle_arena_destroy` call.

For **user types** with `impl:Drop:for:T`, the surface is in
place (parser, sema, registry) but **codegen for user-defined
drop dispatch is out of scope for v0.29.x**. Writing a real
body in a user `drop` method will not produce auto-dispatch
this cycle; the user must continue to call `T_drop` explicitly
if they want the body to run. The four built-in regions are
the only auto-cleanup this cycle ships.

## What is *not* on the surface

- **`impl:Drop:for:` on primitives** (`int32`, `bool`, …).
  Sema rejects via the "type does not have a struct decl"
  check. See [`faq.md`](faq.md#why-no-drop-for-primitives).
- **A user-callable `drop x` expression** that runs a type's
  Drop impl. The keyword `drop` is reserved as a unary
  value-sink (`drop x;` evaluates `x` and discards the
  result, bypassing TBB Result handling — predates v0.29.x),
  and `c.drop()` UFCS sugar is parser-accepted but does not
  dispatch through the Drop registry this cycle.
- **`raw` overrides of automatic Drop.** Per
  [`drop.npk`](https://github.com/alternative-intelligence-cp/nitpick/blob/dev-0.29.x/stdlib/drop.npk),
  the planned opt-out spelling is `raw npk_free(p);` (manual
  free that suppresses the auto-drop on the same binding).
  v0.29.x ships the auto-emit but **does not yet diagnose**
  the explicit-free + auto-drop collision; that detection is
  the v0.29.x successor cycle's work. In v0.29.x, writing
  `npk_free(p);` for a binding the compiler also auto-drops
  is a **double-free** — don't do it.
- **`impl:Drop` on generic types.** Sema-level only; generics
  are tracked in a separate cycle.

## Parser interaction with the `drop` keyword

`drop` is both:

1. A unary-prefix value-sink expression: `drop expr;` (existing,
   pre-v0.29.x).
2. A method name in the `impl:Drop` shape: `func:drop = NIL($m T:self) { ... };`.

DROP-DEC-007a (v0.29.0) resolved the parser collision by
accepting `TOKEN_KW_DROP` as a method-name identifier inside
`parseFuncDecl` and as a member-name identifier inside
`parseMemberExpression`. The existing `drop <expr>` unary form
is unchanged.

## Validation

- bug279–bug284 (v0.29.1) — surface acceptance and rejection.
- The four `NitpickXxxRaii` impls in
  [`stdlib/drop.npk`](https://github.com/alternative-intelligence-cp/nitpick/blob/dev-0.29.x/stdlib/drop.npk)
  are themselves the canonical examples.
