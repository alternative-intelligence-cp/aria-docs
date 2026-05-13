# Borrows and `async`

Nitpick's `async` model in v0.25.x is *synchronous-equivalent*: `await`
runs the async body to completion before continuing. From the borrow
checker's point of view, this means each `async` call is a fresh stack
frame whose loans live for the duration of the call only.

## Borrows inside an `async` body

```aria
async func: read_pair = int32($$i Pair:p) {
    pass p.a + p.b;
};

func:main = int32() {
    Pair:pair = Pair{a: 10, b: 20};
    int32:r = await read_pair(pair);
    exit r;                // 30
};
```

The `$$i p` loan exists only while `read_pair` runs. After `await` returns,
`pair` is unreserved.

## Repeated awaits on the same host

Because the loan is released on each `await` completion, you can call the
same async function multiple times on the same input:

```aria
async func: bump = int32(int32:n) { pass n + 1; };

func:main = int32() {
    int32:x = 10;
    int32:a = await bump(x);
    int32:b = await bump(x);
    int32:c = await bump(x);
    int32:d = await bump(x);
    exit a + b + c + d + x;     // 11+11+11+11+10 = 54
};
```

(K test `145`.)

## Holding a borrow across an `await`

This is rejected:

```aria
$$m int32:r = pair.a;
int32:v = await bump(pair.a);   // ARIA-023: can't take new borrow while r live
```

The `$$m r` loan is still in scope when the call wants its own loan on
`pair.a`. Release `r` (let it leave scope) before the `await`, or call
`bump` with a different host.

## What does *not* exist yet

- True concurrent execution — there is no scheduler in v0.25.x, so two
  awaits cannot interleave and step on each other. The borrow checker
  treats an async call exactly like a synchronous call for loan tracking.
- Cross-frame borrows. A borrow created inside an `async` body cannot
  escape into the awaiter's frame. Return a value instead.

When real concurrency lands (post v0.26.x roadmap), these rules will get
strictly tighter rather than looser.
