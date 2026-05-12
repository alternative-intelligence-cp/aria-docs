# `limit<Rules>` and `assert_static`

Comptime integrates with Nitpick's value-range system so the compiler can
**reject impossible configurations at build time**.

## `limit<R>` Short-Circuit at Comptime

When a `limit<R>` declaration's right-hand side is a `comptime(...)`
expression, the rule is checked during const evaluation. If the value
violates the rule, the build fails — there is no runtime `failsafe`.

```aria
Rules:Port = { $ >= 1, $ <= 65535 };

limit<Port> int32:p = comptime(8080);   // OK
limit<Port> int32:q = comptime(70000);  // BUILD ERROR
```

This is the v0.24.6 COMPTIME-011 feature; see `META/NITPICK_COMPTIME/COMPTIME.md`.

## `assert_static`

For ad-hoc compile-time invariants that don't fit in a `Rules:` block,
use `assert_static(cond, "message")`. The condition must be a comptime
boolean; a `false` condition is a build error with the given message.

```aria
assert_static(@sizeof(Point) == 16, "Point layout regression!");
assert_static(@alignof(int64) == 8, "Unexpected int64 alignment");
```

`assert_static` produces no runtime code.

## Combined: Sized Lookup Tables

A common pattern — generate a table at comptime and statically check its
shape:

```aria
comptime func:bitcount = int32(int32:n) {
    int32:c = 0;
    int32:x = n;
    loop(0, 32, 1) {
        if ((x % 2) == 1) { c = c + 1; }
        x = x / 2;
    }
    pass c;
};

fixed int32[256]:popcount8 = comptime({
    int32[256]:t = [...];
    loop(0, 256, 1) { t[$] = bitcount($); }
    pass t;
});

assert_static(@len(popcount8) == 256, "popcount table size");
```
