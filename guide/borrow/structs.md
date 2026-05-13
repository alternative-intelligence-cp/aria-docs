# Borrowing Through Structs

This chapter covers struct-specific borrow patterns: borrows through whole
struct values, struct update expressions, pick patterns, and methods.

## Borrowing the whole struct

A `$$m` loan on a struct value reserves all of its fields:

```aria
struct:Pair = { int32:a; int32:b; };

func:main = int32() {
    Pair:p = Pair{a: 1, b: 2};
    $$m Pair:r = p;
    r.a = 10;
    r.b = 20;
    exit p.a + p.b;     // 30
};
```

While the loan is live, you cannot also borrow a field of `p` — the field
path is covered by the parent loan (see [paths.md](paths.md)).

## Struct update syntax (`v0.19.1`)

The update form `Type{ ...base, field: expr }` produces a new struct value
based on `base` with the listed fields overridden. It does not introduce a
borrow on `base`; it copies the underlying bytes:

```aria
Pair:base    = Pair{a: 1, b: 2};
Pair:patched = Pair{...base, a: 99};   // copy of base with a=99
exit base.a + patched.a;               // 1 + 99 = 100
```

This is useful when you want a "tweaked" copy without taking a borrow.

## Pick patterns (`v0.19.1`)

Pick patterns destructure a struct by name:

```aria
(Pair{ a, b }) = Pair{a: 10, b: 20};
exit a + b;                  // 30
```

The destructured names are fresh local bindings, not borrows. Use `$$i` /
`$$m` on the source struct if you actually need a loan.

## Method calls on `$$m` self

A method declared with `$$m` self takes a mutable borrow on its receiver
for the duration of the call:

```aria
struct:Counter = { int32:n; };

func:Counter.bump = int32($$m Counter:self) {
    self.n = self.n + 1;
    pass self.n;
};

func:main = int32() {
    Counter:c = Counter{n: 0};
    int32:r1 = c.bump();      // implicit $$m c during call
    int32:r2 = c.bump();
    exit r1 + r2;             // 1 + 2 = 3
};
```

You cannot hold an outer borrow on `c` while calling `c.bump()` — the
implicit `$$m` would conflict. Release the outer borrow first.

## Returning borrows from methods

Methods on `$$i`/`$$m` self can return primitive values freely. Returning
a borrow that *escapes* the receiver lifetime is rejected by the closure
analyzer in v0.25.x — use a value copy or rework the API.
