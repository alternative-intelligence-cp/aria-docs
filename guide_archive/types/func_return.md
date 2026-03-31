# Function Return Types

**Category**: Types → Functions  
**Purpose**: Specifying function return types

---

## Basic Return Types

```aria
// Returns integer
func:get_value = int32() {
    pass(42);
}

// Returns string
func:get_name = string() {
    pass("Alice");
}

// Returns nothing
func:print_hello = NIL() {
    print("Hello");
}
```

---

## Optional Returns

```aria
// Returns optional value
func:find_user = ?User(int32:id) {
    when not found then
        pass(nil);
    end
    pass(user);
}
```

---

## Result Returns

```aria
// Returns Result for error handling
func:divide = Result<int32>(int32:a, int32:b) {
    when b == 0 then
        fail("Division by zero");
    end
    pass(a / b);
}
```

---

## Function Returns

```aria
// Returns function
func:make_multiplier = fn(i32)(int32:n)-> i32 {
    return fn(x: i32) -> i32 {
        pass(x * n);
    };
}
```

---

## Multiple Returns (Tuple)

```aria
// Return multiple values as tuple
func:divide_with_remainder = (i32,(int32:a, int32:b)i32) {
    quotient: i32 = a / b;
    remainder: i32 = a % b;
    pass((quotient, remainder));
}

// Use
(q, r) := divide_with_remainder(17, 5);  // q=3, r=2
```

---

## Related

- [Functions](func.md)
- [Function Declaration](func_declaration.md)
- [Result](result.md)
- [Arrow (->)](../operators/arrow.md)

---

**Remember**: Use `->` to specify **return type**!
