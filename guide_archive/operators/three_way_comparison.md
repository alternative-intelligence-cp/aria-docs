# Three-Way Comparison (<=>)

**Category**: Operators → Comparison  
**Operator**: `<=>`  
**Purpose**: Spaceship operator for ordering

---

## Syntax

```aria
<expression> <=> <expression>
```

---

## Description

The three-way comparison operator `<=>` (spaceship operator) compares two values and returns their ordering relationship.

---

## Return Values

```aria
// Returns:
// -1 if left < right
//  0 if left == right
//  1 if left > right

Result: i32 = 5 <=> 10;  // -1
Result: i32 = 10 <=> 10; //  0
Result: i32 = 15 <=> 10; //  1
```

---

## Sorting

```aria
// Perfect for sort comparisons
func:compare_users = int32(User:a, User:b) {
    pass(a.name <=> b.name);
}

users.sort(compare_users);
```

---

## Chained Comparison

```aria
// Try multiple criteria
func:compare = int32(Item:a, Item:b) {
    // First by priority
    Result: i32 = a.priority <=> b.priority;
    when result != 0 then
        pass(result);
    end
    
    // Then by name
    pass(a.name <=> b.name);
}
```

---

## Best Practices

### ✅ DO: Use for Sorting

```aria
func:compare = int32(int32:a, int32:b) {
    pass(a <=> b);
}
```

### ✅ DO: Use for Multi-Field Comparison

```aria
func:compare_records = int32(Record:a, Record:b) {
    result := a.date <=> b.date;
    when result != 0 then return result end;
    
    pass(a.id <=> b.id);
}
```

### ❌ DON'T: Use for Simple Equality

```aria
// Overkill
when (a <=> b) == 0 then ...

// Better
when a == b then ...
```

---

## Related

- [Less Than (<)](less_than.md)
- [Greater Than (>)](greater_than.md)
- [Equal (==)](equal.md)
- [Sorting](../concepts/sorting.md)

---

**Remember**: `<=>` returns **-1, 0, or 1** for ordering!
