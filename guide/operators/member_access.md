# Member Access & Navigation

| Operator | Meaning | Example |
|----------|---------|---------|
| `.` | Field access | `person.name` |
| `->` | Pointer member access | `ptr->field` |
| `::` | Scope resolution / namespace | `Color::RED`, `vec2::ZERO` |
| `?.` | Safe navigation (NIL-safe) | `ptr?.field` |

## Field Access

```aria
Person:p = { name = "Alice", age = 30 };
println(p.name);    // "Alice"
println(p.age);     // 30
```

## Pointer Access

```aria
Person->:ptr = @person;
println(ptr->name);
```

## Scope Resolution

```aria
Color:c = Color::RED;          // enum variant
vec2:v = vec2::ZERO;           // type constructor
```

## Turbofish — `::<T>`

Explicit type annotation for generic functions:

```aria
int32:val = parse::<int32>("42");
```

## Safe Navigation

```aria
string:city = employee?.address?.city;  // NIL if any step is NIL
```
