# Brace-Delimited Syntax

**Category**: Advanced Features → Syntax  
**Purpose**: Block delimitation using braces

---

## Overview

Braces `{ }` delimit **blocks** of code in Aria.

---

## Function Bodies

```aria
func:simple = NIL() {
    print("Hello");
}

func:with_return = int32() {
    pass(42);
}

func:multi_statement = NIL() {
    x: i32 = 10;
    y: i32 = 20;
    print(x + y);
}
```

---

## Control Flow

```aria
// If statements
if condition {
    do_something();
}

if x > 10 {
    print("big");
} else {
    print("small");
}

// While loops
while running {
    process();
}

// Till loops
till(9, 1) {
    print($);
}
```

---

## Struct Initialization

```aria
struct Point {
    x: i32,
    y: i32,
}

point: Point = Point {
    x: 10,
    y: 20,
};

// Inline
point: Point = Point { x: 10, y: 20 };
```

---

## Blocks as Expressions

```aria
// Block returns a value
value: i32 = {
    x: i32 = 10;
    y: i32 = 20;
    x + y  // Return value (no semicolon)
};

// Conditional blocks
Result: i32 = if condition {
    42
} else {
    0
};
```

---

## Match Expressions

```aria
value: string = match number {
    0 => "zero",
    1 => "one",
    _ => "many",
};

// Multi-line arms
match request {
    Request::Get { path } => {
        print(`GET &{path}`);
        handle_get(path)
    }
    Request::Post { path, body } => {
        print(`POST &{path}`);
        handle_post(path, body)
    }
}
```

---

## Nested Blocks

```aria
func:nested = NIL() {
    {
        x: i32 = 10;
        print(x);
    }  // x goes out of scope here
    
    {
        y: i32 = 20;
        print(y);
    }  // y goes out of scope here
}
```

---

## Scope and Lifetime

```aria
func:scoping_example = NIL() {
    x: i32 = 10;
    
    {
        y: i32 = 20;
        print(x + y);  // Both x and y visible
    }  // y destroyed
    
    print(x);  // Only x visible
    // print(y;  // Error: y not in scope)
}
```

---

## Common Patterns

### Early Return

```aria
func:validate = Result<int32>(int32:x) {
    if x < 0 {
        fail("Negative value");
    }
    
    if x > 100 {
        fail("Value too large");
    }
    
    pass(x);
}
```

---

### Initialization Block

```aria
func:initialize = NIL() {
    // Setup block
    {
        load_config();
        init_database();
        start_services();
    }
    
    // Main logic
    run_application();
}
```

---

### Resource Management

```aria
func:process_file = Result<NIL>(string:path) {
    {
        file: File = open(path)?;
        defer file.close();
        
        // Work with file
        data: string = file.read_all()?;
        process(data);
    }  // file.close() called here
    
    pass();
}
```

---

### Labeled Blocks

```aria
Result: i32 = outer: {
    till(9, 1) {
        if $ == 5 {
            break outer 42;  // Exit with value
        }
    }
    0  // Default value
};
```

---

### Closure Bodies

```aria
// Single expression (no braces)
square = |x| x * x;

// Multiple statements (braces required)
complex = |x| {
    temp: i32 = x * 2;
    Result: i32 = temp + 10;
    result
};
```

---

## Empty Blocks

```aria
func:empty = NIL() {
    // Empty function
}

func:placeholder = NIL() {
    if condition {
        // TODO: implement
    }
}
```

---

## Best Practices

### ✅ DO: Use Braces for Clarity

```aria
// ✅ Clear
if condition {
    do_something();
}

// ✅ Even for single statements
if error {
    fail("Failed");
}
```

### ✅ DO: Indent Consistently

```aria
func:proper_indentation = NIL() {
    if condition {
        if nested {
            do_work();
        }
    }
}
```

### ✅ DO: Use Blocks for Scope Control

```aria
func:scoped_variables = NIL() {
    {
        temp: []u8 = allocate_large_buffer();
        process(temp);
    }  // temp freed here - good for memory
    
    continue_processing();
}
```

### ⚠️ WARNING: Match Brace Placement

```aria
// ⚠️ Unclosed brace
func:bad = NIL() {
    if condition {
        do_work();
    // Missing closing brace!

// ✅ Proper closing
func:good = NIL() {
    if condition {
        do_work();
    }
}
```

### ❌ DON'T: Unnecessary Nesting

```aria
// ❌ Too many nested braces
func:over_nested = NIL() {
    {
        {
            {
                do_work();
            }
        }
    }
}

// ✅ Simpler
func:better = NIL() {
    do_work();
}
```

---

## Brace Styles

### K&R Style (Recommended)

```aria
func:k_and_r = NIL() {
    if condition {
        do_work();
    } else {
        do_other();
    }
}
```

---

### Allman Style

```aria
func:allman = NIL()
{
    if condition
    {
        do_work();
    }
    else
    {
        do_other();
    }
}
```

---

### GNU Style

```aria
func:gnu = NIL()
  {
    if condition
      {
        do_work();
      }
    else
      {
        do_other();
      }
  }
```

---

## Single-Line Blocks

```aria
// Allowed but use sparingly
if x > 0 { print("positive"); }

func:one_liner = NIL() { return 42; }

// Preferred - multi-line for readability
if x > 0 {
    print("positive");
}

func:readable = NIL() {
    pass(42);
}
```

---

## Related

- [colons](colons.md) - Type annotations
- [semicolons](semicolons.md) - Statement terminators
- [whitespace_insensitive](whitespace_insensitive.md) - Whitespace rules

---

**Remember**: Braces **delimit scope** and make code structure **clear** and **explicit**!
