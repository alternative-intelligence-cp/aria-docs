# JIT

The JIT chapter documents how Nitpick programs produce, install,
call, and tear down **executable code at runtime** using the
`wildx` W^X heap and the `stdlib/jit.npk` helper.

This is the *thinnest possible* surface: this cycle ships one
hand-assembled signature (`int32 add(int32, int32)`). It is enough
to prove the loop closes end-to-end. Generic code emission (a real
assembler, multiple signatures, register allocation) is wishlist
for a future cycle and is explicitly *not* documented here as if it
were available.

## When to reach for JIT

| You want…                                              | Use         |
|--------------------------------------------------------|-------------|
| Inline a hot path that's known only at runtime         | `Jit`       |
| Specialise a function for runtime constants            | `Jit`       |
| Trampolines / shims into FFI you don't have at AOT     | `Jit`       |
| Anything compile-time-known                            | regular AOT |
| Plain data heap with manual free                       | `wild`      |
| Lots of small allocs freed together                    | `Handle`    |
| Long-lived references with cycles                      | `gc`        |

JIT is for **code**, not data. Reach for it only when you need the
CPU to execute bytes you produced this run. If you're writing a
specialising interpreter, a regex compiler, a small DSL, or an FFI
shim generator, this is the door. For everything else, the AOT
compiler is already doing it better.

## When *not* to reach for JIT

- **You're trying to escape the borrow checker.** That's what
  [`gc`](../memory/gc.md) is for.
- **You need executable memory for, e.g., a buffer the GPU will
  fetch.** That's a `wild` allocation with a different `mmap`
  flag set; not what `wildx` does.
- **You're on a platform other than x86-64 Linux.** ARM64 is
  documented as future work; this cycle has no code for it
  (`JIT-DEC-002`).
- **You want a JIT *backend* for the Nitpick compiler itself.**
  That's a separate, much larger project. This chapter is about
  user-program JITs running *inside* an already-AOT-compiled
  Nitpick binary.

## Chapters

1. [README](README.md) — this page.
2. [`wildx` lifecycle](wildx.md) — `alloc` → write → `seal` → call
   → `free`, viewed through the JIT lens.
3. [The `jit.npk` helper](helper.md) — `Type:Jit` walkthrough,
   one-signature surface, ownership model.
4. [Safety](safety.md) — what's exempt from the regular checks,
   what's still enforced, what can still go wrong.
5. [FAQ](faq.md) — recurring questions ("can I patch in place?",
   "what about ASLR / NX?", "what's the ARM64 story?").

## See also

- [`guide/memory/wild.md`](../memory/wild.md#wildx--manual-heap-executable-wx)
  — the runtime W^X lifecycle in memory-model terms.
- [`guide/handles/`](../handles/README.md) — for the *data*
  arena counterpart.
- [`guide/drop/`](../drop/README.md) — the v0.29.6 opt-in RAII
  layer that auto-frees `Jit.compile_*` pages at scope end
  (independent flag from generic `wildx` RAII per DROP-DEC-007).

## Validation

- End-to-end smoke: `tests/bugs/bug264_jit_add_i32_smoke.npk`
  (CTest `bug_tests_v0281`, x86-64-only via `SKIP_RETURN_CODE 77`).
- Helper surface: `bug265`, `bug266` (CTest `bug_tests_v0282`).
- W^X enforcement: `bug246`–`bug248` (CTest from v0.27.5, predates
  this chapter).
