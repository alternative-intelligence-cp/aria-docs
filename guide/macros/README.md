# Macros

Nitpick macros are compile-time code rewriting rules. A macro invocation is
replaced by its expanded body **before** type-checking and code generation.

## Contents

- [basic.md](basic.md) — Declaring and invoking macros
- [hygiene.md](hygiene.md) — Variable isolation and gensym
- [recursive.md](recursive.md) — Recursive macros and the depth guard
- [variadic.md](variadic.md) — Variadic macros (`..?params`)
- [code_gen.md](code_gen.md) — Statement-position and code-generating macros
- [builtins.md](builtins.md) — Built-in macros: `assert!`, `todo!`, `unreachable!`, `cfg!`
- [debug.md](debug.md) — Debugging macros with `--expand-macros`

## Overview

```aria
// declare a macro
macro:double = (x) {
    x + x;
};

// invoke it
int32:result = double!(10i32);   // expands to: 10 + 10
```

Macros differ from functions:

| | Macro | Function |
|---|---|---|
| Evaluated at | Compile time | Run time |
| Arguments | Unevaluated AST fragments | Evaluated values |
| Return type | Inferred from expansion | Declared explicitly |
| Recursion depth | Limited (default 64) | Unlimited |
| Hygiene | Yes — fresh variable names | N/A |

## When to Use Macros

- Generating repetitive code patterns
- Compile-time assertions (`assert!`, `todo!`, `unreachable!`)
- Platform-conditional code (`cfg!`)
- Zero-cost abstractions that would otherwise require runtime overhead
