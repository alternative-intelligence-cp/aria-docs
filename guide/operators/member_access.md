# Member Access & Navigation

| Operator | Meaning | Example |
|----------|---------|---------|
| `.` | Field access / enum member | `person.name`, `Color.RED` |
| `->` | Pointer member access | `ptr->field` |
| `?.` | Safe navigation (NIL-safe) | `ptr?.field` |

## Field Access

```aria
stack Point:p = Point{x: 10, y: 20};
int32:px = p.x;
int32:py = p.y;
```

## Pointer Access

```aria
stack Point->:ptr = @p;
int32:x_val = ptr->x;
ptr->x = 100;
```

## Enum Member Access

```aria
Color:c = Color.RED;           // dot notation for enum members
int64:r = Color.RED;
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
