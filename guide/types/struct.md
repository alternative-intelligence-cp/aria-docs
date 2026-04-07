# Struct

## Overview

Structs are composite types that group named fields. Methods are defined via trait
implementations.

## Declaration

```aria
struct:Point = {
    int32:x;
    int32:y;
};

struct:Person = {
    string:name;
    int32:age;
};
```

## Instantiation and Access

Struct literal syntax with named fields:

```aria
stack Point:p = Point{x: 10, y: 20};
println(`&{p.x}`);    // 10
println(`&{p.y}`);    // 20

stack Person:alice = Person{name: "Alice", age: 30};
```

Positional arguments (order must match declaration):

```aria
Point:p = Point{10i32, 20i32};
```

Constructor syntax (desugars to `Type_create(args)`):

```aria
Counter:c = raw instance<Counter>(10i32);
```

Fields can also be assigned after declaration:

```aria
Point:p;
p.x = 10i32;
p.y = 20i32;
```

## Methods (via Trait Impls)

Methods are defined through trait implementations:

```aria
trait:HasArea = {
    func:area: int32(Rect2D:self);
};

struct:Rect2D = {
    int32:w;
    int32:h;
};

impl:HasArea:for:Rect2D = {
    func:area = int32(Rect2D:self) {
        pass self.w * self.h;
    };
};

// Call via UFCS (Uniform Function Call Syntax)
Rect2D:rect;
rect.w = 10i32;
rect.h = 5i32;
Result<int32>:a = rect.area();
```

## Passing Structs

```aria
func:add_coords = int32(Point:p) {
    pass p.x + p.y;
};

Point:origin = Point{ x: 0, y: 0 };
int32:sum = raw add_coords(origin);
```

## Related

- [enum.md](enum.md) — enumeration types
- [memory_model/handle.md](../memory_model/handle.md) — Handle<T> for arena structs
