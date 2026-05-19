# The `jit.npk` helper

`stdlib/jit.npk` is the thin Nitpick-side wrapper over the v0.28.1
JIT runtime helpers and the v0.27.5 `wildx_*` builtins. Its job
is to let user code reach the JIT loop without writing `extern`
blocks or touching W^X state machinery by hand.

## Surface

```aria
use "jit.npk".*;

wildx int8->:f = raw Jit.compile_add_i32();
int32:r        = raw Jit.call_i32_i32(f, 7i32, 35i32);   // r == 42
fixed int32:rc = raw Jit.free(f);                        // rc == 0
```

Three methods, one signature, one ownership rule. That's the
whole surface this cycle ships.

> **`raw` prefix:** cross-`Type:` method calls return
> `Result<T>` per Nitpick's standard calling convention, so the
> caller writes `raw Jit.compile_add_i32()` to extract the value.
> Inside `Type:Jit` itself, the implementation calls the
> internal `wildx_alloc` / `wildx_seal` / `wildx_free` builtins,
> which return plain types (no `raw` needed).

## `Jit.compile_add_i32()`

```aria
func:compile_add_i32 = wildx int8->() {
    wildx int8->:page = wildx_alloc(64);
    fixed int32:write_rc = npk_jit_install_add_i32(page);
    if (write_rc != 0i32) {
        wildx_free(page);
    }
    fixed int32:seal_rc = wildx_seal(page);
    if (seal_rc != 0i32) {
        wildx_free(page);
    }
    pass page;
};
```

Allocates a 64-byte W^X page, writes the 5-byte
`int32 add(int32, int32)` sequence, and seals it. On install or
seal failure the page is freed before returning. The returned
`wildx int8->` is the only thing the caller needs to track.

The 64-byte allocation is much larger than the 5 bytes actually
written; this is intentional. The W^X allocator works at page
granularity, so any allocation under one page rounds up. The
spare bytes are zero-filled and unreachable from the `ret`.

## `Jit.call_i32_i32(f, a, b)`

```aria
func:call_i32_i32 = int32(wildx int8->:f, int32:a, int32:b) {
    pass npk_jit_call_i32_i32(f, a, b);
};
```

Thin shim over the runtime helper, which `memcpy`s `f` into a
typed function pointer of shape `int (*)(int, int)` and invokes
it. **The caller is responsible for matching the signature** —
calling `call_i32_i32` on a page that was compiled for a
different signature is undefined behaviour. In this cycle there
is only one signature, so the rule is vacuously satisfied; later
cycles will introduce a signature tag.

## `Jit.free(f)`

```aria
func:free = int32(wildx int8->:f) {
    pass wildx_free(f);
};
```

Releases the page. Returns `0` on success and `-1` on
double-free or foreign-pointer (`wild`-but-not-`wildx`)
arguments. The borrow checker registers this call as a
deallocator, so:

- `ARIA-014` will fire if you `compile_add_i32()` and don't
  `Jit.free` (or otherwise release) in the same scope.
- `ARIA-022` will fire on a double `Jit.free` of the same
  binding.

These are the same diagnostics that protect plain `wild`
allocations; nothing JIT-specific is involved.

## Ownership

`JitFn` (the `wildx int8->` returned by `compile_add_i32`) is
**explicitly owned by the caller**. There is no destructor hook,
no RAII, no failsafe path that frees it on unwind. You must
`Jit.free(f)` exactly once.

This is the same ownership story as `wild` / `wildx`, and the
same compile-time enforcement applies (`ARIA-014` for leaks,
`ARIA-022` for double-free).

Since v0.29.6, opt-in RAII is available: `use "drop.npk".*;`
flips `jit_fn_raii_enabled` and the compiler auto-emits
`npk_wildx_free(f)` at scope end for every
`wildx int8->:f = Jit.compile_*();` binding. The opt-in is a
separate flag from generic `wildx` RAII (DROP-DEC-007) and
plays correctly with bare-identifier `pass f` move semantics
(DROP-DEC-004) so factory functions still work. Full surface
in the [`guide/drop/`](../drop/README.md) cookbook.

## Why one signature?

Because emitting *user-supplied bytes for a user-supplied
signature* requires either (a) a real x86-64 assembler in the
runtime, or (b) a typed slice FFI that can pass `&[u8]` plus a
signature tag through to C. v0.28.x has neither, so this cycle
ships exactly the pieces needed to prove the loop closes:

- `wildx` builtins (v0.27.5) for the W^X heap.
- `npk_jit_install_add_i32` (v0.28.1) as a stand-in for the
  future "install caller-supplied bytes" path.
- `Jit.compile_add_i32` (v0.28.2) as a stand-in for the future
  generic `Jit.compile(bytes, sig) -> JitFn`.

This will widen as the supporting machinery lands. See the
[FAQ](faq.md) for the rough order.

## Two pages, one process

The W^X registry is a process-global `unordered_map<void*,
WildXGuard>`, so two `compile_add_i32()` calls produce two
independent pages with no shared state:

```aria
use "jit.npk".*;

func:main = int32() {
    wildx int8->:f1 = raw Jit.compile_add_i32();
    wildx int8->:f2 = raw Jit.compile_add_i32();
    fixed int32:r1 = raw Jit.call_i32_i32(f1, 20i32, 22i32);
    fixed int32:r2 = raw Jit.call_i32_i32(f2, 17i32, 25i32);
    fixed int32:rc2 = raw Jit.free(f2);
    fixed int32:rc1 = raw Jit.free(f1);
    if (r1 == 42i32 && r2 == 42i32 && rc1 == 0i32 && rc2 == 0i32) {
        exit 0;
    }
    exit 2;
};
```

The runtime serialises registry mutations under a mutex, so this
also works from multiple threads — but the *pointers themselves*
are not thread-safe to call concurrently without your own
synchronisation if they share state (and the `add` page doesn't,
so it's fine).

## Files

- `stdlib/jit.npk` — the `Type:Jit` surface in this chapter.
- `src/runtime/assembler/jit_smoke.cpp` — the C helpers
  (`npk_jit_install_add_i32`, `npk_jit_call_i32_i32`).
- `src/runtime/allocators/wildx_alloc.cpp` — the W^X page
  registry and `wildx_alloc` / `wildx_seal` / `wildx_free`
  implementations (v0.27.5).

## Validation

- `bug264` — bypass `jit.npk`; call the C helpers directly.
- `bug265` — happy path via `use "jit.npk".*;`.
- `bug266` — two independent pages, freed in reverse order.
