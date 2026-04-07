# Macros & Compile-Time Evaluation

## Overview

Aria provides three mechanisms for compile-time code generation and evaluation:

1. **AST macros** — Template-based code generation invoked with `name!(args)`
2. **Compile-time evaluation** — `comptime(expr)` for constant folding at compile time
3. **Derive macros** — Auto-generate trait implementations (see [Derive Macros](#derive-macros))

---

## AST Macros

### Declaration

Macros use `macro:name = (params) { body; };` syntax, consistent with `func:name`:

```aria
macro:double_it = (x) {
    x + x;
};
```

### Invocation

Invoke a macro with `name!(args)`:

```aria
func:main = int32() {
    int32:val = double_it!(5i32);
    // val == 10 (expanded to: 5i32 + 5i32)
    exit val;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

### Multiple Parameters

```aria
macro:add3 = (a, b, c) {
    a + b + c;
};

func:main = int32() {
    int32:result = add3!(1i32, 2i32, 3i32);
    // result == 6
    exit result;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

### How It Works

1. The parser sees `macro:name = (params) { body; };` and stores the AST template
2. When `name!(args)` is encountered, arguments are substituted into the template
3. Expansion happens at parse time — the macro body becomes inline code
4. Type checking runs on the expanded code, not the macro definition

### Key Differences from C Macros

| Feature | C Preprocessor | Aria Macros |
|---------|---------------|-------------|
| Expansion level | Text substitution | AST-level substitution |
| Type safety | None (text) | Full (post-expansion type check) |
| Hygiene | None | Scoped to expansion site |
| `#` operator | Stringify | **Pin** (borrow checker) |
| `##` operator | Token paste | **Does not exist** |
| Syntax | `#define NAME(x)` | `macro:name = (x) { };` |

> **Important**: Aria's `#` operator is the **pin** operator for the borrow checker, NOT a stringify operator.

---

## Compile-Time Evaluation

### comptime Expression

Evaluate an expression at compile time:

```aria
func:main = int32() {
    int32:size = comptime(4 * 1024);
    // size == 4096, computed at compile time
    exit 0;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

### comptime Function

Mark an entire function for compile-time evaluation:

```aria
comptime func:factorial = int64(int64:n) {
    if (n <= 1i64) {
        pass 1i64;
    }
    pass n * raw factorial(n - 1i64);
};

func:main = int32() {
    int64:val = comptime(raw factorial(10i64));
    // val == 3628800, computed entirely at compile time
    exit 0;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

### pub comptime Function

Export a compile-time function for use in other modules:

```aria
pub comptime func:align_up = int64(int64:val, int64:align) {
    pass ((val + align - 1i64) / align) * align;
};
```

### comptime Block

Execute a block of statements at compile time:

```aria
func:main = int32() {
    comptime {
        // All statements here execute at compile time
        int32:x = 10;
        int32:y = x * 2;
    };

    exit 0;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

---

## Derive Macros

Auto-generate trait implementations for structs using `#[derive(...)]`:

```aria
#[derive(Eq, Ord, Clone, ToString, Debug, Hash)]
struct:Point = {
    int32:x;
    int32:y;
};

func:main = int32() {
    Point:a = Point{ x: 1, y: 2 };
    Point:b = Point{ x: 3, y: 4 };

    bool:equal = raw a.eq(b);
    string:repr = raw a.to_string();

    exit 0;
};

func:failsafe = int32(tbb32:err) {
    exit 1;
};
```

### Available Derive Traits

| Trait | Method | Description |
|-------|--------|-------------|
| `Eq` | `eq(other)` | Field-by-field equality |
| `Ord` | `less_than(other)` | Lexicographic comparison |
| `Clone` | `clone()` | Shallow copy |
| `ToString` | `to_string()` | String representation |
| `Debug` | `debug()` | Debug output with type info |
| `Hash` | `hash()` | FNV-1a hash |

> Derived methods return `Result<T>` — use `raw` to unwrap the value.

---

## Attribute Macros

Attributes modify declarations at compile time:

```aria
#[inline]
func:hot_path = int32(int32:x) {
    pass x * 2;
};

#[noinline]
func:cold_path = int32(int32:x) {
    pass x / 2;
};

#[align(64)]
struct:CacheLine = {
    int64:data;
};
```

### Available Attributes

| Attribute | Target | Description |
|-----------|--------|-------------|
| `#[inline]` | Functions | Suggest inlining |
| `#[noinline]` | Functions | Prevent inlining |
| `#[align(N)]` | Structs/Vars | Set byte alignment |
| `#[derive(...)]` | Structs | Generate trait impls |
| `#[gpu_kernel]` | Functions | Mark as GPU kernel |
| `#[gpu_device]` | Functions | GPU device function |
| `#[comptime]` | Functions | Compile-time evaluation |

---

## See Also

- [MACRO_AUTHORING_GUIDE.md](../../reference/MACRO_AUTHORING_GUIDE.md) — Full derive and attribute reference
- [Traits](traits.md) — Trait system documentation
- [Generics](../functions/generics.md) — Generic functions and turbofish
