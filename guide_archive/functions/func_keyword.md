# The `func` Keyword

**Category**: Functions → Syntax  
**Usage**: `func name(params) -> return_type { body }`  
**Alias**: `fn` (short form, more commonly used)

---

## What is `func`?

`func` is the **long form** of the function declaration keyword. It's **identical** to `fn` - just more explicit.

---

## Syntax

```aria
// Long form
func add(a: i32, b: i32) -> i32 {
    pass(a + b);
}

// Short form (same thing)
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## When to Use Which

### Most Code Uses `fn`

```aria
// Common: Short and clean
func:calculate = int32(int32:x) {
    pass(x * 2);
}
```

### `func` for Emphasis or Style

```aria
// Rare: More explicit
func important_algorithm(data: []i32) -> Result<i32> {
    // Very important code
}
```

---

## Both Are Identical

```aria
// These are 100% identical:

func:method1 = NIL() { }
func method2() { }

// No difference in:
// - Performance
// - Compilation
// - Semantics
// - Visibility
```

---

## Convention

**Most Aria code uses `fn`**:
- ✅ Shorter to type
- ✅ Consistent with `if`, `for`, `when`
- ✅ Familiar to Rust developers
- ✅ Standard in examples and documentation

**Use `func` when**:
- 🎨 Personal preference
- 📖 Matching existing codebase style
- 📝 Emphasis in documentation

---

## Examples

### Both Forms Work

```aria
// fn style (common)
func:greet = NIL(string:name) {
    print("Hello, " + name + "\n");
}

// func style (also valid)
func farewell(name: string) {
    print("Goodbye, " + name + "\n");
}
```

### With Generics

```aria
// fn
func:max = T(T:a, T:b)where T: Comparable {
    when a > b then return a; else return b; end
}

// func
func min<T>(a: T, b: T) -> T where T: Comparable {
    when a < b then return a; else return b; end
}
```

### In Impl Blocks

```aria
struct Calculator;

impl Calculator {
    // fn style
    func:add = int32(int32:a, int32:b) {
        pass(a + b);
    }
    
    // func style
    func subtract(a: i32, b: i32) -> i32 {
        pass(a - b);
    }
}
```

---

## Async Functions

Works with both:

```aria
// fn
async func:fetch_data = Data?() {
    pass(await http_get("/api/data"));
}

// func
async func save_data(data: Data) -> bool {
    pass(await http_post("/api/data", data));
}
```

---

## Lambda Expressions

**Note**: Lambdas **don't** use `fn` or `func`:

```aria
// ❌ Wrong: Can't use fn/func in lambda
// callback = fn |x| x * 2;

// ✅ Right: Just the pipe syntax
callback = |x| x * 2;
```

---

## Related Topics

- [Function Declaration](function_declaration.md) - Function basics
- [Function Syntax](function_syntax.md) - Complete syntax reference
- [Lambda Expressions](lambda.md) - Anonymous functions

---

**Remember**: `fn` and `func` are **identical** - just pick one and be consistent!
