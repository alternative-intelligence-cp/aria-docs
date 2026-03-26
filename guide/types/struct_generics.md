# Struct Generics

**Category**: Types → Structs  
**Purpose**: Generic struct types

---

## Basic Generic Struct

```aria
// Generic container
struct Box<T> {
    value: T
}

// Use with specific types
int_box: Box<i32> = { value = 42 };
string_box: Box<string> = { value = "hello" };
```

---

## Multiple Type Parameters

```aria
struct Pair<T, U> {
    first: T,
    second: U
}

// Use
pair: Pair<i32, string> = {
    first = 42,
    second = "answer"
};
```

---

## Generic Methods

```aria
struct Container<T> {
    items: []T
}

func:add = NIL(Container<T>->:self, T:item) {
    self.items.append(item);
}

func:get = ?T(Container<T>->:self, int32:index) {
    when index >= 0 and index < self.items.length() then
        pass(self.items[index]);
    end
    pass(nil);
}
```

---

## Common Patterns

### Option/Maybe

```aria
struct Option<T> {
    has_value: bool,
    value: T
}
```

### Result

```aria
struct Result<T, E> {
    is_ok: bool,
    value: T,
    error: E
}
```

---

## Related

- [Structs](struct.md)
- [Generics](../advanced_features/generics.md)

---

**Remember**: Generics make structs **reusable** across types!
