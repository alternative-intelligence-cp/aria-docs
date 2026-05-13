# Borrow Cookbook

This subsection of the guide is a focused, example-driven reference for
Nitpick's borrow checker as it stands at the close of the v0.25.x cycle. It
complements the higher-level material in
[`memory_model/borrow.md`](../memory_model/borrow.md) by walking through the
concrete patterns the checker accepts, the diagnostics it produces, and the
edge cases that were locked in across the BORROW-001 ... BORROW-014 work.

## Chapters

1. [Basics](basic.md) — `$$i` / `$$m`, the 1-mut XOR N-immut rule,
   the `failsafe` path, and what "release" means.
2. [Paths](paths.md) — field paths, nested fields, and how the checker
   separates `box.leaf.x` from `box.leaf.y`.
3. [Arrays](arrays.md) — literal vs. dynamic index borrows, the `arr[*]`
   collapse, and the rules that fall out of it.
4. [Structs](structs.md) — borrowing through struct values, struct update
   syntax, pick patterns, and method calls.
5. [Call sites](callsites.md) — passing borrows to functions, multiple `$$m`
   parameters, writeback, and the receiver rules for `$$m` self methods.
6. [Two-phase patterns](two_phase.md) — scope-release, sequenced borrows,
   and the limited two-phase shapes the checker recognises today.
7. [Async](async.md) — borrows that live inside `async` frames, what gets
   released on completion, and the safe shapes around `await`.
8. [Diagnostics](diagnostics.md) — the borrow diagnostic codes
   (`ARIA-023`, `ARIA-026`, ...), what they mean, and how to fix the most
   common cases.

Each chapter is short and self-contained — read whichever one matches the
problem in front of you.

## Conventions used in these chapters

- Examples are minimal `func:main` programs that compile and run with `npkc`.
- "release" means the loan is dropped because the borrower goes out of scope.
- "host" means the variable or path the loan refers to.
- Diagnostics are quoted as the compiler emits them, with the `ARIA-NNN` code.
- Where K semantics is referenced, the corresponding `tests/core/` test number
  is given in parentheses, e.g. `(K test 091)`.

## Validation status (v0.25.7)

- Compiler CTest: 53/53.
- K core tests: 145/145.
- K proofs: 10/10 (path-disjointness and field-alias theorems hold).
- Borrow-related bug regressions: 105 (bugs 040 ... 204).
