# Two-Phase Patterns

"Two-phase" here refers to patterns where a host is borrowed, the loan is
released, and then the host is borrowed again — possibly with a different
mutability. Nitpick supports this through *scope release*, but does not
implement Rust-style implicit two-phase borrows.

## Scope release on plain variables

```aria
int32:a = 10;
{
    $$i int32:r = a;
}                          // loan on a released here
$$m int32:m = a;           // OK — fresh loan, no conflict
m = 12;
exit m;                    // 12
```

(K test `056`.) The loan ends when its binding leaves scope; the next
borrow sees an unreserved host.

## Sequenced same-mutability loans

If you simply use one loan and then take another, the first loan does not
"survive" past the use — but you must still ensure the binding is out of
scope before the next borrow on the same host:

```aria
{
    $$m int32:r = pair.a;
    r = 1;
}
{
    $$m int32:r2 = pair.a;
    r2 = 2;
}
exit pair.a;               // 2
```

## Combined two-phase across different hosts

The most useful real-world shape is releasing one borrow inside a scope and
then taking a borrow on something else:

```aria
int32:a    = 10;
Pair:pair  = Pair{a: 1, b: 2};
{
    $$i int32:r = a;       // immutable borrow of `a`
}                          // released
$$m int32:rb = pair.b;     // mutable borrow of an unrelated path
exit a + rb + 18;          // 30
```

(K test `144`.)

## What is *not* supported

- **Implicit two-phase borrows.** You cannot have a `$$m self.bump()` call
  where `self.bump()` internally re-borrows from `self`. Methods take their
  receiver loan up front and hold it for the whole call.
- **Re-borrow of a covered field after an inner-scope release on a parent.**
  The K rules currently treat the parent loan as live until the outer
  binding leaves scope, even if you opened an inner scope.
- **Conditional release.** Releasing a loan inside one arm of an `if` and
  re-borrowing afterwards is rejected — the checker is not flow-sensitive
  across branches for borrows.

When you hit one of these, restructure with an explicit inner scope
(`{ ... }`) so the loan you want to release is the one whose binding falls
out.
