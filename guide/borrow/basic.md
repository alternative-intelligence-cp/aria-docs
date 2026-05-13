# Basic Borrows

The two borrow operators are `$$i` (immutable) and `$$m` (mutable). They
introduce a *loan* on a host variable that lasts as long as the borrowing
binding is in scope.

## The two operators

```aria
int32:value = 42;
$$i int32:r = value;   // immutable borrow — read-only view of `value`
$$m int32:m = value;   // mutable borrow   — read/write alias of `value`
```

`$$i` is read-only; `$$m` allows assignment through the alias and the host
sees the write after the loan ends.

## The 1-mut XOR N-immut rule

At any point a host can have either:

- any number of `$$i` loans, or
- exactly one `$$m` loan.

Mixing the two on the same host fails:

```aria
$$i int32:r = a;
$$m int32:m = a;       // ARIA-023: conflicting borrow on `a`
```

Two `$$m` loans on the same host also fail (`ARIA-023`).

## Release

A loan is released when its binding leaves scope. The host is then free to
be borrowed again or assigned to:

```aria
int32:a = 10;
{
    $$i int32:r = a;   // loan exists inside this block
}                      // loan released here
$$m int32:m = a;       // OK — new loan, no conflict
m = 12;
exit m;                // 12
```

This pattern is called **scope release**. The K test `056` locks it in.

## `failsafe` and borrow failures

When the borrow checker rejects code at compile time you get an `ARIA-023`
or `ARIA-026` diagnostic and no binary is produced. There is no runtime
borrow check — by the time `failsafe` runs, the rules above are already
satisfied. `failsafe` exists for *other* runtime faults (panics, OOM,
explicit `fail`).

## What "host" means

The host is the variable or path the loan tracks. For

```aria
$$m int32:rx = box.leaf.x;
```

the host is the path `box.leaf.x`, not `box`. This matters for path
disjointness — see [paths.md](paths.md).
