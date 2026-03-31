# Boolean — `bool`

## Overview

Standard boolean type storing `true` or `false`. 1 byte storage. Default value: `false`.

## Declaration

```aria
bool:is_ready = true;
bool:done = false;
```

## Logical Operations

```aria
bool:a = true;
bool:b = false;
bool:and_r = (a && b);   // false — short-circuit AND
bool:or_r  = (a || b);   // true  — short-circuit OR
bool:not_r = (!a);        // false — logical NOT
```

Both `&&` and `||` use **short-circuit evaluation** — the right operand is skipped if the
left operand determines the result.

## Important

**No implicit int→bool conversion.** This is not C:

```aria
// WRONG: bool:flag = 1;     // compile error
// RIGHT: bool:flag = true;
```

## Style

```aria
// DON'T: if (is_ready == true) { ... }
// DO:    if (is_ready) { ... }
```
