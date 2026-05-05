# Nitpick Compiler Architecture Manual

**Target audience:** Anyone who wants to understand the compiler internals, contribute to the compiler, or implement new language features.

**Compiler version:** 0.2.13  
**LLVM version:** 20.1  
**Total codebase:** ~102,000 lines of C/C++ (compiler) + ~37,000 lines (runtime)

---

## Table of Contents

1. [High-Level Overview](#high-level-overview)
2. [Compilation Pipeline](#compilation-pipeline)
3. [Lexer](#lexer)
4. [Parser](#parser)
5. [AST Structure](#ast-structure)
6. [Semantic Analysis](#semantic-analysis)
7. [IR Generation (Backend)](#ir-generation-backend)
8. [Runtime Library](#runtime-library)
9. [Memory Model](#memory-model)
10. [FFI / Extern Mechanism](#ffi--extern-mechanism)
11. [Tooling](#tooling)
12. [Build System](#build-system)
13. [Source Map](#source-map)

---

## High-Level Overview

```
                   ┌─────────────┐
   .aria source ──►│ Preprocessor │
                   └──────┬──────┘
                          ▼
                   ┌──────────┐
                   │  Lexer   │──► Token Stream
                   └────┬─────┘
                        ▼
                   ┌──────────┐
                   │  Parser  │──► AST (ProgramNode)
                   └────┬─────┘
                        ▼
                ┌───────────────┐
                │ Type Checker  │  (+ GenericResolver, ModuleLoader)
                │ Borrow Checker│
                └───────┬───────┘
                        ▼
                ┌───────────────┐
                │ IR Generator  │──► LLVM IR Module
                └───────┬───────┘
                        ▼
                ┌───────────────┐
                │  LLVM Passes  │──► Object Code
                │  + Linker     │──► Executable
                └───────────────┘
```

The compiler (`npkc`) is a single-pass, ahead-of-time compiler. Source files go through preprocessing, lexing, parsing, type checking, borrow checking, and LLVM IR generation. LLVM handles optimization and machine code emission.

The compiler is written in C++17. The LLVM backend remains in C++ by design, while the compiler frontend has been self-hosted in Nitpick (lexer, parser, type checker, borrow checker, safety checker, exhaustiveness checker, const evaluator — 220 tests across 5 modules).

---

## Compilation Pipeline

Entry point: `src/main.cpp` (2,057 lines)

The `compile_to_module()` function orchestrates the full pipeline:

```
Phase 0: Preprocessor    → Macro expansion (NASM-style context stack)
Phase 1: Lexer           → Token stream
Phase 2: Parser          → AST (ProgramNode root)
Phase 3: TypeChecker     → Type inference, generic resolution, module loading
Phase 3.25: Z3Verifier   → SMT-based static proof of Rules/limit constraints (--verify)
Phase 3.5: BorrowChecker → Ownership analysis, loan tracking, move semantics
Phase 4: IRGenerator     → LLVM IR Module
```

After `compile_to_module()`, `main.cpp` handles:
- `--emit-llvm` → dump LLVM IR text
- `--emit-llvm-bc` → write LLVM bitcode
- `--emit-asm` → write assembly
- `--ast-dump` → print AST
- `--tokens` → print token stream
- `-E` → preprocessor output only
- Default → LLVM optimization passes → object file → link with `aria_runtime` → executable

**Optimization levels:** `-O0` through `-O3`, mapped directly to LLVM's `OptimizationLevel`. When `-g` (debug info) is set, optimization is automatically disabled to preserve variable visibility.

**Compiler flags:**
| Flag | Purpose |
|------|---------|
| `-o <file>` | Output path |
| `-O0` to `-O3` | Optimization level |
| `-g` | Emit DWARF debug info |
| `-I <dir>` | Include search path |
| `-l <lib>` | Link library |
| `-L <dir>` | Library search path |
| `-Wall` | Enable all warnings |
| `-Werror` | Warnings as errors |
| `--emit-llvm` | Output LLVM IR text |
| `--emit-asm` | Output assembly || `--emit-wasm` | Compile to WebAssembly (.wasm) || `--ast-dump` | Print AST tree |
| `--tokens` | Print token stream |
| `-E` | Preprocessor output only |

---

## Lexer

**Source:** `src/frontend/lexer/lexer.cpp` (1,466 lines)  
**Token definitions:** `include/frontend/token.h` (444 lines)  
**Token implementation:** `src/frontend/lexer/token.cpp` (429 lines)

The lexer produces a stream of `Token` values, each carrying a `TokenType`, string value, and source location (line, column).

### Token Categories (~200+ token types)

**Type keywords:** `int8` through `int4096` (signed), `uint8` through `uint4096` (unsigned), `tbb8`/`tbb16`/`tbb32`/`tbb64`, `flt32`/`flt64`/`flt128`/`flt256`/`flt512`, `fix256`, `frac8`–`frac64`, `tfp32`/`tfp64`, `bool`, `string`, `trit`, `tryte`, `nit`, `nyte`, `vec2`, `vec3`, `vec9`, and more.

**Memory qualifiers:** `wild`, `wildx`, `stack`, `gc`, `defer`, `move`

**Memory ordering:** `relaxed`, `acquire`, `release`, `acq_rel`, `seq_cst`

**Control flow:** `if`, `else`, `while`, `for`, `loop`, `till`, `when`/`then`/`end`, `pick`/`fall`, `break`, `continue`, `return`, `pass`, `fail`

**Async:** `async`, `await`, `catch`, `in`

**Compile-time:** `comptime`, `inline`, `noinline`

**Declarations:** `func`, `struct`, `enum`, `Type`, `opaque`, `trait`, `impl`, `use`, `mod`, `pub`, `extern`, `const`, `fixed`, `cfg`, `as`

**Contracts:** `requires`, `ensures`, `invariant`

**Operators:**
| Operator | Token | Meaning |
|----------|-------|---------|
| `@` | `AT` | Address-of |
| `$` | `DOLLAR` | Safe reference (borrow) |
| `#` | `HASH` | Pin |
| `->` | `ARROW` | Fat pointer member access |
| `<-` | `BACK_ARROW` | Dereference |
| `=>` | `FAT_ARROW` | Cast / match arm |
| `?.` | `SAFE_NAV` | Safe navigation |
| `??` | `NULL_COALESCE` | Nil coalescing |
| `?!` | `EMPHATIC_UNWRAP` | Checked unwrap |
| `!!!` | `FAILSAFE` | Unchecked / panic-style |
| `\|>` | `PIPE_RIGHT` | Pipeline forward |
| `<\|` | `PIPE_LEFT` | Pipeline backward |
| `..` | `RANGE_INCLUSIVE` | Inclusive range |
| `...` | `RANGE_EXCLUSIVE` | Exclusive range |
| `<=>` | `SPACESHIP` | Three-way comparison |

**Typed literals:** The lexer recognizes per-type suffixes. For example, `42i32` is an int32 literal, `3.14f64` is a float64 literal, `0tT10` is a balanced ternary literal, `0n4D` is a balanced nonary literal.

**Template literals:** Backtick-delimited strings with `&{expr}` interpolation expressions.

---

## Parser

**Source:** `src/frontend/parser/parser.cpp` (4,720 lines)  
**Header:** `include/frontend/parser/parser.h`

Recursive-descent parser producing a `ProgramNode` AST root. Notable features:

- **Generic syntax:** `<T>` on function/struct declarations, turbofish `::<T>` on call sites
- **Extern blocks:** `extern "lib" { ... }` with C-compatible declarations
- **Design-by-contract:** `requires`/`ensures`/`invariant` clauses attached to function declarations
- **Colon syntax:** Nitpick uses `type:name` for declarations (e.g., `int32:x = 5i32;`)
- **Function syntax:** `func:name = ReturnType(params) { body };` — note trailing semicolon
- **Two for-loop forms:** C-style `for(init; cond; step)` and range `for (i in a..b)`

### Parsing Functions (key methods)

| Method | Parses |
|--------|--------|
| `parseProgram()` | Top-level: use/mod/func/struct/enum/extern/trait/impl |
| `parseFuncDecl()` | Function declarations with generics, contracts, async |
| `parseStructDecl()` | Struct declarations with fields and generics |
| `parseExternStatement()` | Standalone extern or block `extern "lib" { }` |
| `parseExpression()` | Full expression with precedence climbing |
| `parsePrimary()` | Literals, identifiers, parens, array/object literals |
| `parseIfStatement()` | if/else chains |
| `parsePickStatement()` | Exhaustive pattern matching (pick/case/fall) |
| `parseForStatement()` | Both for-loop forms |
| `parseFailStatement()` | `fail(expr)` early-exit statement |

---

## AST Structure

**Base node:** `include/frontend/ast/ast_node.h` (128 lines)  
**Expressions:** `include/frontend/ast/expr.h` (488 lines)  
**Statements:** `include/frontend/ast/stmt.h` (646 lines)  
**Types:** `include/frontend/ast/type.h` (139 lines)  
**Implementations:** `src/frontend/ast/` (ast_node.cpp, expr.cpp, stmt.cpp, type.cpp)

All nodes inherit from `ASTNode` and use `std::shared_ptr<ASTNode>` (typedef'd as `ASTNodePtr`).

### NodeType Enum (49 node types)

**Expressions (22):** `LITERAL`, `IDENTIFIER`, `BINARY_OP`, `UNARY_OP`, `CALL`, `INDEX`, `MEMBER_ACCESS`, `POINTER_MEMBER`, `LAMBDA`, `TEMPLATE_LITERAL`, `RANGE`, `TERNARY`, `SAFE_NAV`, `NULL_COALESCE`, `PIPELINE`, `UNWRAP`, `ARRAY_LITERAL`, `OBJECT_LITERAL`, `AWAIT`, `MOVE`, `VECTOR_CONSTRUCTOR`, `CAST`

**Statements (16):** `VAR_DECL`, `FUNC_DECL`, `STRUCT_DECL`, `ENUM_DECL`, `TYPE_DECL`, `OPAQUE_STRUCT`, `TRAIT_DECL`, `IMPL_DECL`, `RETURN`, `PASS`, `FAIL`, `BREAK`, `CONTINUE`, `DEFER`, `BLOCK`, `EXPRESSION_STMT`

**Control Flow (7+):** `IF`, `WHILE`, `FOR`, `LOOP`, `TILL`, `WHEN`, `PICK`, `PICK_CASE`, `FALL`

**Type Nodes (7):** `TYPE_ANNOTATION`, `GENERIC_TYPE`, `ARRAY_TYPE`, `POINTER_TYPE`, `SAFE_REF_TYPE`, `OPTIONAL_TYPE`, `FUNCTION_TYPE`

**Module Nodes (4):** `USE`, `MOD`, `EXTERN`, `PROGRAM`

### Key Expression Nodes

| Node | Fields | Purpose |
|------|--------|---------|
| `LiteralExpr` | value, literal_type | Numeric, string, bool, ternary, nonary literals |
| `BinaryExpr` | left, op, right | All binary operations (+, -, *, ==, &&, `\|>`, etc.) |
| `CallExpr` | callee, arguments, generic_args | Function calls with optional turbofish |
| `MemberAccessExpr` | object, member | Dot access (`obj.field`) |
| `IndexExpr` | object, index | Array/collection indexing |
| `AwaitExpr` | operand | `await` keyword on async calls |
| `CastExpr` | expr, target_type | `@cast<T>(expr)` |
| `LambdaExpr` | params, body, captures | Closure expression |
| `UnwrapExpr` | expr, default_value, is_force | `?` (with default) or `!` (force) unwrap |

### Key Statement Nodes

| Node | Fields | Purpose |
|------|--------|---------|
| `FuncDeclStmt` | name, params, return_type, body, is_async, generics, contracts | Function declaration |
| `StructDeclStmt` | name, fields, generics | Struct type declaration |
| `VarDeclStmt` | type, name, initializer, is_const | Variable declaration |
| `PassStmt` | value | `pass(expr)` — success result |
| `FailStmt` | value | `fail(expr)` — error result |
| `PickStmt` | expression, cases, fall_case | Exhaustive match statement |
| `DeferStmt` | body | Deferred execution block |

---

## Semantic Analysis

15 analysis passes in `src/frontend/sema/`:

### TypeChecker (10,826 lines)

The largest single file. Performs:
- Type inference for variable declarations
- Function signature checking (parameters, return types)
- Generic instantiation via the `GenericResolver` (monomorphization)
- Module loading via `ModuleLoader` (resolves `use` imports)
- Operator type checking with promotion rules
- Result type checking (`pass`/`fail` consistency)
- Struct field validation and layout
- Enum variant checking
- `pick` exhaustiveness (delegates to `ExhaustivenessChecker`)
- "Did you mean?" suggestions for misspelled identifiers
- UFCS (Uniform Function Call Syntax) resolution

**Type representation** (`src/frontend/sema/type.cpp`, 1,148 lines):

`TypeKind` enum: `PRIMITIVE`, `POINTER`, `ARRAY`, `SLICE`, `FUNCTION`, `STRUCT`, `UNION`, `VECTOR`, `GENERIC`, `OPTIONAL`, `RESULT`, `FUTURE`, `DIMENSIONAL`, `HANDLE`, `SIMD`, `UNKNOWN`, `ERROR`

Key type classes:
- `PrimitiveType` — name, bitWidth, isSigned, isFloating, isTBB, isExotic
- `PointerType` — pointeeType, isMutable, isWild, isErased
- `ArrayType` — elementType, size (-1 for dynamic)
- `FunctionType` — paramTypes, returnType, isAsync, isVariadic
- `StructType` — name, fields (name/type/offset/isPublic), size, alignment
- `DimensionalType` — baseType + 7 SI base dimension exponents (length, mass, time, current, temperature, amount, luminosity) with compile-time dimensional algebra

### GenericResolver (1,066 lines)

Handles generic function and struct instantiation:
- Type parameter binding from call-site arguments
- Turbofish `::<T>` explicit specialization
- Monomorphization: each unique `<T>` instantiation produces a separate LLVM function
- Generic constraint checking (future: trait bounds)

### BorrowChecker (2,422 lines)

Implements "Appendage Theory" for memory safety:
- Tracks variable ownership (owned, borrowed, moved)
- `$` creates safe references (loans), `#` creates pinning references
- Validates no use-after-move
- Ensures mutable references are exclusive
- Scope-based lifetime analysis (no explicit lifetime annotations)
- `UnaryExpr` nodes carry `creates_loan`, `loan_target`, `creates_pin`, `pin_target` annotations

### Other Passes

| Pass | Lines | Purpose |
|------|------:|---------|
| ConstEvaluator | 1,469 | Compile-time constant expression evaluation |
| DefiniteAssignment | 465 | Ensures variables are assigned before use |
| ExhaustivenessChecker | 453 | Validates `pick` covers all cases |
| AsyncAnalyzer | 204 | Checks async/await usage validity |
| VisibilityChecker | 150 | Enforces `pub` visibility rules |
| SafetyChecker | — | Audits unsafe patterns (wild, extern, cast, TODO) |
| ClosureAnalyzer | — | Analyzes closure captures |

---

## IR Generation (Backend)

**Main files:**

| File | Lines | Purpose |
|------|------:|---------|
| `ir_generator.cpp` | 10,084 | Module setup, top-level declarations, function codegen, extern blocks, debug info, generic specialization |
| `codegen_expr.cpp` | 10,531 | Expression codegen: literals, binary/unary ops, calls, member access, arrays, casts, LBIM arithmetic, UFCS, pipelines |
| `codegen_stmt.cpp` | 2,929 | Statement codegen: var decl, control flow, defer, return/pass/fail, struct/enum/trait/impl |
| `tbb_codegen.cpp` | 467 | TBB (Twisted Balanced Binary) safe arithmetic with overflow sentinel detection |
| `ternary_codegen.cpp` | 600 | Balanced ternary & nonary arithmetic codegen |

### IRGenerator Class

Wraps LLVM's core objects:
- `llvm::LLVMContext` — type and constant uniquing
- `llvm::Module` — the compilation unit
- `llvm::IRBuilder<>` — instruction insertion point management

Key internal state:
- `named_values` — symbol table mapping variable names to `llvm::Value*`
- `value_types` — tracks the Nitpick type of each `llvm::Value*`
- `var_aria_types` — maps variable names to Nitpick types (for borrow checker interop)
- `current_function` — the LLVM function being generated
- `loop_stack` — stack of (continue_bb, break_bb) for nested loops
- `pick_merge_bb` — merge block for pick/case chains
- `defer_stack` — LIFO stack of defer blocks for scope exit

### Expression Codegen Highlights

**Result types:** `pass(val)` generates a struct `{value, error, is_error}` with `is_error = false`. `fail(err)` generates the same struct with `is_error = true`. The `?` unwrap operator checks `is_error` and either extracts the value or falls through to the default.

**Async/await:** Uses LLVM coroutine intrinsics:
- `@llvm.coro.id`, `@llvm.coro.begin` — coroutine frame setup
- `@llvm.coro.suspend` — suspension points
- `@llvm.coro.promise` — access promise area for pass/fail result storage
- `@llvm.coro.resume`, `@llvm.coro.destroy` — caller-side resume/cleanup
- Coroutine frames allocated with `malloc`, freed with `free` (not GC)
- `drop()` on async values auto-resumes and destroys the coroutine frame

**LBIM (Limb-Based Integral Model):** For large integers (128–4096 bit), LLVM's native `iN` types have bugs at high widths. LBIM splits large integers into 64-bit limbs and implements arithmetic (add, sub, mul, div, mod, comparisons) using carry chains.

**UFCS (Uniform Function Call Syntax):** `x.method(args)` is rewritten to `method(x, args)` when no struct method matches.

### Debug Info

When `-g` is passed, `IRGenerator` uses a `DIBuilder` to emit DWARF debug information:
- Source file/directory metadata
- Subprogram entries for every function
- Variable locations tied to alloca instructions
- Line number metadata on every instruction
- Compile unit with language ID, producer string

---

## Runtime Library

**Location:** `src/runtime/` (~33,900 lines) + headers in `include/runtime/`  
**Build target:** `aria_runtime` (static library linked into every compiled Nitpick program)

### Core Components

| Component | Source | Lines | Description |
|-----------|--------|------:|-------------|
| **GC** | `gc/gc.cpp`, `gc/allocator.cpp` | 521 + 396 | Hybrid generational: copying nursery + mark-sweep old gen |
| **Allocators** | `allocators/` (8 files) | ~2,400 | wild_alloc, wildx_alloc, arena, pool, slab |
| **Strings** | `strings/strings.cpp` | 1,008 | Creation, concatenation, comparison, slicing |
| **Collections** | `collections/` (5 files) | ~1,500 | Dynamic arrays, typed collections, hash maps |
| **I/O** | `io/io.cpp` | 1,256 | File I/O with stream abstraction |
| **Async** | `async/` (11 files) | ~3,200 | Event loop, executor, io_uring, work-stealing scheduler |
| **Streams** | `streams/streams.cpp` | 1,039 | Stream abstraction layer |
| **Threading** | `thread/thread.cpp` | 898 | Thread management, synchronization primitives |
| **Atomics** | `atomic/` (2 files) | ~1,260 | Atomic operations with memory ordering |
| **Math** | `math/` (5 files) | ~2,750 | Basic math, LBIM extended, fix256 (Q128.128) |
| **High-precision** | `highprec_float.cpp`, `highprec_simd.cpp` | 580 + 1,157 | flt256/flt512 with AVX-512 SIMD |
| **JSON** | `json/` (2 files) | ~1,550 | Parsing and serialization |
| **TOML** | `toml/` (2 files) | ~1,940 | Parsing (uses toml11 vendor lib) |
| **Process** | `process/process.cpp` | 696 | Process spawning, piping |
| **Networking** | `net/net.cpp` | 257 | TCP socket operations |
| **Formatting** | `fmt/formatters.cpp` | 583 | String formatting engine |
| **Telemetry** | `telemetry/telemetry.cpp` | 735 | Runtime diagnostics |
| **Assembler** | `assembler/` (3 files) | ~1,240 | JIT assembler, code caching |
| **Panic** | `panic/panic_handler.cpp` | 207 | Panic handler (failsafe entry point) |
| **Result** | `result/result.cpp` | 342 | Result type runtime support |

### Exotic Type Runtime Ops

These live in `src/backend/runtime/` and support Nitpick's unique numeric types:

| File | Lines | Types Supported |
|------|------:|-----------------|
| `ternary_ops.cpp` | 821 | Balanced ternary (trit/tryte) and nonary (nit/nyte) |
| `ttensor_ops.cpp` | 730 | 9D toroidal tensor (for Nikola ATPM model) |
| `tfp_ops.cpp` | 610 | Twisted floating-point |
| `frac_ops.cpp` | 563 | Exact rational arithmetic |
| `tmatrix_ops.cpp` | 514 | Twisted matrix operations |
| `vec9_ops.cpp` | — | 9D vector operations |

---

## WebAssembly Target (v0.2.13)

When `--emit-wasm` is passed, the compiler targets the `wasm32-unknown-wasi` triple:

- LLVM emits WebAssembly object code
- Linked with `wasm-ld` against a WASM-specific runtime (`aria_runtime_wasm`)
- Output is a `.wasm` module runnable under WASI-compatible runtimes (Wasmtime, Wasmer)

### WASM Restrictions

- No threading (single-threaded only)
- No process spawning (`fork`/`exec` unavailable in WASM sandbox)
- No signals or `mmap` (WASM uses linear memory)
- No native FFI (WASM has its own import model)
- File I/O requires a WASI-compatible runtime

### WASM Runtime

A separate `aria_runtime_wasm` static library is built targeting wasm32. It excludes threading, io_uring, process, and networking subsystems. Core features (strings, math, collections, Result types) work unchanged.

---

## Memory Model

Nitpick provides three allocation strategies:

### 1. GC (Default)

Hybrid generational garbage collector:
- **Nursery (young gen):** Bump-pointer allocation with copying collection
- **Old gen:** Mark-sweep collection
- **Object header:** 64-bit `ObjHeader` on every GC allocation — mark/pin/forward/nursery bits + size_class + type_id
- **Thread-local allocation buffers (TLABs)** for lock-free young-gen allocation
- **`#` operator (pin):** Prevents GC relocation for objects that need stable addresses (interop with wild pointers, FFI)

### 2. Wild (`wild` keyword)

Manual allocation, opting out of GC:
- `wild_alloc` wraps `malloc`/`free`
- `wildx_alloc` for JIT executable memory (`mmap` with `PROT_EXEC`)
- `move(x)` transfers ownership; borrow checker tracks use-after-move
- `defer { ... }` for RAII cleanup in LIFO order
- Required for FFI interop where C code expects raw pointers

### 3. Stack (`stack` keyword)

Explicit stack allocation:
- `stack` qualifier forces stack placement
- No GC overhead, no heap allocation
- Lifetime tied to enclosing scope

### Borrow Checker Integration

The borrow checker enforces safety across all strategies:
- `$$i T:name = value` creates an immutable borrow/alias
- `$$m T:name = value` creates a mutable borrow/alias
- `#x` pins an object (prevents GC relocation)
- Scope-based lifetimes — no explicit `'a` annotations needed
- Move semantics: variables are moved by default; declare `$$i`/`$$m` borrow
        intent at declarations and parameters

---

## FFI / Extern Mechanism

### Syntax

Two forms:

```aria
// Standalone extern
extern func:strlen = uint64(string:s);

// Block extern with library name
extern "libm" {
    func:sin = flt64(flt64:x);
    func:cos = flt64(flt64:x);
    func:sqrt = flt64(flt64:x);
};
```

### How It Works

**Parsing** (`parser.cpp`, `parseExternStatement()`):
- Standalone: creates an `ExternStmt` with single declaration
- Block: parses `extern "libname" { ... }` with multiple function/variable/struct/opaque declarations

**Type Mapping** (`ir_generator.cpp`, `mapFFIType()`):
- Nitpick primitive types → matching LLVM types (int32 → i32, flt64 → double, etc.)
- `string` → `ptr` (C string pointer)
- `wild T->` → `ptr` (opaque pointer)
- `?*` (type-erased pointer) → `ptr` (for C's `void*`)
- Unknown/opaque types → `ptr`

**Code Generation:**
- `llvm::Function::Create()` with `ExternalLinkage` (no body)
- Callsites use standard LLVM call instructions
- Linking: `-l <lib>` and `-L <dir>` flags passed through to the system linker

**Conventions:**
- `*` (thin C pointers) allowed only inside extern blocks
- `->` (fat Nitpick pointers) only outside extern blocks
- `opaque struct:Name;` declares C types the compiler cannot inspect
- `void` return type allowed only in extern context; Nitpick code uses `NIL`

### C Shim Pattern

For complex C libraries, Nitpick packages use a C shim that provides a simplified API:

```
┌──────────┐     ┌───────────┐     ┌──────────┐
│ Nitpick code │────►│ C shim.so │────►│ C library│
│ (extern)  │     │ (wrapper) │     │ (e.g. PQ)│
└──────────┘     └───────────┘     └──────────┘
```

The shim handles:
- Complex struct lifecycle (allocate/use/free pools)
- String memory management (shim owns strings, Nitpick reads via helpers)
- Error code translation

---

## Tooling

### npkc (Compiler Driver)

**Entry:** `src/main.cpp` (1,596 lines)

Parses CLI flags, runs the pipeline, handles output modes (executable, IR dump, AST dump, token dump).

### aria-ls (Language Server)

**Source:** `src/tools/lsp/` (~1,736 lines)

LSP server providing:
- Hover (type signatures + builtin descriptions)
- Goto-definition
- Completion (37 keywords + 15 types + file symbols)
- Document symbols
- Find references
- Signature help

Links only lexer/parser/AST/diagnostics — no LLVM dependency.

### aria-dap (Debug Adapter)

**Source:** `src/tools/debugger/` (~2,218 lines)

DAP server using LLDB 20 as backend:
- Breakpoints (line, conditional, logpoints)
- Stepping (into, over, out)
- Stack traces and variable inspection
- Nitpick-specific type formatters
- Memory visualization
- Async coroutine debugging

Conditionally built — requires LLDB development libraries.

### npkpkg (Package Manager)

**Source:** `src/tools/pkg/` (~1,078 lines)

- Install packages from registry
- Search and list available packages
- Pack and publish packages
- Tarball extraction and manifest parsing

### aria-doc (Documentation Generator)

**Source:** `src/tools/doc/` (~1,309 lines)

- Parses Nitpick source files for doc comments
- Generates HTML documentation pages
- Produces 435+ pages from the ecosystem

---

## Build System

**File:** `CMakeLists.txt` (480 lines)

### Prerequisites

- **LLVM 20.1+** — core dependency
- **CMake 3.20+** — build system
- **C++17 compiler** — Clang or GCC
- **liburing** — for io_uring async runtime
- **LLDB** (optional) — for aria-dap debugger

### Build Targets

| Target | Type | Description |
|--------|------|-------------|
| `aria_runtime` | Static lib | Runtime linked into compiled programs |
| `npkc` | Executable | Main compiler |
| `ariac_testing` | Executable | Separate test compiler (avoids fuzzer disruption) |
| `aria-ls` | Executable | Language Server |
| `aria-dap` | Executable | Debugger (conditional on LLDB) |
| `npkpkg` | Executable | Package manager |
| `aria-doc` | Executable | Documentation generator |

### Build Options

| Option | Default | Purpose |
|--------|---------|---------|
| `USE_ASAN` | OFF | AddressSanitizer |
| `USE_COVERAGE` | OFF | Code coverage instrumentation |

### Quick Build

```bash
# Development build
cmake -B build_tmp -DCMAKE_BUILD_TYPE=Debug
cmake --build build_tmp -j$(nproc)

# Release build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

Vendored dependencies: toml11, nlohmann/json. NVPTX target enabled for GPU support. AVX-512/AVX2 auto-detected for high-precision SIMD.

---

## Source Map

### Directory Structure with Line Counts

```
src/
├── main.cpp                          1,596   Compiler driver
├── frontend/
│   ├── lexer/
│   │   ├── lexer.cpp                 1,466   Tokenizer
│   │   └── token.cpp                   429   Token utilities
│   ├── parser/
│   │   └── parser.cpp                4,646   Recursive-descent parser
│   ├── ast/
│   │   ├── ast_node.cpp                      AST base node
│   │   ├── expr.cpp                          Expression nodes
│   │   ├── stmt.cpp                          Statement nodes
│   │   └── type.cpp                          Type annotation nodes
│   ├── preprocessor/
│   │   └── preprocessor.cpp          1,415   NASM-style macros
│   └── sema/
│       ├── type_checker.cpp         10,615   Type inference & checking
│       ├── borrow_checker.cpp        2,422   Ownership & lifetimes
│       ├── const_evaluator.cpp       1,469   Compile-time evaluation
│       ├── generic_resolver.cpp      1,066   Generic instantiation
│       ├── type.cpp                  1,148   Internal type representation
│       ├── definite_assignment.cpp     465   Assignment analysis
│       ├── exhaustiveness.cpp          453   Pick exhaustiveness
│       ├── symbol_table.cpp            246   Symbol table
│       ├── async_analyzer.cpp          204   Async validation
│       ├── visibility_checker.cpp      150   Pub/private enforcement
│       ├── safety_checker.cpp                Safety auditing
│       ├── closure_analyzer.cpp              Capture analysis
│       ├── module_loader.cpp                 Import resolution
│       ├── module_resolver.cpp               Module path resolution
│       └── module_table.cpp                  Module registry
├── backend/
│   ├── ir/
│   │   ├── ir_generator.cpp          9,803   Main IR generation
│   │   ├── codegen_expr.cpp         10,183   Expression codegen
│   │   ├── codegen_stmt.cpp          2,929   Statement codegen
│   │   ├── tbb_codegen.cpp             467   TBB arithmetic
│   │   └── ternary_codegen.cpp         600   Ternary/nonary codegen
│   └── runtime/
│       ├── ternary_ops.cpp             821   Trit/tryte/nit/nyte ops
│       ├── ttensor_ops.cpp             730   Toroidal tensor ops
│       ├── tfp_ops.cpp                 610   Twisted float ops
│       ├── frac_ops.cpp                563   Rational arithmetic
│       ├── tmatrix_ops.cpp             514   Twisted matrix ops
│       └── vec9_ops.cpp                      9D vector ops
├── runtime/
│   ├── gc/                           ~917    Garbage collector
│   ├── allocators/                 ~2,400    Memory allocators
│   ├── strings/                    ~1,166    String runtime
│   ├── collections/                ~1,500    Dynamic arrays, maps
│   ├── io/                         ~1,904    File I/O + zero-copy
│   ├── async/                      ~3,200    Async runtime (io_uring)
│   ├── streams/                    ~1,039    Stream abstraction
│   ├── thread/                       ~898    Threading
│   ├── atomic/                     ~1,263    Atomic operations
│   ├── math/                       ~2,750    Math + LBIM + fix256
│   ├── json/                       ~1,551    JSON support
│   ├── toml/                       ~1,943    TOML support
│   ├── process/                      ~696    Process management
│   ├── net/                          ~257    Networking
│   ├── fmt/                          ~583    Formatting
│   ├── telemetry/                    ~735    Diagnostics
│   ├── assembler/                  ~1,240    JIT assembler
│   ├── panic/                        ~207    Panic handler
│   ├── result/                       ~342    Result type runtime
│   └── ...                                   Self-hosting helpers
└── tools/
    ├── lsp/                        ~1,736    Language Server
    ├── debugger/                   ~2,218    DAP debugger
    ├── pkg/                        ~1,078    Package manager
    └── doc/                        ~1,309    Doc generator

include/                              mirrors src/ with headers

Total: ~139,000 lines (102K compiler + 37K runtime)
```

### The Three Megafiles

Three files exceed 9,000 lines and are candidates for future splitting:

1. **`type_checker.cpp` (10,826 lines)** — Contains type inference, generic resolution, operator checking, module loading, and error reporting all in one file.
2. **`codegen_expr.cpp` (10,531 lines)** — Expression code generation for every expression type, plus UFCS, pipeline, cast, and arithmetic logic.
3. **`ir_generator.cpp` (10,084 lines)** — Module initialization, function generation, extern blocks, debug info emission, and top-level orchestration.

These files work correctly but are difficult to navigate. Future refactoring may split them along logical boundaries (e.g., separating generic resolution from type checking, or splitting arithmetic codegen from call codegen).
