# CTFE Internals

This chapter covers how the Nitpick compiler executes `comptime` code:
the **constant evaluator**, its budget, and what guarantees it gives you.

## Pipeline

```
        ┌──────────────┐    ┌────────────┐    ┌──────────┐
parse → │ type checker │ →  │   comptime │ →  │ codegen  │
        │  (markers)   │    │ const eval │    │ (literal)│
        └──────────────┘    └────────────┘    └──────────┘
```

The type checker tags every `comptime(...)` node with its expected type,
then the **const evaluator** (`src/frontend/sema/const_evaluator.cpp`)
recursively reduces the AST to a `ComptimeValue`. The result is a literal
inserted back into the IR.

## What CTFE Can Do

- Integer & float arithmetic, comparisons, boolean logic
- String concatenation and equality (`+`, `==`, `!=`)
- `if/else`, `pick { ... }`, `loop(start, end, step) { ... }`
- Mutable locals (CTFE has its own register file)
- Function calls into `comptime func:` and other pure helpers
- Struct construction, struct update syntax, field reads
- Array literals, indexing, `@len`, `@sizeof`, `@alignof`, `@typeInfo`

## Recursion and Memoization

Each call to a `comptime func:` decrements the **CTFE budget** (default
≈ 100k steps). Pure calls with the same argument tuple are memoized
within a compilation unit, so repeated lookups (`fib(20)`) collapse to one
evaluation.

```aria
comptime func:fib = int32(int32:n) {
    if (n < 2) { pass n; }
    pass fib(n - 1) + fib(n - 2);
};

fixed int32:f10 = comptime(fib(10));   // 55, computed once
fixed int32:f10b = comptime(fib(10));  // memoized, no re-evaluation
```

Exceeding the budget produces a diagnostic; raise it with the (planned)
`--comptime-budget` flag.

## Mutable Locals

Inside a `comptime { ... }` or `comptime func:`, you may declare and
mutate locals exactly as at run time. The mutations are confined to CTFE
and never leak into the runtime program.

```aria
fixed int32:total = comptime({
    int32:s = 0;
    loop(1, 11, 1) { s = s + $; }
    pass s;
});  // 55
```

## Determinism

CTFE is fully deterministic. The same source, compiled with the same
compiler version on any host, produces the same comptime values. There is
no clock, no RNG, no environment access — see [limitations.md](limitations.md).
