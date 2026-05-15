# Stack

The `stack` region is the **default**. A bare local declaration uses it,
and the explicit `stack` keyword produces the same LLVM `alloca` — the
keyword exists to make intent obvious at the call site and to give the
borrow checker an explicit anchor for "must not escape this frame"
diagnostics.

```aria
func:hypotenuse_sq = int32(int32:a, int32:b) {
    stack int32:aa = a * a;
    stack int32:bb = b * b;
    pass aa + bb;
};
```

## Lifetime

A `stack` value lives for the surrounding function frame and is gone the
moment that frame returns. The borrow checker enforces this with
`ARIA-028` (formerly `ARIA-017`):

```aria
func:bad = $$m int32() {
    stack int32:tmp = 7i32;
    $$m int32:r = tmp;
    pass r;                  // [ARIA-028] tmp's stack frame ends here
};
```

The check is transitive — chaining through another local doesn't help —
and conservative: even a borrow into a `gc` *binding* is rejected,
because the binding name itself goes out of scope when the frame ends.

See [`borrow/diagnostics.md`](../borrow/diagnostics.md#aria-028--stack-escape-returnpass-of-borrow-into-a-local)
for the full text and fix recipes.

## When to use `stack`

Pretty much always, unless you have a concrete reason to pick something
else. Stack allocation is free (it's the function prologue/epilogue),
the borrow checker handles it cleanly, and there is no GC pressure or
manual cleanup.

Reach for [`gc`](gc.md) only when the value's lifetime needs to outlast
the current frame; reach for [`wild`](wild.md) only when interfacing
with C-style APIs.

## Inner-block scopes

`stack` lifetime extends to the *innermost* enclosing block, not just
the function. A borrow taken in an inner block must not be passed out
of that block:

```aria
func:bad = $$m int32() {
    if (1i32 == 1i32) {
        stack int32:inner = 99i32;
        $$m int32:r = inner;
        pass r;              // [ARIA-028] inner's frame ends at the }
    };
    pass 0i32;
};
```

The `outlives its host` variant of the diagnostic catches this even
when the binding survives long enough for the borrow to *create*; the
problem is the borrow trying to *leave*.

## Validation

K core test `147_alloc_stack_pass.aria` pins the runtime path; bug201,
bug205–208, and bug232–234 in the bug suite cover stack escape from
every shape the parser produces (`pass`, `fail`, nested blocks,
chained borrows, struct-field initializers).
