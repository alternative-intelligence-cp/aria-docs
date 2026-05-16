# `Handle<T>`

A `Handle<T>` is a 64-bit value packed as:

```
[ generation : 32 | arena_id : 16 | slot_index : 16 ]
```

Generation `0` is reserved for `NPK_HANDLE_NULL`. Every `alloc` mints
a generation ≥ 1; every `free` and every `destroy` bumps the slot's
generation, so any previously-handed-out handle to that slot fails
its generation check on deref.

## Allocation

```aria
use "handle.npk".*;

int64:a            = raw HandleArena.create();
Handle<int32>:h    = raw HandleArena.alloc(a, 4i64);
```

`alloc(arena, size)` returns an `int64`. Because `Handle<T>` lowers
to `int64` and the two are bidirectionally assignable, you usually
write the typed binding directly. The `T` parameter is not enforced
beyond preventing accidental cross-type assignment
(`bug257_handle_type_mismatch_rejected.npk`):

```aria
Handle<int32>:hi = raw HandleArena.alloc(a, 4i64);
Handle<int64>:hl = hi;   // error: Handle<int32> -> Handle<int64> requires equal types
```

Handles **can** be assigned to / from raw `int64` freely. This is the
escape hatch you use for FFI and storage.

```aria
Handle<int32>:h = raw HandleArena.alloc(a, 4i64);
int64:as_int    = h;                  // store as int64
Handle<int32>:back = as_int;          // and back
```

## Dereference

`HandleArena.deref(h)` returns an `int64` that is either the
underlying buffer pointer (cast to `int64`) or `0i64` for NULL,
stale, or freed handles.

```aria
int64:p = raw HandleArena.deref(h);
if (p == 0i64) {
    exit 1;          // stale or NULL
};
// `p` is the buffer pointer; cast / FFI from here.
```

The runtime never returns a stale pointer. There is no UAF window —
even if the slot was reused by a later `alloc` (which gets a new
generation), the old handle's generation check fails first.

`bug258_handle_uaf_returns_null.npk` locks in that derefing a freed
handle returns `0i64`.

## Free

```aria
raw HandleArena.free(h);
```

`free` bumps the slot's generation. Subsequent derefs of `h` (or any
copy of `h`) return `0i64`. `free` of a NULL or stale handle is a
no-op.

You do **not** need to free every handle before destroying the
arena. `HandleArena.destroy(a)` bumps the generation of every slot
in one shot.

## Generation churn

Slots are reused. After `2³²` allocations into the same slot the slot
is retired (saturated). The runtime test
`tests/runtime/test_handle_v0277.cpp` covers `65 536` allocation
cycles to exercise the generation counter; the saturation case is
exercised via direct ABI manipulation.

## Member access

`Handle<T>` is opaque. The earlier design exposed `.index` and
`.generation` member access; v0.27.8 removed it. If you need to bit-
unpack a handle for a debugger or test, do it explicitly:

```aria
int64:as_int = h;
int32:gen    = (as_int >> 32i64) as int32;
int16:arena  = ((as_int >> 16i64) & 0xFFFFi64) as int16;
int16:slot   = (as_int & 0xFFFFi64) as int16;
```

In normal code, treat handles as opaque.

## See also

- [Arenas](arenas.md) — where handles come from.
- [Lifetimes](lifetimes.md) — when the compiler catches misuse.
- [Diagnostics](diagnostics.md) — error codes you may see.
