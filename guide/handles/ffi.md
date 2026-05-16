# Handles and FFI

Handles cross the FFI boundary in two shapes:

1. **As `int64` tokens** — pure value, safe to pass to any C function
   that wants to round-trip an opaque id back later.
2. **As the dereferenced buffer pointer** — `HandleArena.deref(h)`
   returns an `int64` that is the raw allocation pointer. This is the
   shape C usually wants.

## Passing the token

`Handle<T>` lowers to `int64`. To hand it to C you can assign through
an `int64` binding:

```aria
extern func:c_remember_handle = void(int64:tok);

Handle<int32>:h = raw HandleArena.alloc(a, 4i64);
int64:tok = h;                        // typed → raw
c_remember_handle(tok);
```

The C side stores the token. When it later wants to reach back into
the data:

```aria
extern func:c_recall_handle = int64();

int64:tok        = c_recall_handle();
Handle<int32>:h2 = tok;               // raw → typed
int64:p          = raw HandleArena.deref(h2);
if (p == 0i64) {
    // arena was destroyed (or slot freed) while C was holding the token.
    // This is sound — no UB, just NULL.
    exit 1;
};
```

The round trip is intentional. Handles are values; copies of the same
handle deref the same slot until the generation changes.

## Passing the buffer pointer

If C wants the raw bytes, deref first and pass the pointer:

```aria
extern func:c_write = void(int64:buf, int64:nbytes);

int64:p = raw HandleArena.deref(h);
if (p != 0i64) {
    c_write(p, 4i64);
};
```

Two rules of thumb:

- **Do not let the pointer outlive the deref site.** If C stores `p`
  for later, a subsequent `free(h)` or `destroy(arena)` makes that
  copy of `p` dangling — there is no generation check on the bare
  pointer. Either re-deref on every use, or copy out by value before
  returning to Aria.
- **Do not pass `p` back to Aria as if it were a borrow.** Handles do
  not produce `$$i` / `$$m` borrows. The pointer is a `wild`-style
  raw value the moment it leaves the handle layer.

## Storage shape

When you need to embed a handle in a struct that crosses FFI, use
`int64`:

```aria
Type:Job = {
    int64:id;          // handle as int64, opaque to C
    int32:priority;
};
```

On the Aria side you can read it back as `Handle<T>` by assignment:

```aria
Handle<JobPayload>:h = job.id;
```

## Lifetime gotcha

The static `ARIA-032` rule
([lifetimes](lifetimes.md)) does **not** follow handles through FFI
or through struct storage. A handle that left for C is no longer
tracked by the borrow checker; the runtime generation check is your
only safety net there. That is by design — the static rule is for
straight-line, single-function misuse; the runtime check is the
universal backstop.

## Validation

- `tests/runtime/test_handle_v0277.cpp` exercises the ABI directly,
  including stale-handle deref and arena-destroy invalidation.
- `bug251_wild_ffi_passthrough_pass.npk` is the analogous pattern
  for `wild` pointers across FFI; the same idioms apply to handle-
  derived pointers.

## See also

- [Handles](handles.md) — the `Handle<T>` surface.
- [`guide/memory/interop.md`](../memory/interop.md) — broader GC ↔
  `wild` interop rules; handles play by the `wild`-like rules once
  you deref past the handle layer.
