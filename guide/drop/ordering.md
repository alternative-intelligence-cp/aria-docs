# Ordering

The Drop ordering rules are all locked by codegen and verified
by IR-level scope-count assertions. This page is the operational
reference.

## Within a single block: reverse declaration order

```nitpick
use "drop.npk".*;

func:demo = NIL() {
    wildx int8->:a = wildx_alloc(16i64);  // declared first
    wildx int8->:b = wildx_alloc(16i64);  // declared second
    wildx int8->:c = wildx_alloc(16i64);  // declared third
    pass;
    // Drops emit: c, then b, then a (reverse).
};
```

Mechanism: each `wildx`/`wild`/`HandleArena`/`JitFn` binding
pushes a `DropEntry` onto the current scope's `drop_stack_`
slot. At scope exit, the emit walks the vector via reverse
iterator. DROP-DEC-003.

## Nested blocks: innermost-out

```nitpick
use "drop.npk".*;

func:demo = NIL() {
    wildx int8->:outer = wildx_alloc(16i64);
    {
        wildx int8->:inner = wildx_alloc(16i64);
        // Inner-block fall-through drops `inner` here.
    }
    // outer drops at function-block fall-through.
    pass;
};
```

For an **early exit** (`pass v`, `fail e`, `return ...`) from
the inner block, the codegen calls `executeAllScopeDrops()`,
which walks the whole `drop_stack_` from the innermost frame
outward. So `pass 0i32;` inside the inner block drops `inner`
first, then `outer`, then runs function defers, then returns.

## Across early exits

Every early-exit path goes through the same emit chain:

```
executeAllScopeDrops(moved_var_name)   // DROP-DEC-010 — first
        ↓
executeFunctionDefers()                // user defers — after
        ↓
ret / TBB-error return / hard exit
```

| Statement              | Drops? | Defers? | Move skip?                                  |
|------------------------|--------|---------|----------------------------------------------|
| fall-through `}`       | yes    | yes     | n/a                                          |
| `pass v;`              | yes    | yes     | if `v` is a bare identifier (DROP-DEC-004) |
| `pass;` (no value)     | yes    | yes     | no skip                                      |
| `fail e;`              | yes    | yes     | no skip (error path)                         |
| `return Result{...};`  | yes    | yes     | bare-identifier inside a `return v;` is **rejected by the parser** — return requires a Result literal |
| `exit N;`              | **no** | no      | n/a (DROP-DEC-008)                           |

## Drop vs `defer`: drop runs first

```nitpick
use "drop.npk".*;

func:demo = NIL() {
    defer { /* runs second */ }
    wildx int8->:p = wildx_alloc(16i64);   // drop runs first
    pass;
};
```

DROP-DEC-010 is a **static guarantee**: in every early-exit
handler, `executeAllScopeDrops()` is unconditionally called
**before** `executeFunctionDefers()`. The reasoning: Drop is
RAII destructor semantics — it tears down the resource the
value owns. `defer` is a user cleanup hook — it runs after the
value is gone.

## `failsafe` interaction

```nitpick
use "drop.npk".*;

func:fallible = int32() {
    wildx int8->:p = wildx_alloc(16i64);
    fail 7i32;   // (1) drop p   (2) run defers   (3) TBB-return 7
};

func:main = int32() {
    Result<int32>:r = fallible();
    if (r.is_error) { exit 7; }
    exit 0;
} failsafe {
    exit 1;
};
```

The order of operations for the `fallible()` call above:

1. Inside `fallible`, the `fail 7i32` site emits scope drops
   (here just `p`), then defers (none), then writes the TBB
   error and returns.
2. The caller (`main`) receives a `Result<int32>` with
   `is_error == true`. There is no implicit propagation; if
   `main` wants to bail it must check `r.is_error` itself.
3. `failsafe` only fires on **unhandled** `fail` propagation —
   i.e., a `fail` whose enclosing function never captured it
   into a `Result`. `main` cannot use `pass`/`fail` directly
   (Nitpick `main` is exit-code-only), so `failsafe` is reached
   via deep helpers that propagate up.

Note that **`main` itself cannot use `pass` or `fail`** in
Nitpick — only `exit N`. So to exercise early-exit drops you
write a helper and call it from `main`.

## Struct-field destruction order

For user types with `impl:Drop:for:T` (registered surface,
auto-dispatch not in v0.29.x — see [`surface.md`](surface.md)):
struct-field drop order is **reverse declaration order** on
destruction, matching construction order in reverse. This rule
is stated for forward compatibility; the four built-in regions
the compiler currently auto-drops do not have user fields with
their own Drop impls.

## Move semantics: bare-identifier transfer

```nitpick
use "drop.npk".*;

func:make_page = wildx int8->() {
    wildx int8->:p = wildx_alloc(64i64);
    pass p;        // drop for `p` is SKIPPED — ownership moves to caller
};

func:demo = NIL() {
    wildx int8->:page = make_page();
    // `page` is still owned here and drops at scope end.
    pass;
};
```

DROP-DEC-004 is **scoped to bare `IdentifierExpr` values** in
`pass v` and `return v`. Compound expressions (`pass arr[i]`,
`pass s.field`, `pass f()`) do **not** trigger move; they drop
everything in scope as usual. General-expression move was
intentionally deferred — it requires borrow-checker-level
analysis and risks UAF if mis-recognized.

This rule is what makes `Jit.compile_*` and `make_page`-style
factories work without explicit move syntax. Without it, the
factory would auto-free the page before handing it back.

## Validation

The runner [`tests/bugs/run_bug_tests_0297.sh`](https://github.com/alternative-intelligence-cp/nitpick/blob/dev-0.29.x/tests/bugs/run_bug_tests_0297.sh)
performs 20 assertions:

- **bug301** — `pass` drops 1 wildx.
- **bug302** — `fail` drops 1 wildx, caller checks `Result.is_error`.
- **bug303** — `exit` drops 0 (DROP-DEC-008).
- **bug304** — `pass p;` drops 0 (DROP-DEC-004 move).
- **bug305** — inner-then-outer drop count = 2.
- **bug306** — two siblings drop reverse, count = 2.
- **bug307** — drop before defer (same scope, one of each).
- **bug308** — `return Result{...}` drops 1.
- **bug309** — `fail` from inner block drops outer + inner = 2.
- **bug310** — failsafe path; runtime `Result` propagation,
  drop runs once.

CTest `bug_tests_v0297` (20/20 PASS) and the cycle's full
`ctest` (85/85) are the operational lock for the rules on this
page.
