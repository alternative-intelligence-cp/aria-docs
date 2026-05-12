# Generics via Comptime

Nitpick has no separate "generics" feature. Instead, **types are first-class
at comptime** and you write generic code by passing types as `T: type`
parameters and dispatching on them with comptime intrinsics.

## `T: type` Parameters

A `comptime func:` may take a parameter of pseudo-type `type`. The argument
must be a type name visible at the call site.

```aria
comptime func:padding = int32(T: type) {
    int32:s = @sizeof(T);
    int32:a = @alignof(T);
    pass (a - (s % a)) % a;
};

fixed int32:p = comptime(padding(int32));   // 0
fixed int32:q = comptime(padding(Point));   // 0
```

## Type-Level Dispatch

Use `@typeInfo(T).kind` (or `.name`) to branch on type structure inside a
comptime function.

```aria
comptime func:descr = string(T: type) {
    if (@typeInfo(T).kind == "struct") {
        pass @typeInfo(T).name + " (struct)";
    }
    pass @typeInfo(T).name + " (primitive)";
};
```

## Generic Wrappers

Combine `T: type` with field reflection to build print/serialize helpers
without writing one per type.

```aria
comptime func:has_field = bool(T: type, name: string) {
    pass @fieldType(T, name) != "";
};

fixed bool:hx = comptime(has_field(Point, "x"));  // true
fixed bool:hz = comptime(has_field(Point, "z"));  // false
```

## Limits

- `T: type` parameters are **comptime-only**: they cannot appear on a
  runtime `func:`.
- Type names are erased after const evaluation: there is no runtime RTTI.
- Recursion over types is bounded by the CTFE budget.
