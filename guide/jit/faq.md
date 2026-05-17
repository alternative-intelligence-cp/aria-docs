# JIT FAQ

## Can I patch generated code in place?

No — not in v0.28.x. Once a page is sealed, the MMU enforces
`PROT_EXEC`-only and any write traps with `SIGSEGV`. The
v0.28.x answer is: allocate a new page, write the new bytes,
seal it, swap your `JitFn` over, and free the old page.

Patch-in-place would require a transient `WRITABLE` state — a
fourth state in the `wildx` state machine plus an `unseal`
operation. That's a real design discussion (do you re-hash on
re-seal? do you double-map the page R-X and R-W as some real
JITs do?) and it isn't in scope this cycle.

## Is ARM64 supported?

No, this cycle is x86-64 Linux only (`JIT-DEC-002`). The
`jit_smoke.cpp` byte sequence is x86-64 SysV-AMD64, and the
runtime does not emit the cache-coherence barriers (`isb` / `dc
cvau` / `ic ivau`) that ARM64 requires between writing
instructions and executing them.

The `bug264` test fixture self-skips on non-x86-64 hosts with
`SKIP_RETURN_CODE 77`, so the CTest suite stays green on ARM
boxes without lying about coverage.

A future cycle could add ARM64 in one of two ways: (a) a
parallel `jit_arm64.cpp` with the equivalent `add(i32, i32)`
sequence and the barrier dance, or (b) a slice / signature-tag
API that decouples the byte stream from the runtime entirely
and lets user code emit whatever it wants. (b) is the right
long-term shape but waits on slice FFI.

## What about ASLR / NX / PaX / SELinux / `noexec` mounts?

- **ASLR** — `wildx_alloc` adds an ASLR jitter base in the
  guard struct and stores it for integrity checks. The
  allocation itself is `mmap`-placed by the kernel, so it gets
  the same ASLR treatment any anonymous `mmap` does.
- **NX bit** — fully respected. The page is `PROT_READ |
  PROT_WRITE` (no `PROT_EXEC`) until `wildx_seal` flips it.
  After seal, it is `PROT_READ | PROT_EXEC` (no `PROT_WRITE`).
  This is the entire purpose of the W^X discipline.
- **PaX MPROTECT** — incompatible. PaX explicitly forbids the
  `PROT_WRITE → PROT_EXEC` transition that `wildx_seal`
  performs. There is no workaround inside the language; this
  is the kernel's call. If you need JIT on a PaX-hardened
  system, you need to grant the process the
  `PAX_MPROTECT` exemption (or use a shared-memory file
  double-mapped R-W and R-X, which is a different design).
- **SELinux `execmem` / `execheap`** — same situation. The
  runtime needs the policy to allow `mprotect` of anonymous
  memory to `PROT_EXEC`. Audit logs will tell you if it's
  being denied.
- **`noexec` mounts** — not relevant. `wildx` uses anonymous
  `mmap`, not a file mapping; mount flags don't apply.

## Why is the only signature `int32 add(int32, int32)`?

Two reasons:

1. **The runtime helper is hand-written.** Generalising it
   requires either a real x86-64 assembler in the runtime or a
   typed slice FFI so user code can pass `&[u8]` plus a
   signature tag to a `Jit.compile(bytes, sig)`. Neither
   exists in v0.28.x.
2. **It's the smallest possible end-to-end proof.** The whole
   point of v0.28.x is to land the W^X heap, the helper, and
   the cookbook in one coherent slice. Shipping `add(i32,i32)`
   exercises every state transition; shipping `add(i32,i32)`
   *plus* `mul(i32,i32)` would not exercise anything new.

The cycle that lands slice FFI is the cycle that will widen
the signature set.

## How does this compare to `wild` for "I need executable bytes"?

`wild` allocates with `PROT_READ | PROT_WRITE` and never adds
`PROT_EXEC`. Calling into a `wild` allocation on a system with
NX enforced (every modern desktop / server) will fault at the
first instruction. `wild` is for *data*; `wildx` is for *code*.

## Can I share a `JitFn` across threads?

The pointer is safe to copy and pass around. Concurrent calls
through the same `JitFn` are safe iff the code in the page is
reentrant — which the v0.28.x `add(i32, i32)` is, since it
touches no memory. `Jit.free` mutates the W^X registry under a
mutex, but as with any explicit-ownership type, you still need
to make sure no other thread is calling through the pointer
when you free it.

For workloads that *generate* a `JitFn` per thread (which is
what a typical specialising JIT does), there's no sharing
problem to begin with.

## Why isn't this hooked into `failsafe`?

The same reason `wild` and `wildx` aren't: failsafe is the
unwind path on panic / OOM, and unwinding through arbitrary
JIT'd code is a much harder problem than unwinding through
known Nitpick frames. The v0.28.x choice is explicit
ownership: you allocate, you free, leak detection catches the
forgetful path.

RAII / drop semantics on `wildx` bindings is wishlist; when it
lands, it will arrive uniformly for `wild`, `wildx`, and
`Handle`.

## What about `perf` / debugger support?

Out of scope this cycle. A JIT that wants `perf` to symbolicate
its frames needs to write `/tmp/perf-<pid>.map` entries or use
`perf record --jit`. Nothing in Nitpick does this for you.
Adding it is a one-page runtime change but is not part of
v0.28.x.

## What about the integrity hash stored on `seal`?

`wildx_seal` records an FNV-1a hash of the page contents.
Today, that hash is bookkeeping — nothing in the runtime
actively re-checks it. A future cycle may add a verify-on-call
mode (off by default, on under a runtime flag for fuzzing /
hardening) that re-hashes before each call and panics on
mismatch. The hook is there; the policy is not yet.

## Where do I go for more?

- [`README`](README.md) — when to reach for JIT.
- [`wildx.md`](wildx.md) — the lifecycle in detail.
- [`helper.md`](helper.md) — the `Type:Jit` surface.
- [`safety.md`](safety.md) — exemptions and enforced rules.
- [`guide/memory/wild.md`](../memory/wild.md) — `wild` and
  `wildx` from the memory-model angle.
