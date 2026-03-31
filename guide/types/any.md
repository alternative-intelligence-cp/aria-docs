# Dynamic Type — `any`

## Overview

`any` is Aria's dynamically-typed value container (equivalent to C's `void*`). It can hold
any type, with the actual type tracked at runtime. Internally implemented as a fat pointer
`{ptr, i64}` with a type tag.

Also available as `dyn` (alias).

## Declaration

```aria
any:value = 42;
value = "hello";    // reassign to different type — legal with any
value = 3.14;
```

## Type Checking

!!! NOTE:
incorrect syntax for when use.
Correct form:
when(condition){
    //loops
}then{
    //happy path
}end{
    //error path
} is correct form
incorrect use of is. example use of is ternary: int8:a = is 2 > 1 : 4 : 5;
!!! END NOTE

```aria
when value is int32 then { 
    println("It's an integer");
}

// Pattern match with binding
when value is int32(n) then {
    println(`Integer value: &{n}`);
}
```

## Casting

!!! NOTE:
I am not aware of as being used to cast, we have some kind of cast<>() thing and => for casting
any type should have some methods attached that function in the way you are using as here. 
We just need to look them up and see what they are.
!!! END NOTE 

```aria
int32:num = value as int32;   // unsafe cast — panics on type mismatch
```

## Heterogeneous Collections

```aria
any[]:items = [42, "hello", 3.14, true];
```

## Performance

Dynamic type checking has runtime overhead. **Prefer static types** wherever possible.
Use `any` only when truly needed (e.g., heterogeneous containers, plugin interfaces).

## Related

- [struct.md](struct.md) — static composite types
- [result.md](result.md) — type-safe error handling
