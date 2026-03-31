# Stack Allocation

## Overview

Stack allocation is the **default** in Aria. Variables are allocated on the function's
stack frame and automatically freed when the scope exits.

## Declaration

```aria
int32:x = 42;           // implicitly stack
stack int32:x = 42;      // explicitly stack (same effect)
```

## Scope Rules

Stack variables exist only within their declaring scope:

```aria
func:example = NIL() {
    int32:outer = 1;
    if (true) {
        int32:inner = 2;   // only exists in this if-block
    }
    // 'inner' is gone here
    pass NIL;
}
```

## Performance

Stack allocation is essentially free — just a pointer bump. No garbage collection
overhead, no manual deallocation needed.

## Related

- [overview.md](overview.md) — all allocation modes
- [gc.md](gc.md) — for data that must outlive scope
