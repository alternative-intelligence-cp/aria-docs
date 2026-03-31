# Struct

## Overview

Structs are composite types that group named fields. Methods are defined separately
from the struct declaration.

## Declaration

```aria
struct Point {
    flt64:x,
    flt64:y
}

struct Person {
    string:name,
    int32:age
}
```

## Instantiation and Access

```aria
Point:p = { x = 1.0, y = 2.0 };
println(p.x);    // 1.0
println(p.y);    // 2.0

Person:alice = { name = "Alice", age = 30 };
```

## Methods

Methods are defined as functions with a `->` self parameter:

```aria
func:area = flt64(Rectangle->:self) {
    pass (self.width * self.height);
}
```

## Nested Structs

```aria
struct Employee {
    string:name,
    Address:address
}

struct Address {
    string:city,
    string:zip
}

Employee:emp = { name = "Bob", address = { city = "Austin", zip = "78701" } };
println(emp.address.city);  // "Austin"
```

## Related

- [enum.md](enum.md) — enumeration types
- [memory_model/handle.md](../memory_model/handle.md) — Handle<T> for arena structs
