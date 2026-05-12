# Debugging Comptime

When a `comptime(...)` expression fails, the compiler emits a diagnostic
anchored at the source line of the call. This chapter explains how to
read those messages and recover useful information.

## Anatomy of a Comptime Diagnostic

```
error: comptime evaluation failed
  --> src/util.npk:42:18
   |
42 |     fixed int32:p = comptime(check(70000));
   |                     ^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: limit<Port> rule violated: $ <= 65535 with $ = 70000
   = note: comptime call chain:
       check (src/util.npk:31)
       └─ violates limit<Port>
```

Three things to look for:

1. **The location of the `comptime(...)` call** — the caret marks the
   *outer* expression, not the inner failure.
2. **The reason** (`= note:`) — overflow, divide-by-zero, rule violation,
   budget exhaustion, forbidden operation, etc.
3. **The call chain** — which `comptime func:` was on the stack when the
   error fired.

## Common Errors

| Diagnostic | Cause | Fix |
|---|---|---|
| `comptime budget exceeded` | Recursion or loop ran past the step cap | Add a base case; refactor to iterative |
| `division by zero in comptime` | `a / 0` reduced at comptime | Guard with `if (b != 0) { ... }` |
| `integer overflow in comptime` | tbb-typed value crossed its rail | Widen to `int64` or relax the type |
| `forbidden at comptime: <op>` | I/O, GC, extern, etc. (see [limitations.md](limitations.md)) | Move op to runtime |
| `unbound name in comptime` | Referenced a runtime-only binding | Pass the value in, or recompute |

## Tracing Comptime Evaluation

The compiler accepts (planned) `--trace-comptime=<funcname>` to dump the
intermediate values of a specific `comptime func:` while it executes.
Use this when a result is *wrong* (rather than failing).

## Sanity Checks

When you suspect comptime is silently producing the wrong constant:

```aria
// Surface the value through a static assertion
assert_static(comptime(myfn(7)) == 49, "myfn(7) regression");

// Or print it from main(); the compiler folds it to a literal
println(`&{comptime(myfn(7))}`);
```

The latter is the quickest way to confirm what value was actually folded
into the binary.

## Filing a Bug

If a comptime expression appears to fail incorrectly:

1. Reduce the failing call to a `tests/bugs/bugNNN_*.npk` file.
2. Run the compiler with `--dump-ir` and inspect the failed IR node.
3. File against `META/NITPICK/BUGS/` with the reduction.
