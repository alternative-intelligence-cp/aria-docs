# Traits and Implementations

**Category**: Advanced Features → Type System  
**Syntax**: `trait:Name = { ... };`, `impl:Trait:for:Type = { ... };`  
**Purpose**: Define shared behavior contracts and implement them for types  
**Since**: v0.2.34

---

## Overview

Traits define a set of **method signatures** that a type must implement. They enable:

- **Polymorphism** — write generic code constrained by capabilities
- **UFCS dispatch** — trait methods become `TypeName_methodName()` functions
- **Trait bounds** — generic functions requiring specific capabilities

---

## Defining a Trait

```aria
trait:Describable = {
    func:describe = int32(int32:self);
};
```

A trait declares method signatures without bodies. Each method must list the type it applies to as the first parameter (named `self` by convention).

---

## Implementing a Trait

```aria
impl:Describable:for:int32 = {
    func:describe = int32(int32:self) {
        pass(self + 100i32);
    };
};
```

The `impl` block provides concrete method bodies for the specified type. Methods are mangled to `TypeName_methodName` at the ABI level (UFCS).

---

## Calling Trait Methods

Trait methods can be called via **UFCS** (Uniform Function Call Syntax):

```aria
int32:x = 5i32;
int32:result = raw(int32_describe(x));
// result == 105
```

Or via **method syntax** (dot notation):

```aria
int32:x = 5i32;
int32:result = raw(x.describe());
// result == 105
```

---

## Generic Functions with Trait Bounds

Constrain generic parameters to types that implement specific traits:

```aria
trait:Addable = {
    func:add_ten = int32(int32:self);
};

impl:Addable:for:int32 = {
    func:add_ten = int32(int32:self) {
        pass(self + 10i32);
    };
};

// T must implement Addable
func<T: Addable>:apply_trait = int32(T:val) {
    pass(val.add_ten());
};

func:main = int32() {
    int32:x = 7i32;
    int32:result = raw(apply_trait::<int32>(x));
    // result == 17
    pass(result);
};
```

---

## Multiple Traits

A type can implement multiple traits:

```aria
trait:Printable = {
    func:to_string = string(int32:self);
};

trait:Comparable = {
    func:compare = int32(int32:self, int32:other);
};

impl:Printable:for:int32 = {
    func:to_string = string(int32:self) {
        pass("an integer");
    };
};

impl:Comparable:for:int32 = {
    func:compare = int32(int32:self, int32:other) {
        pass(self - other);
    };
};
```

---

## Rules

1. Trait names must start with uppercase (convention)
2. All methods in a trait must be implemented in the `impl` block
3. The first parameter represents the receiver type (`self`)
4. Methods are mangled to `Type_method` for UFCS dispatch
5. Use turbofish `::<Type>` for explicit generic instantiation

---

## See Also

- [Generics](../functions/generics.md) — Generic type parameters
- [Type: System](type_system.md) — Composable type declarations
