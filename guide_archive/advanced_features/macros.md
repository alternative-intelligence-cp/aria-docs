# Preprocessor Macros

**Category**: Advanced Features → Macros  
**Added**: v0.2.12  
**Purpose**: Text-level code generation and conditional compilation  
**Style**: NASM-inspired `%`-prefixed directives

---

## Overview

Aria's preprocessor runs as Phase 0 of compilation, before lexing. It performs text substitution, macro expansion, conditional compilation, and file inclusion. All directives use the `%` prefix.

---

## Simple Constants (`%define` / `%undef`)

```aria
%define MAX_SIZE 1024
%define APP_NAME "MyApp"
%define PI 3.14159265358979

func:main = int32() {
    int32:size = MAX_SIZE;
    pass(size);
};

failsafe {
    pass(1);
};
```

`%define` creates a text substitution — every occurrence of the identifier is replaced with the value.

```aria
%define TEMP_FLAG 1
// ... use TEMP_FLAG ...
%undef TEMP_FLAG
// TEMP_FLAG is no longer defined
```

**Note**: `%define` is value-only (no parameters). For parametric macros, use `%macro`.

---

## Numeric Assignment (`%assign`)

```aria
%assign COUNT 0
%assign NEXT_ID COUNT + 1
```

`%assign` evaluates a numeric expression and assigns the result.

---

## Multi-Line Macros (`%macro` / `%endmacro`)

```aria
%macro DECLARE_GETTER 2
func:get_%1 = %2() {
    pass(self.%1);
};
%endmacro
```

The number after the macro name is the **parameter count**. Parameters are referenced by position: `%1`, `%2`, etc. (1-indexed).

### Invocation

Macros can be invoked with parenthesized arguments or NASM-style space-separated arguments:

```aria
// Parenthesized (recommended for Aria)
DECLARE_GETTER(name, string)
DECLARE_GETTER(age, int32)

// NASM-style (also supported)
DECLARE_GETTER name string
```

---

## Parameter Substitution

| Pattern | Meaning | Example |
|---|---|---|
| `%1`, `%2`, ... | Positional parameter (1-indexed) | First arg, second arg, ... |
| `%0` | Argument count | `3` if called with 3 args |
| `%*` | All arguments as comma-separated list | `a, b, c` |
| `#%1` | Stringification — wraps param in quotes | `"hello"` if arg is `hello` |
| `##` | Token pasting — concatenates adjacent tokens | `foo##bar` → `foobar` |

### Example: Stringification and Token Pasting

```aria
%macro MAKE_TEST 2
func:test_##%1 = int32() {
    string:name = #%1;
    print("Testing: " + name + "\n");
    int32:result = %2;
    pass(result);
};
%endmacro

MAKE_TEST(addition, 2 + 3)
// Expands to:
// func:test_addition = int32() {
//     string:name = "addition";
//     print("Testing: " + name + "\n");
//     int32:result = 2 + 3;
//     pass(result);
// };
```

---

## Variadic Macros (`N+`)

Add `+` after the parameter count to accept additional arguments:

```aria
%macro LOG 1+
print("[" + %1 + "] " + %* + "\n");
%endmacro

LOG("INFO", "User logged in from ", "192.168.1.1")
// %1 = "INFO", %* = "User logged in from ", "192.168.1.1"
```

The `+` means "at least N arguments, accept more." Extra arguments beyond N are accessible via `%*`.

---

## Conditional Compilation

```aria
%define DEBUG

%ifdef DEBUG
func:log = NIL(string:msg) {
    print("[DEBUG] " + msg + "\n");
};
%endif

%ifndef RELEASE
// Include debug-only code
%endif
```

### Full Conditional Directives

| Directive | Purpose |
|---|---|
| `%ifdef NAME` | True if NAME is defined |
| `%ifndef NAME` | True if NAME is NOT defined |
| `%if expr` | True if expression is nonzero |
| `%elif expr` | Else-if branch |
| `%else` | Else branch |
| `%endif` | End conditional block |

### Example: Platform-Specific Code

```aria
%define PLATFORM_LINUX

%ifdef PLATFORM_LINUX
%define NEWLINE "\n"
%elif defined(PLATFORM_WINDOWS)
%define NEWLINE "\r\n"
%else
%define NEWLINE "\n"
%endif
```

---

## File Inclusion (`%include`)

```aria
%include "common_macros.aria"
%include "config.aria"
```

The included file's contents are inserted at the directive site, then preprocessed.

---

## Repeat Blocks (`%rep` / `%endrep`)

```aria
%assign I 0
%rep 5
// This block is repeated 5 times
%assign I I + 1
%endrep
```

---

## Context Stack (`%push` / `%pop`)

For local labels and scoped definitions:

```aria
%push my_context
%define %$local_var 42
// %$local_var is only valid in this context
%pop
// %$local_var is no longer accessible
```

The `%$` prefix creates context-local names, useful for avoiding collisions in nested macro expansions.

---

## Magic Constants

The preprocessor provides built-in constants:

| Constant | Expansion | Type |
|---|---|---|
| `__FILE__` | Current source filename (quoted string) | `"filename.aria"` |
| `__LINE__` | Current source line number (integer) | `42` |
| `__COUNTER__` | Auto-incrementing counter (starts at 0) | `0`, `1`, `2`, ... |

```aria
func:main = int32() {
    print("Running from " + __FILE__ + " line " + __LINE__ + "\n");
    
    // __COUNTER__ gives a unique integer each time it's referenced
    int32:id1 = __COUNTER__;
    int32:id2 = __COUNTER__;
    // id1 == 0, id2 == 1

    pass(0);
};

failsafe {
    pass(1);
};
```

**Note**: `__FILE__` expands to a quoted string. `__LINE__` and `__COUNTER__` expand to bare integers.

---

## Macro Hygiene

Aria's preprocessor automatically applies **hygienic renaming** to prevent variable name collisions across multiple expansions of the same macro. When the preprocessor detects `Type:Name` declaration patterns inside a macro body, it appends a unique suffix (`_hNNN`) to the variable name.

```aria
%macro GEN_ABS 1
func:abs_%1 = %1(%1:val) {
    %1:temp = val;
    if(temp < 0) {
        temp = 0 - temp;
    }
    pass(temp);
};
%endmacro

GEN_ABS(int8)
GEN_ABS(int32)
// Both expansions get unique 'temp' variables:
// int8:temp_h0 and int32:temp_h1 — no collision
```

### Hygiene Rules
- Only `Type:Name` patterns are renamed (not bare identifiers)
- Reserved keywords and built-in types are excluded
- Macro parameters (`%1`, `%2`, etc.) are not renamed
- Each expansion gets a unique atomic counter suffix

---

## Recursion Protection

- Direct recursion is caught via an `expanding_macros` tracking set
- Maximum expansion depth: 1000 levels
- If recursion is detected, the macro name is emitted as literal text

---

## Common Patterns

### Type-Generic Function Generator

```aria
%macro MAKE_ADD 1
func:add_%1 = %1(%1:a, %1:b) {
    pass(a + b);
};
%endmacro

MAKE_ADD(int32)
MAKE_ADD(int64)
MAKE_ADD(flt64)
```

### Assertion Macro

```aria
%macro ASSERT 1
if(!(%1)) {
    stderr_write("Assertion failed: " + #%1 + " at " + __FILE__ + ":" + __LINE__ + "\n");
    pass(1);
}
%endmacro
```

### Feature Flags

```aria
%define ENABLE_LOGGING
%define ENABLE_METRICS

%ifdef ENABLE_LOGGING
%macro LOG 1+
print("[LOG] " + %* + "\n");
%endmacro
%else
%macro LOG 1+
%endmacro
%endif
```

---

## Related

- [Compile-Time Evaluation](comptime.md) — `comptime(expr)` for semantic-level CTFE
- [Inline Functions](inline.md) — `inline func:` / `noinline func:` hints