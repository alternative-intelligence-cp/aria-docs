# Comptime

Nitpick **comptime** is the compile-time function evaluation (CTFE) layer.
Code marked `comptime` is reduced to a constant value during compilation,
so the resulting binary contains only the answer — never the computation.

## Contents

- [basic.md](basic.md) — `comptime(expr)`, `comptime { ... }`, `comptime func:`
- [ctfe.md](ctfe.md) — How CTFE works: const folding, recursion, memoization
- [intrinsics.md](intrinsics.md) — `@typeInfo`, `@sizeof`, `@alignof`, `@len`, `@fieldType`, `@offsetof`
- [generics.md](generics.md) — `T: type` parameters and type-level dispatch
- [macros.md](macros.md) — Comptime vs. macros (and how they cooperate)
- [limits.md](limits.md) — `limit<Rules>` short-circuit and `assert_static`
- [limitations.md](limitations.md) — What is **not** allowed at comptime
- [debug.md](debug.md) — Reading comptime diagnostics and call chains

## Overview

```aria
// comptime(expr) — evaluate any pure expression at compile time
fixed int32:KB = comptime(1024);
fixed int32:MB = comptime(KB * 1024);

// comptime { ... } — run an entire block at compile time
comptime {
    fixed int32:check = 2 + 2;
};

// comptime func: — declare a CTFE-only function
comptime func:square = int32(int32:n) { pass n * n; };
fixed int32:nine = comptime(square(3));
```

Comptime differs from macros:

| | Comptime | Macro |
|---|---|---|
| Operates on | Values & types | AST fragments |
| Type-checked? | Yes (full type system) | After expansion |
| Recursion | Bounded by CTFE budget | Bounded by depth guard |
| Side effects | None (pure) | Pure expansion only |
| Best for | Type reflection, pure math, table generation | Code patterns, syntactic sugar |

## When to Use Comptime

- Compute lookup tables, constants, masks, bit-widths at build time
- Inspect types via `@typeInfo`, `@sizeof`, `@fieldType`
- Drive generic algorithms with `T: type` parameters
- Fail the build on invalid configurations (`assert_static`)

See also: `META/NITPICK_COMPTIME/COMPTIME.md` for the design tracker.
