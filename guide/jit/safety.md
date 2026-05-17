# Safety

JIT is the part of Nitpick where the language hands you the
keys and asks you to drive carefully. This page is the honest
list of which safety nets are still under you, and which ones
are not.

## What JIT is *exempt from*

### The borrow checker on the bytes themselves

The bytes inside a `wildx` page are opaque to the borrow checker.
There is no liveness analysis of what's in there, no aliasing
analysis on the registers the code uses, no checking that the
machine code respects the SysV calling convention. The compiler
trusts that what you `memcpy`'d into the page is what you say
it is.

The borrow checker *does* still see the `wildx int8->` pointer
itself — it knows the page exists, that it was allocated, and
that it must be freed. What it doesn't see is what the page does
when called.

### The garbage collector

The GC marker does not follow pointers into `wildx` pages. From
the GC's perspective, a `wildx` page is just another `wild`
allocation: opaque memory the user owns. If your JIT'd code
holds a reference to a `gc` value across a call, you must root
it yourself with `npk_shadow_stack_add_root` — exactly as you
would from any other unmanaged region (see
[`guide/memory/interop.md`](../memory/interop.md)).

### Pin semantics

`pin` is a borrow-checker concept tied to Aria-visible bindings.
A function compiled into a `wildx` page is not an Aria binding;
it has no pin frame, no shadow stack entry, and no participation
in the K-semantics model. Anything the JIT'd code mutates is
mutated by the CPU directly, outside the borrow rule system.

## What JIT is *still subject to*

### W^X discipline

The hardware enforces W^X. A sealed page that is written to
traps with `SIGSEGV` from the MMU, not from a runtime check.
You cannot opt out of this without going to the OS for a fresh
mapping with both `PROT_WRITE` and `PROT_EXEC` — and if you do
that, you're no longer in `wildx` territory and the runtime
will not help you.

### Leak detection (`ARIA-014`)

Every `wildx_alloc` / `Jit.compile_*` must be paired with a
matching release in the same scope. The borrow checker fires
`ARIA-014` at scope exit otherwise. Regression: `bug248`.

### Double-free / use-after-free (`ARIA-022`, `ARIA-015`)

`wildx_free` and `Jit.free` are both registered as deallocators.
A second free of the same binding is rejected at compile time
(`ARIA-022`, regression `bug247`). Reading a freed pointer is
rejected at compile time (`ARIA-015`).

The runtime `-1` return on a double-free is defense-in-depth
for pointers smuggled through function boundaries the borrow
checker cannot see across — it should never fire in
well-typed Nitpick code.

### State-machine integrity

The W^X state machine itself is runtime-enforced:

- `wildx_seal` on an already-sealed page returns `-1`
  (regression `bug246`).
- `wildx_seal` on a freed page returns `-1`.
- `wildx_free` on a foreign or freed page returns `-1`.

These return paths exist because the borrow checker cannot
always know which state a pointer is in across complex control
flow. They are the runtime's last line of defense.

### Quota accounting

The W^X allocator tracks total bytes allocated and rejects
allocations that would exceed a process-wide quota. Code that
allocates pages in a loop without freeing will eventually start
getting NULL back from `wildx_alloc`.

## What can still go wrong

### Signature mismatch

`Jit.call_i32_i32` `memcpy`s the page pointer into a typed
function pointer of shape `int (*)(int, int)`. If the bytes in
the page implement a different signature (different argument
count, different register usage, different return type), the
call lies to the CPU about the calling convention. Behaviour
is undefined.

In v0.28.x this is vacuously safe — there's exactly one
signature on the surface, and it matches. Once
`Jit.compile(bytes, sig)` lands, the helper will refuse to
hand out a typed `JitFn` for a signature the runtime cannot
prove the bytes implement.

### Bad bytes

If the bytes you write are not valid x86-64 machine code, the
CPU may execute them anyway — possibly with arbitrary side
effects, possibly to a `SIGILL`, possibly to a `SIGSEGV`,
possibly to a remote code execution. Nothing in Nitpick
validates the byte stream.

This is the entire reason the v0.28.x surface only exposes
`compile_add_i32()`: those 5 bytes are pinned by the runtime,
hand-verified, and not user-supplied. Letting user bytes flow
through `wildx_seal` is a real engineering project: at minimum,
a disassembler-validator that rejects bytes that escape the
page, touch forbidden registers, or call un-allowed targets.

### Spectre / Meltdown / branch-prediction leaks

Outside Nitpick's scope. If you're building a sandbox, you need
the same CPU-vulnerability mitigations every other JIT needs:
indirect-branch barriers, retpolines, RSB stuffing, etc.
Nitpick does not insert these for you and does not pretend to.

### Cross-thread sharing of `wildx` pointers

The W^X *registry* is mutex-serialised; the *bytes in a sealed
page* are read-only by hardware, so concurrent calls are safe
as long as the JIT'd code itself is reentrant (the v0.28.x
`add` page is). The bookkeeping is safe to share; the
correctness of concurrent calls depends entirely on what the
code does.

## Where to draw the line

JIT is a sharp tool. The Nitpick design choice here is
explicit: `wildx` exists to make runtime code generation
*possible*, not to make it *safe in the way a regular Nitpick
binding is safe*. If you find yourself wanting more guarantees
than this page lists, the right move is to express your problem
in regular Nitpick (where the checker can help) and JIT only
the part that actually has to be runtime-generated.

## Validation

- `bug246` — sealing twice is rejected.
- `bug247` — double `wildx_free` rejected at compile time.
- `bug248` — leaked `wildx` page rejected at compile time.
- `bug264`–`bug266` — happy path under all the above
  constraints.
