# Type: System

**Category**: Advanced Features → Type System  
**Syntax**: `Type:Name = { func:...; struct:...; };`  
**Purpose**: Composable type declarations grouping struct data with UFCS methods  
**Since**: v0.2.34

---

## Overview

`Type:` declarations organize related **struct definitions** and **methods** into a single cohesive unit. They replace scattered struct + function declarations with a structured grouping pattern.

A Type contains:
- `func:create` — Constructor
- `func:destroy` — Destructor
- `struct:internal` — Private fields
- `struct:interface` — Public fields
- `struct:type` — Static/class fields
- Additional `func:` methods — Instance and static methods

---

## Basic Type Declaration

```aria
Type:Counter = {
    func:create = Counter(int32:initial) {
        Counter:c = Counter{value: initial, id: 0i32};
        pass(c);
    };
    
    func:destroy = NIL(Counter:self) {
        pass(NIL);
    };
    
    func:increment = NIL(Counter:self) {
        pass(NIL);
    };
    
    func:nextId = int32() {
        pass(99i32);
    };
    
    struct:internal = {
        int32:id;
    };
    
    struct:interface = {
        int32:value;
    };
    
    struct:type = {
        int32:globalCount;
    };
};
```

---

## Creating Instances

Use `instance<T>()` to call the constructor:

```aria
Counter:c = raw(instance<Counter>(10i32));
// Calls Counter_create(10i32)
```

---

## Method Calls (UFCS)

Methods are desugared to `TypeName_methodName()` via UFCS:

```aria
c.increment();
// Becomes: Counter_increment(c)
```

Static methods (no `self` parameter):
```aria
int32:id = Counter.nextId();
// Becomes: Counter_nextId()
```

---

## Struct Sections

| Section | Purpose | Access |
|---------|---------|--------|
| `struct:interface` | Public fields — visible to callers | Public |
| `struct:internal` | Private fields — implementation detail | Private |
| `struct:type` | Static/class-level data (shared) | Static |

---

## Real-World Example: Channel Patterns

```aria
Type:FanOut = {
    func:create = int64(int64:in_ch, int32:n_out, int32:out_capacity) {
        pass(aria_chanpat_fanout_create(in_ch, n_out, out_capacity));
    };
    
    func:get_output = int64(int64:slot, int32:index) {
        pass(aria_chanpat_fanout_get_output(slot, index));
    };
    
    func:destroy = int32(int64:slot) {
        pass(aria_chanpat_fanout_destroy(slot));
    };
};
```

Usage:
```aria
int64:fanout = FanOut.create(input_ch, 4i32, 16i32);
int64:worker0 = FanOut.get_output(fanout, 0i32);
FanOut.destroy(fanout);
```

---

## Rules

1. Type names must start with uppercase
2. Methods inside a Type are UFCS — `Type.method()` becomes `Type_method()`
3. `create` is the conventional constructor name
4. `destroy` is the conventional destructor name
5. Instance methods take the type as first parameter (`self`)
6. Static methods have no `self` parameter

---

## See Also

- [Traits](traits.md) — Define shared behavior across types
- [Structs](../types/struct.md) — Standalone struct declarations
