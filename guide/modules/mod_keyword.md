# `mod` Keyword

**Category**: Modules → Declaration  
**Syntax**: `mod <name> { ... }` or `mod <name>;`  
**Purpose**: Declare or reference modules

---

## Overview

`mod` declares inline modules or references external module files.

---

## Inline Module

```aria
mod math {
    pub func:add = int32(int32:a, int32:b) {
        pass(a + b);
    }
    
    func:helper = int32() {  // Private to module
        pass(42);
    }
}

// Use module
Result: i32 = math.add(5, 3);
```

---

## External Module File

```aria
// Reference external file math.aria
mod math;

// Use it
Result: i32 = math.add(5, 3);
```

---

## Module from Directory

```
utils/
├── mod.aria       # Module root
├── string.aria
└── array.aria
```

```aria
// Reference directory module
mod utils;

// Access submodules
utils.string.reverse("hello");
```

---

## Nested Modules

```aria
mod outer {
    pub mod inner {
        pub func:function = NIL() {
            print("Inner function");
        }
    }
    
    pub func:outer_fn = NIL() {
        inner.function();  // Access nested
    }
}

// Use from outside
outer.inner.function();
outer.outer_fn();
```

---

## Visibility

```aria
mod private_mod {  // Module itself is private
    pub func:public_fn = NIL() { }  // Function is public within module
}

pub mod public_mod {  // Module is public
    pub func:public_fn = NIL() { }  // Accessible from outside
    func:private_fn = NIL() { }     // Not accessible
}
```

---

## Best Practices

### ✅ DO: One Module Per File

```aria
// math.aria
pub func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

// main.aria
mod math;  // ✅ Clean
```

### ✅ DO: Use `mod.aria` for Directories

```
database/
  mod.aria        # Main module file
  connection.aria
  queries.aria
```

### ❌ DON'T: Nest Modules Too Deeply

```aria
mod level1 {
    mod level2 {
        mod level3 {
            mod level4 {  // ❌ Too deep!
            }
        }
    }
}
```

---

## Related

- [mod](mod.md) - Modules overview
- [import](import.md) - Importing modules
- [pub](pub.md) - Public visibility

---

**Remember**: `mod` **declares** modules, `import` **uses** them!
