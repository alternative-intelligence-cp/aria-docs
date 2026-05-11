# Macro Hygiene

Nitpick macros are **hygienic**: variables introduced inside a macro body
do not conflict with variables at the call site, and call-site variables
do not accidentally shadow macro-internal names.

## The Problem Without Hygiene

Without hygiene, a macro that declares a temporary variable could clash
with a variable of the same name at the call site:

```aria
// hypothetical unhygienic macro (NOT what Nitpick does)
macro:swap_bad = (a, b) {
    int32:tmp = a;   // if caller also has 'tmp', this would clash
    a = b;
    b = tmp;
};
```

## How Nitpick Handles It

The compiler performs **AST cloning with gensym** before substituting the
macro body at each call site. Each internal binding gets a fresh unique
name (e.g., `tmp` → `__macro_tmp_42`). This happens automatically — you
do not need to do anything special.

```aria
macro:swap = (a, b) {
    int32:tmp = a;   // internally renamed to a fresh name at each call site
    a = b;
    b = tmp;
};

func:main = int32() {
    int32:x = 1i32;
    int32:y = 2i32;
    int32:tmp = 99i32;   // NOT affected by macro's internal 'tmp'
    swap!(x, y);
    println(`x=&{x} y=&{y} tmp=&{tmp}`);  // x=2 y=1 tmp=99
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

## What Hygiene Covers

- All variable bindings declared inside the macro body (`int32:tmp`, `string:buf`, etc.)
- Loop index variables introduced by `loop()` inside the macro

## What Hygiene Does Not Cover

- **Parameters** — parameter names are substituted literally (they refer to
  call-site expressions, not new bindings)
- **Global/module-level names** — macros that reference module functions or
  global constants do so by their declared names; hygiene does not rename these

## Inspecting Hygienic Expansion

Use `--expand-macros` to see the gensym-renamed variables:

```
$ npkc myfile.aria --expand-macros
// macro expansion at myfile.aria:10:5
// call:     swap!(Ident(x), Ident(y))
// expanded: Block([VarDecl(__macro_tmp_1, ...)])
```

See [debug.md](debug.md) for full `--expand-macros` documentation.
