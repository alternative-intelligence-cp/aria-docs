# Comptime Intrinsics

Nitpick exposes a small set of `@`-prefixed intrinsics that operate on
**types** at compile time. They are most useful inside `comptime(...)`
contexts where the result becomes a literal.

| Intrinsic | Returns | Description |
|---|---|---|
| `@sizeof(T)` | `int64` | Size of `T` in bytes |
| `@alignof(T)` | `int64` | Alignment of `T` in bytes |
| `@offsetof(T, "f")` | `int64` | Byte offset of field `f` in struct `T` |
| `@len(arr)` | `int64` | Element count of a fixed-size array |
| `@typeInfo(T)` | comptime struct | Reflection record for `T` |
| `@fieldType(T, "f")` | `string` | Type-name of field `f` in struct `T` |

## `@sizeof`, `@alignof`, `@offsetof`

```aria
Type:Point = { flt64:x; flt64:y; };

fixed int64:psize = comptime(@sizeof(Point));        // 16
fixed int64:palign = comptime(@alignof(Point));      // 8
fixed int64:y_off = comptime(@offsetof(Point, "y")); // 8
```

## `@len`

```aria
fixed int32[8]:ring = [0,0,0,0,0,0,0,0];
fixed int64:n = comptime(@len(ring));   // 8
```

## `@typeInfo`

`@typeInfo(T)` returns a comptime struct describing `T`. For struct types
it now includes a per-field `fields` map (v0.24.7, COMPTIME-013):

```aria
Type:Point = { flt64:x; flt64:y; };

fixed int32:nf = comptime(@typeInfo(Point).field_count);  // 2
fixed string:tx = comptime(@typeInfo(Point).fields.x.type_name);  // "flt64"
fixed int32:bx = comptime(@typeInfo(Point).fields.x.bit_width);   // 64
```

Available fields on a struct's `@typeInfo`:

| Path | Type | Notes |
|---|---|---|
| `.kind` | `string` | `"struct"` / `"int"` / `"flt"` / ... |
| `.name` | `string` | `"Point"` |
| `.size` | `int64` | Same as `@sizeof(T)` |
| `.alignment` | `int64` | Same as `@alignof(T)` |
| `.field_count` | `int32` | Number of fields |
| `.field_names` | `string` | Comma-separated names (legacy) |
| `.fields.<name>.name` | `string` | Field's name |
| `.fields.<name>.type_name` | `string` | Field's type as a string |
| `.fields.<name>.bit_width` | `int32` | 8/16/32/64 for primitives |
| `.fields.<name>.alignment` | `int64` | Field alignment |
| `.fields.<name>.offset` | `int64` | Same as `@offsetof(T, "<name>")` |

## `@fieldType` (v0.24.7)

A shorthand: `@fieldType(T, "f")` is equivalent to
`@typeInfo(T).fields.f.type_name`.

```aria
fixed string:t = comptime(@fieldType(Point, "y"));  // "flt64"
```
