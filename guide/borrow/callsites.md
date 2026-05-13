# Call Sites

Borrows cross function boundaries through `$$i` and `$$m` parameters. The
checker treats each parameter as a separate loan on the caller's argument.

## `$$i` parameters

```aria
func:read_pair = int32($$i Pair:p) {
    pass p.a + p.b;
};

func:main = int32() {
    Pair:pair = Pair{a: 10, b: 20};
    int32:r = read_pair(pair);     // implicit $$i pair for the call
    exit r;                        // 30
};
```

Multiple `$$i` arguments to the same call on the same host are fine — they
all coexist under the N-immut rule.

## `$$m` parameters

```aria
func:bump_a = int32($$m Pair:p) {
    p.a = p.a + 1;
    pass p.a;
};
```

Only one `$$m` loan per host is allowed, so a call with two `$$m`
parameters must use *different* hosts (or different paths):

```aria
func:swap = int32($$m int32:x, $$m int32:y) {
    int32:t = x; x = y; y = t;
    pass 0;
};

int32:a = 1;
int32:b = 2;
swap(a, b);                  // OK — different hosts
swap(a, a);                  // ARIA-023 — same host twice
```

## `$$m` writeback

When a `$$m` parameter is written through, the new value is visible to the
caller after the call returns:

```aria
func:set42 = int32($$m int32:x) { x = 42; pass 0; };

func:main = int32() {
    int32:v = 0;
    set42(v);
    exit v;                  // 42
};
```

(K test `063`.) Multiple `$$m` parameters all writeback independently
(K tests `101`–`103`.)

## Holding a borrow across a call

If you hold a `$$i` or `$$m` loan on `host` and then call a function that
also wants `$$m host`, the call is rejected. Release the outer loan first
(scope it, or finish using it) before calling.

## Field-path arguments

You can pass a borrow on a path:

```aria
$$m int32:r = pair.a;
some_helper(r);              // r is the host now
```

But you cannot pass a *field expression* with an explicit borrow operator
in the call list — bind the borrow first, then pass the binding.
