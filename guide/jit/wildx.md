# The `wildx` lifecycle (JIT view)

`wildx` is the same allocator the [memory chapter](../memory/wild.md)
covers, but JIT pulls on every stage of its W^X state machine.
This page walks the lifecycle from the JIT angle: what each step
is for, what fails if you skip it, and what the runtime
guarantees in return.

## States

```
UNINITIALIZED ──wildx_alloc──▶ WRITABLE ──wildx_seal──▶ EXECUTABLE ──wildx_free──▶ FREED
                                  │                          │
                                  │                          └── any write here
                                  │                              traps with SIGSEGV
                                  │                              (PROT_EXEC only)
                                  │
                                  └── safe to memcpy machine bytes;
                                      no PROT_EXEC bit set yet
```

A page is keyed by its writable address in a process-wide
registry. The transitions are one-way: you cannot un-seal, you
cannot re-arm a freed page, and a foreign pointer (one that did
not come from `wildx_alloc`) is rejected by every operation.

## Step 1 — `wildx_alloc(size)`

```aria
wildx int8->:page = wildx_alloc(64);
```

`mmap`s an anonymous, page-aligned region with `PROT_READ |
PROT_WRITE` (no `PROT_EXEC`). Registers the writable address in
the W^X registry along with an ASLR jitter base, an FNV-1a code
hash slot (filled in at seal time), and the quota accounting.

- **Failure:** returns `0` (NULL) on `mmap` failure or quota
  exhaustion. Callers should NULL-check exactly like a plain
  `wild alloc()`.
- **Static checks:** the borrow checker treats `wildx_alloc` as
  a `wild` allocator. `ARIA-014` will fire if you forget to pair
  it with `wildx_free` in the same scope (regression: `bug248`).

## Step 2 — write machine bytes

```aria
// In Nitpick, you'll usually call an extern helper:
fixed int32:write_rc = npk_jit_install_add_i32(page);
```

This is where the JIT meets the metal. In v0.28.x we hand off to
[`npk_jit_install_add_i32`](https://github.com/alternative-intelligence-cp/nitpick/blob/main/src/runtime/assembler/jit_smoke.cpp#L20),
which `memcpy`s 5 bytes of x86-64 SysV-AMD64 ABI machine code:

```text
89 f8     mov  eax, edi      ; ret = a
01 f0     add  eax, esi      ; ret += b
c3        ret
```

The writable mapping makes this a plain `memcpy`. No syscalls,
no privilege boundary; just bytes into a writable buffer. Writes
that overrun the page allocation are caught by the OS — pages
are page-granular, and the next page is unmapped.

## Step 3 — `wildx_seal(page)`

```aria
fixed int32:seal_rc = wildx_seal(page);
```

Transitions the page from WRITABLE to EXECUTABLE. The runtime:

1. Computes an FNV-1a hash of the page contents and stores it
   in the guard for later integrity checks.
2. Calls `mprotect(..., PROT_READ | PROT_EXEC)` — the page is no
   longer writable. **The W is now gone, only the X is set.**
3. Records the state transition.

- **Returns** `0` on success, `-1` if the page is not in the
  WRITABLE state. Calling `wildx_seal` twice fails the second
  call (regression: `bug246`).
- **Hardware enforcement:** any subsequent write to the page
  traps with `SIGSEGV` from the MMU, not from a runtime check.
  This is the entire point of W^X: even a compromised JIT cannot
  modify already-sealed code.

## Step 4 — call through the page

```aria
fixed int32:result = npk_jit_call_i32_i32(page, 7i32, 35i32);
// result == 42
```

The call helper `memcpy`s the page pointer into a typed function
pointer and invokes it. The CPU executes the bytes you wrote.

- **No alignment requirements** beyond page alignment, which the
  allocator guarantees.
- **No CPU cache invalidation needed** on x86-64 — the L1i / L1d
  caches are coherent under `mprotect`. Other architectures
  (notably ARM64) need an `isb` / cache-flush dance before
  calling sealed code; that, plus the rest of ARM64 support, is
  deferred (`JIT-DEC-002`).

## Step 5 — `wildx_free(page)`

```aria
fixed int32:free_rc = wildx_free(page);
```

`munmap`s the page and removes the registry entry. The pointer
is now invalid for every operation:

- A second `wildx_free` on the same pointer returns `-1`.
- A `wildx_seal` on the same pointer returns `-1`.
- A `npk_jit_call_*` through the same pointer is undefined
  behaviour from the OS's perspective — the address is no
  longer mapped, and the call will fault.

The borrow checker rejects most of these statically (`ARIA-022`
double-free, `ARIA-015` use-after-release; regression
`bug247`). The runtime `-1` return is a backstop for pointers
that get smuggled through function boundaries the borrow checker
cannot see across.

## Pitfalls specific to JIT

- **Forgetting to seal.** Calling an unsealed page works on
  most CPUs (the page is still readable), but trips `PROT_EXEC`
  enforcement on systems with NX strictly enforced — and is a
  pure security own-goal regardless. Always seal.
- **Writing after seal.** Hardware will SIGSEGV you. There is
  no "soft" failure path. If you need to patch generated code,
  the v0.28.x answer is: free the page, allocate a new one,
  re-emit. (Patch-in-place is wishlist; see [FAQ](faq.md).)
- **Calling a freed page.** No safety net. Treat your `JitFn`
  binding as consumed once you `Jit.free` it.
- **Mixing `wild` and `wildx` pointers in calls to
  `wildx_*`.** The registry rejects pointers it did not mint
  (`-1` return). Don't try to seal a `wild alloc()` pointer.

## Validation

- `bug246` — double-seal returns `-1`.
- `bug247` — double-free is a compile error.
- `bug248` — leaked `wildx` allocation is a compile error.
- `bug264` — full end-to-end alloc → install → seal → call →
  free with the expected `7 + 35 == 42`.
