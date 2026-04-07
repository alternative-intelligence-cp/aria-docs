---
title: "Aria Programming Manual"
subtitle: "Version 1.1 — Based on Aria v0.16.11"
date: "April 2026"
author: "Alternative Intelligence"
geometry: "margin=1in"
toc: true
toc-depth: 3
numbersections: true
colorlinks: true
linkcolor: blue
urlcolor: blue
header-includes:
  - \usepackage{fancyhdr}
  - \pagestyle{fancy}
  - \fancyhead[L]{Aria Programming Manual}
  - \fancyhead[R]{v1.1}
  - \usepackage{listings}
  - \lstset{basicstyle=\ttfamily\small,breaklines=true,breakatwhitespace=true}
  - \usepackage{xurl}
---


# Part I: Quick Reference

## Language Specification Cheat Sheet

```
### Mandatory Functions ########################################
func:main = int32(argc,argv)     ** program entry, must exit() **
func:failsafe = int32(tbb32:err) ** safety net, must exit(>0) **

### Keywords ###################################################
return Result<T>        ** exit function with Result<T> value **
ok maybeUnknownVal      ** allow passing unknown **
drop myFunc()           ** bypass Result system and drop Result **
raw myFunc()            ** bypass Result system and return Result.value **
defaults value          ** default if error in chain **
fall label              ** used in pick for fallthrough **
pass T:retVal           ** return Result<T> with value **
fail tbb8:errCode       ** return Result<T> with error **
exit int32:errCode      ** exit program (main/failsafe) **
wild                    ** unmanaged memory **
defer                   ** guarantees cleanup code runs when scope exits **
async                   ** declare function async **
const                   ** constant,  only in extern blocks **
fixed                   ** Aria version of const **
use                     ** import modules/files **
mod                     ** define module **
pub                     ** public visibility **
extern                  ** external C functions **
stack                   ** explicit stack allocation **
gc                      ** explicit GC allocation **
wildx                   ** executable memory allocation (for JIT) **
requires                ** Design by Contract: function preconditions **
ensures                 ** Design by Contract: function postconditions **
invariant               ** Design by Contract: loop invariants **
result                  ** return value in ensures clause **
limit<Rules>            ** value constraint checks **
break                   ** exit innermost loop (while, for, till, loop) **
continue                ** skip to next loop iteration **
macro                   ** compile-time macro **
comptime                ** compile-time evaluation **
await                   ** suspend execution until async operation completes **
catch                   ** error/exception handling keyword **
in                      ** membership test in for loops and collections **
as                      ** alias in imports: use "mod.aria".* as alias **
cfg                     ** conditional compilation attribute **
inline                  ** suggest function inlining to compiler **
noinline                ** prevent function inlining **
trait                   ** define a trait (interface) **
impl                    ** implement trait for type **
derive                  ** auto-derive trait impls **
opaque                  ** opaque type declaration (hides internal representation) **
Type                    ** generic/existential constraint **
Rules                   ** compile-time constraint ruleset used with limit<Rules> **
move                    ** explicit move semantics (transfer ownership) **
prove                   ** formal verification assertion (checked by Z3 backend) **
assert_static           ** compile-time assertion **
relaxed                 ** memory ordering: no synchronization guarantees **
acquire                 ** memory ordering: acquire **
release                 ** memory ordering: release **
acq_rel                 ** memory ordering: acq+rel **
seq_cst                 ** memory ordering: seq cst **
stream                  ** reserved: stream types **
pipe                    ** reserved keyword for pipe/IPC types **
process                 ** reserved keyword for process types **
debug                   ** reserved keyword for debug output mode **

### Builtin Helpers ############################################
astack(capacity?)       ** init scope stack (default 256) **
apush(handle,value)     ** push value onto stack **
apop(handle)            ** pop top value from stack **
apeek(handle)           ** peek at top value **
acap(handle)            ** return size of stack in bytes of scope stack **
asize(handle)           ** return number of bytes used in scope stack **
afits(handle,val)       ** true if val fits on stack **
atype(handle)           ** type of item on top of stack **
ahash(cap)              ** create hash table, cap bytes **
ahsize(handle)          ** hash table size in bytes **
ahget(handle, key)      ** get value at key (or NIL) **
ahset(handle, key, val) ** set key to value **
ahcount(handle)         ** number of keys in table **
ahfits(handle, val)     ** 1 if val fits, 0 otherwise **
ahtype(handle, key)     ** type tag at key, -1 if none **
sys(CONST, args...)     ** safe syscall, Result<int64> **
sys!!(CONST, args...)   ** full syscall, Result<int64> **
sys!!!(expr, args...)   ** raw syscall, bare int64 **

### Special Values #############################################
ERR                     ** Tbb type sticky error sentinel **
NIL                     ** No value / nothing, Aria equivalent of void **
NULL                    ** No reference / no address / 0x00 **
unknown                 ** special sentinel for undefined **

### Types ######################################################
Result<T>               ** Result type, all functions **
any                     ** Aria equivalent of C void* **
void                    ** in extern blocks only **
int[1..4096]            ** signed integers **
uint[1..4096]           ** unsigned integers **
tbb[8,16,32,64]                                     ** twisted balanced binary ** 
flt[32,64,128,256,512]                              ** floating point **
tfp[32,64]                                          ** twisted floating point **
fix256                                              ** fixed point 128.128 **
fix256<[Joules,Meters,...]>  ** dimensional types **
frac[8,16,32,64]        ** fraction type **
bool                                                ** boolean **
vec[2,3,9]              ** vector types **
dyn                     ** trait object dispatch **
obj                     ** dynamic dispatch values **
struct                                              ** standard struct **
enum                                                ** standard enum **
string                                              ** Aria string **
func                                                ** function defintion **
array                                               ** standard array **
trit                                                ** ternary equivalent to bit **
tryte                                               ** ternary equivalent to byte **
nit                                                 ** nonary equivalent to bit **
nyte                                                ** nonary equivalent to byte **
tensor                                              ** tensor type **
ttensor                                             ** ternary tensor type **
matrix                                              ** matrix type **
tmatrix                                             ** ternary matrix type **
binary                                              ** binary data type **
buffer                                              ** memory buffer type **
complex<T>              ** complex number type **
atomic<T>               ** lock-free atomic type **
simd<T,N>               ** SIMD vector type **
Handle<T>               ** safe arena pointer **

### Control Flow ################################################
if/else                 ** standard if/else construct **
while                   ** standard while loop for advanced uses **
for                     ** standard for for advanced uses **
till(limit,step)        ** loop with automatic iteration tracking **
loop(start,limit,step)  ** loop with automatic iteration tracking **
when/then/end           ** better while with state tracking **
pick                    ** Aria equivalent of switch/case constructs **

### Operators ##################################################
+                       ** add **
-                       ** subtract **
*                       ** multiply (also ptr in extern) **
/                       ** divide **
%                       ** modulo **
++                      ** increment **
--                      ** decrement **
=                       ** assignment **
==                      ** equality check **
<=>                     ** spaceship, comparison that returns -1/0/1 **
is                      ** ternary: is a > b : T : F **
+=                      ** add and assign **
-=                      ** subtract and assign **
*=                      ** multiply and assign **
/=                      ** divide and assign **
%=                      ** modulo and assign **
!                       ** logical not **
||                      ** logical or **
&&                      ** logical and **
&                       ** bitwise and **
|                       ** bitwise or ** 
^                       ** bitwise xor **
~                       ** bitwise not **
<<                      ** bitwise left shift **
>>                      ** bitwise right shift **
!!!                     ** failsafe(err) shorthand, example: '!!! err' **
->                      ** pointer-to / member access **
<-                      ** dereference (value FROM pointer) **
@                       ** address of **
=>                      ** cast **
$                       ** iteration var / Rules value **
<T>?                    ** Optional as in int64? **
?                       ** Safe unwrap ** 
??                      ** Null coalesce **
?!                      ** Emphatic unwrap, calls failsafe if error **
?.                      ** Safe navigation**
?|                      ** defaults shorthand **
_?                      ** drop shorthand **
_!                      ** raw shorthand **
|>                      ** Pipe forward **
<|                      ** Pipe backward **
..                      ** inclusive range **
...                     ** exclusive range **
:                       ** annotation in types, separator in ternary clause **
::<T>                   ** turbofish type annotation **
#                       ** pin **
$$i                     ** immutable borrow **
$$m                     ** mutable borrow **
""                      ** string literal **
''                      ** char literal **
``                      ** template literal **
&{ }                    ** template interpolation **
//                      ** line comment **
/*                      ** begin block comment **
*/                      ** end block comment **
\                       ** escape **

### Aria STREAMS ###############################################
stdout                  ** text output stream **
stderr                  ** error output stream **
stddbg                  ** debug output stream **
stdin                   ** text input stream **
stddati                 ** data input stream **
stddato                 ** data output stream **

### Compiler Flags (ariac) #####################################

## Output
-o <file>               ** write output to <file> **
--emit-llvm             ** emit LLVM IR text (.ll) **
--emit-llvm-bc          ** emit LLVM bitcode (.bc) **
--emit-asm              ** emit assembly (.s) **
--emit-deps             ** emit JSON dependency manifest (for aria_make) **
--emit-ptx              ** emit PTX assembly for GPU execution **
--emit-wasm             ** compile to WebAssembly (.wasm) **

## Compilation Modes
-c                      ** compile library (no failsafe required) **
--shared                ** compile to shared library (.so) **
--static                ** link as fully static executable **

## Optimization & Debug
-O<0-3>                 ** optimization level (0=none, 3=max) **
-g                      ** emit DWARF debug info (for aria-dap) **
-v, --verbose           ** verbose compiler output **

## Verification (Z3 SMT Solver)
--verify                ** Z3 verification of Rules/limit **
--verify-report         ** emit detailed proof results (implies --verify) **
--verify-contracts      ** verify requires/ensures function contracts **
--verify-overflow       ** verify integer arithmetic cannot overflow **
--verify-concurrency    ** detect data races and deadlocks in concurrent code **
--verify-memory         ** detect use-after-free bugs in wild pointer code **
--smt-opt               ** SMT-guided optimizations **
--smt-timeout=N         ** Z3 timeout in ms (default 5000) **
--prove-report          ** Proven/Disproven per check **

## GPU Target (NVIDIA CUDA/PTX)
--target=<arch>         ** cpu, gpu, gpu+cpu, wasm32-* **
--gpu-arch=<sm>         ** CUDA CC (default sm_50) **
--gpu-opt=<0-3>         ** GPU optimization level (default: 3) **
--gpu-debug             ** embed debug info in PTX **

## Input & Linking
-I <dir>                ** add directory to module search path **
-l<library>             ** link against library (e.g., -lm, -lpthread) **
-L<path>                ** add library search path **
-Wl,<option>            ** pass option to linker **

## Warnings
-Wall                   ** enable all warnings **
-Werror                 ** treat warnings as errors **
-W<warning>             ** enable specific warning **
-Wno-<warning>          ** disable specific warning **

## Diagnostics
--ast-dump              ** dump AST and exit **
--tokens                ** dump tokens and exit **
-E                      ** preprocess only (output to stdout) **

## General
--version               ** show version **
--help, -h              ** show help text **

### MORE INFO ##################################################
aria-docs/guide```

\newpage

## Version History

| Version | Aria | Key Changes |
|---------|------|-------------|
| v1.1 | v0.16.11 | Layout fixes, content update |
| v1.0 | v0.13.7 | Initial release |

### Since v1.0

- **v0.14.x**: Expanded Z3 SMT solver coverage, bitvector-accurate
  proofs, Rules consistency checking, solver-guided optimizations
- **v0.15.x**: Self-hosting foundation — 5 compiler modules ported
  to Aria (3,070 lines), C-bridge FFI shim pattern
- **v0.16.x**: Code review series — dead code removal (~46K lines),
  TODO/FIXME audit, debug cleanup, guide and example fixes

\newpage

# Part II: Programming Guide


## Types


##### Dynamic Type — `any`

#### Overview

`any` is Aria's dynamically-typed value container. It can hold any type, with the actual
type tracked at runtime. Internally implemented as a fat pointer `{ptr, i64}` with a type tag.

> **Note:** `dyn` is NOT an alias for `any`. `dyn` is used exclusively for trait object
> dispatch (`dyn TraitName` in function parameters). See [traits](../advanced_features/traits.md).

#### Declaration

```aria
any:box = 42i64;
```

#### Access Methods

Use turbofish syntax to read, write, and resolve `any` values:

```aria
// Read with get::<T>()
int64:val = box.get::<int64>();

// Write with set::<T>(value)
box.set::<int64>(100i64);

// Resolve — consuming transform to concrete pointer
int64->:ptr = box.resolve::<int64>();
```

#### Casting

`any` does not use `as` for casting. Aria has three cast forms:

```aria
// Infix arrow cast
int32:num = some_value => int32;

// Checked builtin
int32:num = @cast<int32>(some_value);

// Unchecked builtin (wraps/truncates on overflow)
int8:truncated = @cast_unchecked<int8>(large_value);
```

For `any`, use `.get::<T>()` or `.resolve::<T>()` to extract the concrete value.

#### Heterogeneous Collections

```aria
any[5]:items;
items[0] = 42i64;
items[1] = "hello";
```

#### Performance

Dynamic type checking has runtime overhead. **Prefer static types** wherever possible.
Use `any` only when truly needed (e.g., heterogeneous containers, plugin interfaces).

#### Related

- [struct.md](struct.md) — static composite types
- [result.md](result.md) — type-safe error handling


##### Array — `array`

#### Overview

Standard contiguous arrays with compile-time sizing.

#### Declaration

```aria
int32[5]:numbers;
numbers[0] = 10i32;
numbers[1] = 20i32;
numbers[2] = 30i32;
numbers[3] = 40i32;
numbers[4] = 50i32;
```

#### Access

```aria
int32:first = numbers[0];     // zero-indexed
int32:last = numbers[4];
if (numbers[2] != 30i32) {
    // handle error
}
```

#### Slicing

```aria
int64[5]:nums;
int64[]:slice = nums[0..4];    // inclusive range
int64[]:slice2 = nums[0...2];  // exclusive range
```

#### Related

- [types/vec.md](vec.md) — fixed-size vector types
- [collections/astack.md](../collections/astack.md) — user stack (LIFO)
- [collections/ahash.md](../collections/ahash.md) — hash tables


##### Atomic Types — `atomic<T>`

#### Overview

`atomic<T>` provides lock-free concurrent access to a wrapped value. Lock-free when
T ≤ 64 bits; locked (mutex-backed) for larger types.

#### Declaration

Two forms are supported:

```aria
stack atomic<int32>:counter;                       // uninitialized stack allocation
atomic<int32>:counter = atomic_new(0i32);           // initialized with value
stack atomic<bool>:flag;                            // boolean atomic
```

**Type constraint:** `atomic<T>` requires lock-free compatible types:
`int8`–`int64`, `uint8`–`uint64`, `bool`, `tbb8`–`tbb64`.
Floats, strings, and arrays are **rejected** at compile time.

#### Memory Ordering

Default ordering is **Sequential Consistency** (SeqCst) — the safest option.
Weaker orderings require `unsafe { }`:

| Ordering | Safety | Use Case |
|----------|--------|----------|
| SeqCst | Safe (default) | General purpose |
| Acquire/Release | Requires unsafe | Producer/consumer patterns |
| Relaxed | Requires unsafe | Counters where ordering doesn't matter |

#### Operations

```aria
int32:val = counter.load();                         // read
counter.store(42i32);                               // write
int32:old = counter.swap(100i32);                   // exchange
bool:ok = counter.compare_exchange(99i32, 100i32);  // CAS
int32:prev = counter.fetch_add(1i32);               // atomic increment
int32:prev = counter.fetch_sub(1i32);               // atomic decrement
```

Weak CAS (`compare_exchange_weak`) may spuriously fail — use in a loop.

#### ERR Sentinel

The TBB ERR sentinel value remains atomic — ERR propagation works through atomic operations.

#### Status

Basic operations (load, store, swap, CAS, fetch_add, fetch_sub) are implemented.
Advanced memory orderings are in progress.

#### Related

- [advanced_features/concurrency.md](../advanced_features/concurrency.md) — threads, channels
- [tbb.md](tbb.md) — TBB error propagation through atomics


##### Boolean — `bool`

#### Overview

Standard boolean type storing `true` or `false`. 1 byte storage. Default value: `false`.

#### Declaration

```aria
bool:is_ready = true;
bool:done = false;
```

#### Logical Operations

```aria
bool:a = true;
bool:b = false;
bool:and_r = (a && b);   // false — short-circuit AND
bool:or_r  = (a || b);   // true  — short-circuit OR
bool:not_r = (!a);        // false — logical NOT
```

Both `&&` and `||` use **short-circuit evaluation** — the right operand is skipped if the
left operand determines the result.

#### Important

**No implicit int→bool conversion.** This is not C:

```aria
// WRONG: bool:flag = 1;     // compile error
// RIGHT: bool:flag = true;
```

#### Style

```aria
// DON'T: if (is_ready == true) { ... }
// DO:    if (is_ready) { ... }
```


##### Complex Numbers — `complex<T>`

#### Overview

Generic complex number type: `complex<T>` where T is the component type.

```
complex<T> = { T:real, T:imag }
```

Common instantiations: `complex<flt64>`, `complex<fix256>`, `complex<tbb64>`.

#### Declaration

```aria
complex<flt64>:z = { real = 3.0, imag = 4.0 };  // 3 + 4i
```

#### Arithmetic

Standard complex arithmetic: `+`, `-`, `*`, `/` with proper mathematical formulas.

```aria
complex<flt64>:a = { real = 1.0, imag = 2.0 };
complex<flt64>:b = { real = 3.0, imag = 4.0 };
complex<flt64>:sum = (a + b);  // {4.0, 6.0}
```

- `conjugate()` — implemented
- `magnitude()` — future
- `phase()` — future

#### ERR Propagation

If either component is ERR (when using TBB component types), the entire complex
number is tainted.

#### SIMD

Interleaved layout for SIMD: `[real₀, imag₀, real₁, imag₁, ...]`

#### Status

**[WARN] Not yet implemented in the compiler.** Consider the `aria-entangled` package.

#### Related

- [flt.md](flt.md) — float component types
- [fix256.md](fix256.md) — deterministic component type
- [tbb.md](tbb.md) — error-propagating component type


##### Enum

#### Overview

Enumerations define a set of named integer constants. Backed by `int64` at the ABI level.
Since v0.2.39.

#### Declaration

```aria
enum:Color = { RED, GREEN, BLUE };                        // auto-numbered: 0, 1, 2
enum:HttpStatus = { OK = 200, NOT_FOUND = 404, ERROR = 500 };  // explicit values
enum:Mixed = { A, B = 10, C };                            // C = 11 (continues from last)
```

#### Usage

```aria
Color:my_color = Color.RED;
int64:val = Color.GREEN;     // enums convert to int64

pick (my_color) {
    (Color.RED)   { drop println("Red"); },
    (Color.GREEN) { drop println("Green"); },
    (Color.BLUE)  { drop println("Blue"); },
    (*) {}
}
```

Enum comparison:

```aria
Color:a = Color.RED;
Color:b = Color.BLUE;
if (a != b) {
    drop println("Different colors");
}
```

#### Rules

- Variant names must be unique within the enum
- Values must be valid `int64`
- Convention: variant names in UPPER_CASE
- Declared at top level (not inside functions)
- Supports `==` and `!=` comparison
- Enum-typed variables enforce same-enum assignment
- Integrates with `pick` statement exhaustiveness checking

#### Related

- [struct.md](struct.md) — composite types
- [control_flow/pick.md](../control_flow/pick.md) — switch/case with exhaustiveness


##### Exotic Number Types

#### Balanced Ternary

| Type | Description | Notes |
|------|-------------|-------|
| `trit` | Ternary digit: -1, 0, +1 | ~2 bits packed |
| `tryte` | 9 trits | Ternary byte equivalent |

Trit values: `T` (−1), `0` (0), `1` (+1).

#### Balanced Nonary

| Type | Description | Notes |
|------|-------------|-------|
| `nit` | Nonary digit: -4 to +4 | Base-9 unit |
| `nyte` | Collection of nits | Nonary byte equivalent |

#### Status

Basic initialization and comparison operations compile and pass tests:

```aria
trit:a = 1;
trit:b = 0;
tryte:ta = 1;
nit:n1 = 3;
nyte:ny1 = 7;
```

These types are functional for basic operations. Advanced arithmetic is limited.

#### Related

- [tbb.md](tbb.md) — twisted balanced binary (production error-propagating type)
- [int.md](int.md) — standard binary integers


##### Fixed Point — `fix256`

#### Overview

`fix256` is a **Q128.128 deterministic fixed-point type**: 128-bit integer part + 128-bit
fractional part = 256 bits total. It provides bit-exact identical results across all
platforms (x86-64, ARM64, RISC-V, CUDA).

#### Specifications

| Property | Value |
|----------|-------|
| Total size | 256 bits (32 bytes) |
| Integer part | 128 bits (signed) |
| Fractional part | 128 bits |
| Precision | 2^-128 ≈ 2.9×10^-39 |
| Range | ±2^127 ≈ ±1.7×10^38 |
| Storage | 4 × 64-bit limbs, `limb[3]` is MSB with sign |
| Deterministic | Yes — zero drift, bit-exact across platforms |

#### Declaration

```aria
fix256:a = fix256_from_int(10);        // from integer
fix256:b = fix256_from_float(3.0);     // from float
```

#### Arithmetic

```aria
fix256:sum  = a + b;
fix256:diff = a - b;
fix256:prod = a * b;
fix256:quot = a / b;
if (a != b) { /* comparison works */ }
```

#### Conversion Back

```aria
int64:val = fix256_to_int(sum);
```

#### Dimensional Analysis Variants

`fix256` supports dimensional type annotations for physics/engineering:

```aria
fix256<Joules>:energy = fix256(100);
fix256<Meters>:distance = fix256(50);
fix256<Seconds>:time = fix256(10);
fix256<Newtons>:force = fix256(25);
fix256<Kelvin>:temp = fix256(300);
```

The compiler enforces dimensional consistency — you cannot add Joules to Meters.

#### Why Not Floats?

- **Float problem**: `0.1 + 0.2 ≠ 0.3` (IEEE 754 representation error)
- **Float problem**: Results differ across platforms (x87 vs SSE, ARM vs x86)
- **Float problem**: Non-associative: `(a + b) + c ≠ a + (b + c)`
- **fix256**: Same bits in → same bits out, every time, every platform

#### Use Cases

- Safety-critical systems (aerospace, medical, nuclear)
- Financial calculations requiring exact decimal representation
- Deterministic physics simulation
- Cross-platform reproducible scientific computation
- Post-quantum cryptographic operations

#### Related

- [flt.md](flt.md) — IEEE 754 floating point (faster but non-deterministic)
- [frac.md](frac.md) — exact rational arithmetic
- [int.md](int.md) — LBIM integers used internally


##### Floating Point — `flt`

**Widths:** `flt32`, `flt64`, `flt128`, `flt256`, `flt512`

#### Overview

IEEE 754 floating-point types for real number arithmetic. `flt64` is recommended for
general use. For deterministic fixed-point arithmetic, see [fix256.md](fix256.md).

#### Width Table

| Type | Bytes | Precision | Range (approx) | Alias | Notes |
|------|-------|-----------|-----------------|-------|-------|
| flt32 | 4 | ~7 decimal digits | ±3.4×10^38 | f32 | |
| flt64 | 8 | ~15 decimal digits | ±1.8×10^308 | f64 | Recommended |
| flt128 | 16 | ~33 decimal digits | ±1.2×10^4932 | f128 | LBIM |
| flt256 | 32 | ~70 decimal digits | Extended | f256 | LBIM |
| flt512 | 64 | ~150 decimal digits | Extended | f512 | LBIM |

#### Declaration

```aria
flt64:pi = 3.14159265358979;
flt32:temp = 98.6f32;          // type suffix
flt64:big = 1.23e10;           // scientific notation
```

#### Special Values

- `NaN` — Not a Number (result of `0.0 / 0.0`)
- `+∞` / `-∞` — overflow results
- `+0` / `-0` — signed zero (IEEE 754)

`NaN != NaN` is **true** (standard IEEE behavior). Always compare with epsilon:

```aria
flt64:a = 0.1 + 0.2;
flt64:epsilon = 0.000001;
// DON'T: if (a == 0.3) { ... }
// DO:    if (abs(a - 0.3) < epsilon) { ... }
```

#### Conversions

```aria
flt64:from_int = x;            // int→float: safe widening
int32:from_flt = f => i32;     // float→int: truncates (no rounding)
flt64:from_f32 = small;        // flt32→flt64: safe widening
```

#### ABI Note

**`flt32` passes as `double` at the C ABI level.** If writing C shims for extern functions,
the C side must declare parameters as `double`, then cast internally. This is an Aria
codegen convention, not a bug.

#### When NOT to Use Floats

- **Money**: Use cents as `int64`, or `fix256`
- **Deterministic simulation**: Use `fix256` (bit-exact across platforms)
- **Error-propagating math**: Use `tbb` types

#### Related

- [fix256.md](fix256.md) — deterministic fixed-point
- [tfp.md](tfp.md) — twisted floating point
- [int.md](int.md) — integer types


##### Fractions — `frac`

**Widths:** `frac8`, `frac16`, `frac32`, `frac64`

#### Overview

Fraction types store exact rational numbers as mixed fractions: whole part + numerator +
denominator. Built on TBB internally, inheriting sticky ERR propagation.

#### Width Table

| Type | Component Type | Storage | Notes |
|------|---------------|---------|-------|
| frac8 | tbb8 | 3 bytes | `{tbb8:whole, tbb8:num, tbb8:denom}` |
| frac16 | tbb16 | 6 bytes | |
| frac32 | tbb32 | 12 bytes | |
| frac64 | tbb64 | 24 bytes | |

#### Representation

A frac value `{whole, num, denom}` represents `whole + num/denom`:

- `{1, 1, 3}` = 1⅓ (exactly)
- `{0, 1, 3}` = ⅓ (exactly — not 0.333...)
- `{0, 0, 1}` = 0

#### Canonical Form Invariants

After every operation, fractions auto-normalize to canonical form:
- Denominator > 0
- Numerator ≥ 0 when whole ≠ 0
- Numerator < denominator
- Reduced to lowest terms (GCD applied)

#### Why Fractions?

Floating point cannot represent ⅓ exactly. Fractions can:

```
flt64:  1/3 = 0.333333333333... (truncated, drift accumulates)
frac32: 1/3 = {0, 1, 3}        (exact, zero drift)
```

#### Status

`frac8`, `frac16`, and `frac32` compile with `{whole, num, denom}` initializer syntax:

```aria
frac8:half = {0, 1, 2};       // 0 + 1/2
frac16:third = {0, 1, 3};     // 0 + 1/3
frac32:mixed = {1, 1, 3};     // 1 + 1/3
```

`frac64` can be declared but does not yet support initialization.

#### Related

- [tbb.md](tbb.md) — underlying component type
- [fix256.md](fix256.md) — deterministic fixed-point alternative
- [flt.md](flt.md) — IEEE floating point


##### Signed Integers — `int`

**Widths:** `int1`, `int2`, `int4`, `int8`, `int16`, `int32`, `int64`,
`int128`, `int256`, `int512`, `int1024`, `int2048`, `int4096`

#### Overview

Aria provides two's complement signed integers in powers of 2 from 1-bit to 4096-bit.
`int32` (alias `i32`) is the **default integer type** — bare literals like `100` infer as `int32`.

#### Width Table

| Type | Bytes | Range | Alias | Notes |
|------|-------|-------|-------|-------|
| int1 | 1 | -1 to 0 | i1 | Sign bit only |
| int2 | 1 | -2 to 1 | i2 | |
| int4 | 1 | -8 to 7 | i4 | |
| int8 | 1 | -128 to 127 | i8 | `abs(-128)` is undefined |
| int16 | 2 | -32768 to 32767 | i16 | |
| int32 | 4 | -2^31 to 2^31-1 | i32 | **Default**. Wraps on overflow |
| int64 | 8 | -2^63 to 2^63-1 | i64 | |
| int128 | 16 | -2^127 to 2^127-1 | i128 | LBIM — see below |
| int256 | 32 | -2^255 to 2^255-1 | i256 | LBIM |
| int512 | 64 | -2^511 to 2^511-1 | i512 | LBIM |
| int1024 | 128 | -2^1023 to 2^1023-1 | i1024 | LBIM |
| int2048 | 256 | -2^2047 to 2^2047-1 | i2048 | LBIM |
| int4096 | 512 | -2^4095 to 2^4095-1 | i4096 | LBIM |

#### Declaration

```aria
int32:x = 42;                 // explicit type
int64:big = 9999999i64;       // suffix selects width
int128:huge = 99999999999999999999i128;  // suffix required for large literals
```

#### Arithmetic

```aria
int32:a = 10;
int32:b = 3;
int32:sum  = (a + b);    // 13
int32:quot = (a / b);    // 3 (truncates toward zero)
int32:rem  = (a % b);    // 1
a++;                      // 11
b--;                      // 2
```

Integer division truncates: `1000000 / 2000000 = 0`.
Overflow wraps (two's complement).

#### Conversions

```aria
int64:wide = a;            // widening: always safe
int32:narrow = big => i32; // narrowing: truncates high bits
int32:from_flt = f => i32; // float→int: truncates (no rounding)
```

Widening from any intN to intM (M > N) is always safe. Narrowing truncates.

#### LBIM Types (≥128 bit)

Widths `int128` through `int4096` use the **Limb-Based Integral Model**:
- Stored as arrays of 64-bit limbs
- **Slower than int64** — use only when necessary
- Passed via `sret`/`byval` at the ABI level (not in registers)
- Type suffix required for literals: `42i128`, `99i1024`
- Use cases: cryptography, post-quantum key material, UUID storage, high-precision financial

#### ABI Notes

| Width | ABI Passing |
|-------|-------------|
| int1–int64 | Register (zero/sign-extended) |
| int128+ | Pointer (sret/byval) |

#### Related

- [uint.md](uint.md) — unsigned integers
- [tbb.md](tbb.md) — twisted balanced binary (error-propagating integers)
- [fix256.md](fix256.md) — fixed-point using LBIM


##### Pointers — `wild`, `wildx`, `->`, `@`

#### Overview

Aria has three memory modes. Pointers are only directly used in `wild` (unmanaged) mode
and `extern` blocks.

#### Pointer Operators

| Operator | Meaning |
|----------|---------|
| `@` | Address-of (get pointer to variable) |
| `->` | In type: pointer to T. In expression: member access through pointer |
| `<-` | Dereference (get value from pointer) |
| `?.` | Safe navigation (NIL-safe member access) |

#### Wild Pointers

```aria
wild int32:x = 42;          // unmanaged allocation
int32->:ptr = @x;           // pointer to x
int32:val = <-ptr;           // dereference
```

`wild` memory is **not garbage collected**. You must manage it manually or use `defer`
for cleanup.

#### Executable Memory — `wildx`

`wildx` allocates executable memory for JIT compilation:

```aria
wildx uint8->:code = wildx_alloc(4096);  // 4KB executable page
```

#### Safe Alternative: Handle<T>

For arena-allocated data, prefer `Handle<T>` over raw pointers. See
[memory_model/handle.md](../memory_model/handle.md).

#### Related

- [memory_model/wild.md](../memory_model/wild.md) — wild allocation mode
- [memory_model/handle.md](../memory_model/handle.md) — generational handles
- [functions/extern.md](../functions/extern.md) — FFI pointer patterns


##### Result<T> — Mandatory Error Handling

#### Overview

**Every function in Aria returns `Result<T>`** (except extern functions). The compiler
forces you to handle the result — unhandled Results are compile errors. This is the
core of Aria's safety model at the function level.

#### Structure

```
Result<T> = { T|NULL:value, tbb8:error, bool:is_error }
```

- Success: `value` is set, `error` is NIL, `is_error` is false
- Error: `value` is NIL, `error` is set, `is_error` is true

#### Creating Results

Inside functions (not `main` or `failsafe`):

```aria
func:divide = flt64(int32:a, int32:b) {
    if (b == 0) {
        fail 1;              // return error Result
    }
    pass (a / b);            // return success Result
}
```

- `pass value` — return success, wraps value in Result
- `fail errcode` — return error, wraps tbb8 error code in Result
- Both support paren syntax: `pass(value)`, `fail(errcode)`

#### Handling Results

```aria
// Safe unwrap with fallback
flt64:val = divide(10, 3) ? 0.0;

// Null coalesce
flt64:val = divide(10, 0) ?? default_value;

// Emphatic unwrap — calls failsafe on error
flt64:val = divide(10, 3)?!;

// Defaults keyword — scoped fallback for chains
flt64:val = a() + b() + c() defaults 0.0;
// Shorthand: flt64:val = a() + b() + c() ?| 0.0;

// Drop (discard result entirely)
drop divide(10, 3);
// Shorthand: _? divide(10, 3);

// Raw (extract value, ignore error)
flt64:val = raw divide(10, 3);
// Shorthand: flt64:val = _! divide(10, 3);

// Discard assignment
_ = divide(10, 3);
```

#### Operator Summary

| Operator | Keyword | Meaning |
|----------|---------|---------|
| `?` | — | Safe unwrap with fallback value |
| `??` | — | Null coalesce |
| `?!` | — | Emphatic unwrap (failsafe on error) |
| `?\|` | `defaults` | Scoped fallback for expression chains |
| `_?` | `drop` | Discard Result entirely |
| `_!` | `raw` | Extract value, bypass error checking |

#### Auto-Unwrap in Argument Position

When passing a Result<T> as an argument to a function expecting T, it auto-unwraps:

```aria
func:add = int32(int32:a, int32:b) { pass (a + b); }
func:get_val = int32() { pass 42; }

// get_val() returns Result<int32>, but add() expects int32
// Result auto-unwraps in argument position
int32:sum = raw add(get_val(), get_val());
```

This does **not** apply to variable assignment — you must explicitly unwrap.

#### The `ok` Keyword

`ok` allows passing a variable whose value may be `unknown`:

```aria
ok maybe_unknown_value;
```

#### main and failsafe

`main` and `failsafe` are special — they use `exit` instead of `pass`/`fail`:

```aria
func:main = int32() {
    exit 0;   // exit program with code 0
}

func:failsafe = int32(tbb32:err) {
    exit 1;   // must exit with code > 0
}
```

#### Related

- [tbb.md](tbb.md) — TBB error propagation (value-level)
- [functions/main_failsafe.md](../functions/main_failsafe.md) — mandatory functions
- [functions/result_system.md](../functions/result_system.md) — pass/fail patterns


##### SIMD Types — `simd<T,N>`

#### Overview

Generic SIMD (Single Instruction, Multiple Data) vector type where T is the element type
and N is the lane count (must be a power of 2).

```aria
simd<flt32, 4>:v = simd_new(1.0f32, 2.0f32, 3.0f32, 4.0f32);
simd<int32, 8>:vi = simd_splat(42);  // all lanes = 42
```

#### Lane Counts

N must be a power of 2: 2, 4, 8, 16, 32, etc.
Hardware support varies by platform:
- x86-64 SSE: 4×f32, 2×f64
- x86-64 AVX2: 8×f32, 4×f64
- x86-64 AVX-512: 16×f32, 8×f64

#### Operations

Component-wise arithmetic: `+`, `-`, `*`, `/` apply to all lanes simultaneously.

#### Related

- [vec.md](vec.md) — fixed-size vector types
- [tensor_matrix.md](tensor_matrix.md) — tensor and matrix types


##### String — `string`

#### Overview

Aria strings are **UTF-8 encoded, immutable, reference-counted, and length-tracked**
(not null-terminated internally). String operations return new strings.

#### Declaration

```aria
string:name = "Alice";
string:greeting = "Hello, world!";
string:empty = "";
```

#### String Literals

| Syntax | Type | Escape Processing |
|--------|------|-------------------|
| `"..."` | Regular string | Yes — `\n`, `\t`, `\\`, etc. |
| `` `...` `` | Raw string | No — backslashes are literal |
| `` `Hello, &{name}!` `` | Template literal | Interpolation with `&{expr}` |

#### Interpolation

Use backtick strings with `&{expression}` for interpolation:

```aria
string:name = "Aria";
int32:version = 4;
println(`&{name} version &{version}`);  // "Aria version 4"
```

#### Operations

```aria
string:full = (first + " " + last);  // concatenation
int64:len = string_length(name);      // character count (UTF-8 aware)
string:sub = text[0..5];              // substring slice (inclusive range)
```

Built-in string functions:
- `string_length(s)` — character count
- `string_contains(s, substr)` — boolean search
- `string_concat(a, b)` — concatenation (also `+` operator)

#### Performance

**Don't concatenate in loops.** String concatenation creates new strings each time.
For repeated building, collect parts in an array and join.

#### ABI Notes

This is critical for FFI:
- **Parameters**: `string` passes as `const char*` (single register, null-terminated)
- **Returns**: `AriaString {char* data, int64_t length}` by value (%rax=data, %rdx=length)
- C shims must use `const char*` for input params, `AriaString` struct for returns

#### Related

- [operators/string_ops.md](../operators/string_ops.md) — string operators
- [io_system/print.md](../io_system/print.md) — print/println


##### Struct

#### Overview

Structs are composite types that group named fields. Methods are defined via trait
implementations.

#### Declaration

```aria
struct:Point = {
    int32:x;
    int32:y;
};

struct:Person = {
    string:name;
    int32:age;
};
```

#### Instantiation and Access

Struct literal syntax with named fields:

```aria
stack Point:p = Point{x: 10, y: 20};
println(p.x);    // 10
println(p.y);    // 20

stack Person:alice = Person{name: "Alice", age: 30};
```

Positional arguments (order must match declaration):

```aria
Point:p = Point{10i32, 20i32};
```

Constructor syntax (desugars to `Type_create(args)`):

```aria
Counter:c = raw instance<Counter>(10i32);
```

Fields can also be assigned after declaration:

```aria
Point:p;
p.x = 10i32;
p.y = 20i32;
```

#### Methods (via Trait Impls)

Methods are defined through trait implementations:

```aria
trait:HasArea = {
    func:area: int32(Rect2D:self);
};

struct:Rect2D = {
    int32:w;
    int32:h;
};

impl:HasArea:for:Rect2D = {
    func:area = int32(Rect2D:self) {
        pass self.w * self.h;
    };
};

// Call via UFCS (Uniform Function Call Syntax)
Rect2D:rect;
rect.w = 10i32;
rect.h = 5i32;
Result<int32>:a = rect.area();
```

#### Passing Structs

```aria
func:add_coords = int32(Point:p) {
    pass p.x + p.y;
};

Point:origin = Point{ x: 0, y: 0 };
int32:sum = raw add_coords(origin);
```

#### Related

- [enum.md](enum.md) — enumeration types
- [memory_model/handle.md](../memory_model/handle.md) — Handle<T> for arena structs


##### Twisted Balanced Binary — `tbb`

**Widths:** `tbb8`, `tbb16`, `tbb32`, `tbb64`

#### Overview

TBB types are Aria's **error-propagating integers**. They use a symmetric range with a
dedicated ERR sentinel value. Once a TBB value becomes ERR, it stays ERR through all
subsequent operations — "sticky error propagation." This is the foundation of Aria's
safety model.

Think of ERR as **NaN for integers**, but with a single well-defined value and deterministic
comparison behavior.

#### Width Table

| Type | Bytes | Valid Range | ERR Sentinel | Notes |
|------|-------|-------------|--------------|-------|
| tbb8 | 1 | -127 to +127 | -128 (0x80) | `abs(-127) = +127` always safe |
| tbb16 | 2 | -32767 to +32767 | -32768 (0x8000) | |
| tbb32 | 4 | -2^31+1 to +2^31-1 | -2^31 (0x80000000) | |
| tbb64 | 8 | -2^63+1 to +2^63-1 | -2^63 (0x8000000000000000) | |

Note the **symmetric range** — unlike standard two's complement, the positive and negative
maximums are equal in magnitude. The minimum value is reserved as the ERR sentinel.

#### Declaration

```aria
tbb32:value = 42;
tbb8:small = 100tbb8;     // optional type suffix
tbb32:error = ERR;         // ERR literal
```

Default initial value is `0`.

#### Sticky Error Propagation

Any arithmetic involving ERR produces ERR:

```aria
tbb32:a = 10;
tbb32:b = ERR;
tbb32:c = (a + b);   // ERR — propagated from b
tbb32:d = (c * 100); // ERR — propagated from c
```

This lets you chain computations and check once at the end:

```aria
tbb32:result = step3(step2(step1(input)));
if (result == ERR) {
    // handle error — one check covers entire pipeline
}
```

#### What Triggers ERR

- Overflow (result exceeds valid range)
- Underflow (result below valid range)
- Division by zero
- Explicit assignment: `val = ERR;`
- Any operation with an ERR operand (sticky propagation)

#### ERR Comparison

Unlike IEEE NaN, ERR has well-defined comparison behavior:
- `ERR == ERR` is **true**
- ERR compares as **less than all valid values**

#### TBB vs Standard Integers

| Feature | int32 | tbb32 |
|---------|-------|-------|
| Range | -2^31 to 2^31-1 | -2^31+1 to 2^31-1 |
| Overflow | Wraps | ERR |
| Div by zero | Undefined | ERR |
| Error propagation | None | Sticky |
| abs(min) | Undefined | Valid (+max) |
| Performance | Faster | Slight overhead |

Use `tbb` when error propagation matters. Use `int` when you need maximum performance
or the full range.

#### Related

- [result.md](result.md) — Result<T> error handling (function-level)
- [int.md](int.md) — standard signed integers
- [frac.md](frac.md) — fraction types built on TBB
- [fix256.md](fix256.md) — fixed-point built on limbs


##### Tensor & Matrix Types

#### tensor

Multi-dimensional array type for numerical computing and AI workloads.

#### matrix

2D matrix type. Semantically a rank-2 tensor with matrix-specific operations.

#### Status

**[WARN] Not yet implemented.** The types are registered in the parser and can be declared,
but have no constructors or operations. Also registered as `tmatrix` and `ttensor`.

```aria
matrix:mat;   // compiles (declaration only)
tensor:tens;  // compiles (declaration only)
```

#### Related

- [vec.md](vec.md) — fixed-size vector types
- [simd.md](simd.md) — SIMD vector types


##### Twisted Floating Point — `tfp`

**Widths:** `tfp32`, `tfp64`

#### Overview

Twisted floating point types combine IEEE-like floating-point representation with
TBB-style sticky error propagation. They are to floats what TBB is to integers.

| Type | Bytes | Notes |
|------|-------|-------|
| tfp32 | 4 | Single-precision with ERR sentinel |
| tfp64 | 8 | Double-precision with ERR sentinel |

#### Status

`tfp32` and `tfp64` compile with `{exponent, mantissa}` initializer syntax:

```aria
tfp32:pi_approx = {1, 14159};
tfp64:e_approx = {2, 71828};
```

Basic initialization works. Full arithmetic operations are in progress.

#### Related

- [flt.md](flt.md) — standard floating point
- [tbb.md](tbb.md) — twisted balanced binary (integer ERR propagation)


##### Unsigned Integers — `uint`

**Widths:** `uint1`, `uint2`, `uint4`, `uint8`, `uint16`, `uint32`, `uint64`,
`uint128`, `uint256`, `uint512`, `uint1024`, `uint2048`, `uint4096`

#### Overview

Unsigned integers store non-negative values only. Required for bitwise operations
(`&`, `|`, `^`, `~`, `<<`, `>>`).

#### Width Table

| Type | Bytes | Range | Alias | Notes |
|------|-------|-------|-------|-------|
| uint1 | 1 | 0 to 1 | u1 | Single bit |
| uint2 | 1 | 0 to 3 | u2 | |
| uint4 | 1 | 0 to 15 | u4 | |
| uint8 | 1 | 0 to 255 | u8 | Byte |
| uint16 | 2 | 0 to 65535 | u16 | |
| uint32 | 4 | 0 to 2^32-1 | u32 | |
| uint64 | 8 | 0 to 2^64-1 | u64 | |
| uint128 | 16 | 0 to 2^128-1 | u128 | LBIM |
| uint256 | 32 | 0 to 2^256-1 | u256 | LBIM |
| uint512 | 64 | 0 to 2^512-1 | u512 | LBIM |
| uint1024 | 128 | 0 to 2^1024-1 | u1024 | LBIM |
| uint2048 | 256 | 0 to 2^2048-1 | u2048 | LBIM |
| uint4096 | 512 | 0 to 2^4096-1 | u4096 | LBIM |

#### Declaration

```aria
uint32:flags = 0xFF00u32;
uint64:addr = 0xDEADBEEFu64;
uint8:byte = 255u8;
```

#### Bitwise Operations

Bitwise ops require unsigned types:

```aria
uint32:a = 0xFF00u32;
uint32:b = 0x0FF0u32;
uint32:and_result = (a & b);   // 0x0F00
uint32:or_result  = (a | b);   // 0xFFF0
uint32:xor_result = (a ^ b);   // 0xF0F0
uint32:not_result = (~a);      // 0xFFFF00FF
uint32:shl = (a << 4u32);     // 0xF0000
uint32:shr = (a >> 4u32);     // 0x0FF0
```

#### LBIM Types (≥128 bit)

Same limb-based model as signed integers. See [int.md](int.md) for LBIM details.
Passed via sret/byval at ABI level.

#### Related

- [int.md](int.md) — signed integers
- [tbb.md](tbb.md) — twisted balanced binary


##### Vector Types — `vec`

**Widths:** `vec2`, `vec3`, `vec9`

#### Overview

Fixed-size vector types for 2D, 3D, and 9-component math. SIMD-optimized where possible.
Defaults to float components.

#### Width Table

| Type | Components | Size (float) | Size (double) | Use Case |
|------|-----------|--------------|---------------|----------|
| vec2 | x, y | 8 bytes | 16 bytes | 2D graphics, UV coords |
| vec3 | x, y, z | 12 bytes | 24 bytes | 3D graphics, physics |
| vec9 | 9 elements | 36 bytes | 72 bytes | 3×3 matrices, tensors |

#### Declaration

```aria
vec2:pos = vec2(1.0, 2.0);
vec3:color = vec3(0.5, 0.8, 1.0);
vec9:v = 0;                     // zero-init only (vec9 has no constructor yet)
```

Components are `flt64` by default. Access via `.x`, `.y`, `.z`:

```aria
flt64:px = pos.x;
flt64:py = pos.y;
```

#### Operations

```aria
vec2:a = vec2(1.0, 2.0);
vec2:b = vec2(3.0, 4.0);

vec2:sum  = (a + b);       // component-wise add: {4, 6}
vec2:diff = (a - b);       // component-wise sub: {-2, -2}
vec2:prod = (a * b);       // component-wise mul (Hadamard): {3, 8}
```

**`*` between two vectors is component-wise (Hadamard product), NOT dot product.**

#### Status

**Partial compiler support.** `vec2` and `vec3` constructors and arithmetic work.
`vec9` supports zero-init only — no constructor or indexing yet.

#### Related

- [simd.md](simd.md) — generic SIMD types
- [tensor_matrix.md](tensor_matrix.md) — tensor and matrix types

\newpage

## Operators


##### Arithmetic Operators

#### Binary Operators

| Operator | Operation | Example | Notes |
|----------|-----------|---------|-------|
| `+` | Addition | `(a + b)` | Also string concatenation |
| `-` | Subtraction | `(a - b)` | |
| `*` | Multiplication | `(a * b)` | Also pointer/deref in extern |
| `/` | Division | `(a / b)` | Integer division truncates toward zero |
| `%` | Modulo | `(a % b)` | Remainder after division |

#### Unary Operators

| Operator | Operation | Example |
|----------|-----------|---------|
| `++` | Increment | `x++` |
| `--` | Decrement | `x--` |

#### Compound Assignment

| Operator | Equivalent | Example |
|----------|------------|---------|
| `+=` | `x = (x + y)` | `x += 5;` |
| `-=` | `x = (x - y)` | `x -= 5;` |
| `*=` | `x = (x * y)` | `x *= 2;` |
| `/=` | `x = (x / y)` | `x /= 2;` |
| `%=` | `x = (x % y)` | `x %= 3;` |

#### Example

```aria
int32:a = 10;
int32:b = 3;
int32:sum = (a + b);     // 13
int32:quot = (a / b);    // 3 (truncates)
int32:rem = (a % b);     // 1
a++;                      // 11
a += 5;                   // 16
```

#### Integer Division

Integer division truncates toward zero: `1000000 / 2000000 = 0`.
For float division, use `flt64` operands.


##### Assignment Operator

#### Basic Assignment

```aria
int32:x = 42;            // typed declaration with assignment
x = 100;                  // reassignment
```

#### Discard Assignment

```aria
_ = some_function();      // discard Result — acknowledges unused return
```

This is one way to handle nodiscard Results. See also `drop` and `raw`.

#### Compound Assignment

See [arithmetic.md](arithmetic.md) for `+=`, `-=`, `*=`, `/=`, `%=`.
See [bitwise.md](bitwise.md) for `<<=`, `>>=`.

#### Fixed Values

```aria
fixed int32:MAX = 100;    // Aria's const (use 'fixed', not 'const')
```

`const` is reserved for extern blocks. Use `fixed` for Aria constants.

**Note:** `fixed` is specified in the language but has limited compiler test coverage.
Compute in a variable first, then assign to `fixed` if needed.


##### Bitwise Operators

**Bitwise operations require unsigned types** (`uint8`, `uint16`, `uint32`, `uint64`).

| Operator | Operation | Example |
|----------|-----------|---------|
| `&` | Bitwise AND | `(a & b)` |
| `\|` | Bitwise OR | `(a \| b)` |
| `^` | Bitwise XOR | `(a ^ b)` |
| `~` | Bitwise NOT | `(~a)` |
| `<<` | Left shift | `(a << n)` |
| `>>` | Right shift | `(a >> n)` |

#### Compound Assignment

| Operator | Example |
|----------|---------|
| `<<=` | `flags <<= 1;` |
| `>>=` | `flags >>= 1;` |

#### Example

```aria
uint32:flags = 0xFF00u32;
uint32:mask  = 0x0FF0u32;

uint32:result = (flags & mask);   // 0x0F00
uint32:all    = (flags | mask);   // 0xFFF0
uint32:diff   = (flags ^ mask);   // 0xF0F0
uint32:inv    = (~flags);         // 0xFFFF00FF
uint32:shl    = (flags << 4u32);  // 0xF0000
uint32:shr    = (flags >> 4u32);  // 0x0FF0
```

#### Common Patterns

```aria
// Set bit
flags = (flags | (1u32 << bit));

// Clear bit
flags = (flags & (~(1u32 << bit)));

// Toggle bit
flags = (flags ^ (1u32 << bit));

// Check bit
bool:is_set = ((flags & (1u32 << bit)) != 0u32);
```


##### Cast & Type Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=>` | Cast / type conversion | `val => int32` |
| `@cast<T>` | Explicit checked cast | `@cast<int32>(val)` |
| `@cast_unchecked<T>` | Unchecked cast | `@cast_unchecked<int32>(val)` |
| `::<T>` | Turbofish type annotation | `func::<int32>()` |

#### Cast Operator

```aria
flt64:f = 3.14;
int32:i = f => int32;          // truncates: 3
int64:wide = i => int64;       // widening: safe
```

#### Built-in Cast Functions

```aria
int32:val = @cast<int32>(f);              // checked
int32:val = @cast_unchecked<int32>(f);    // unchecked (no validation)
```

#### Optional Types — `<T>?`

```aria
int64?:maybe = get_value();    // may be NIL
```

#### Pin — `#`

```aria
#value;                  // pin value (prevent move)
```

#### Borrow Operators

| Operator | Meaning |
|----------|---------|
| `$$i` | Immutable borrow |
| `$$m` | Mutable borrow |

```aria
$$i int32:ref = value;         // immutable borrow
$$m int32:mut_ref = value;     // mutable borrow
```


##### Comparison Operators

| Operator | Operation | Example | Result |
|----------|-----------|---------|--------|
| `==` | Equal | `(a == b)` | `bool` |
| `!=` | Not equal | `(a != b)` | `bool` |
| `<` | Less than | `(a < b)` | `bool` |
| `>` | Greater than | `(a > b)` | `bool` |
| `<=` | Less or equal | `(a <= b)` | `bool` |
| `>=` | Greater or equal | `(a >= b)` | `bool` |
| `<=>` | Spaceship | `(a <=> b)` | `-1`, `0`, or `1` |

#### Spaceship Operator

Three-way comparison returning `-1` (less), `0` (equal), or `1` (greater).
Result type is `int64`.

```aria
int64:cmp = (a <=> b);
pick (cmp) {
    (-1i64) { println("a < b"); },
    (0i64)  { println("a == b"); },
    (1i64)  { println("a > b"); },
    (*) {}
}
```

#### Ternary Conditional — `is`

Aria uses `is` for ternary expressions:

```aria
int32:max = is (a > b) : a : b;
string:msg = is (count == 0) : "empty" : "has items";
```

Syntax: `is (condition) : true_value : false_value`

#### Float Comparison

**Never use `==` with floats.** Use epsilon comparison:

```aria
flt64:epsilon = 0.000001;
bool:equal = (abs(a - b) < epsilon);
```


##### Logical Operators

| Operator | Operation | Example |
|----------|-----------|---------|
| `&&` | Logical AND | `(a && b)` |
| `\|\|` | Logical OR | `(a \|\| b)` |
| `!` | Logical NOT | `(!a)` |

All logical operators use **short-circuit evaluation**:
- `&&` — skips right operand if left is false
- `||` — skips right operand if left is true

#### Example

```aria
bool:valid = (age > 0 && age < 150);
bool:allowed = (is_admin || has_permission);
bool:denied = (!allowed);
```

#### In Control Flow

```aria
if (is_ready && has_data) {
    process();
}

when (!done) {
    work();
} then {
    // completed normally
} end {
    // condition was false initially
}
```


##### Member Access & Navigation

| Operator | Meaning | Example |
|----------|---------|---------|
| `.` | Field access / enum member | `person.name`, `Color.RED` |
| `->` | Pointer member access | `ptr->field` |
| `?.` | Safe navigation (NIL-safe) | `ptr?.field` |

#### Field Access

```aria
stack Point:p = Point{x: 10, y: 20};
int32:px = p.x;
int32:py = p.y;
```

#### Pointer Access

```aria
stack Point->:ptr = @p;
int32:x_val = ptr->x;
ptr->x = 100;
```

#### Enum Member Access

```aria
Color:c = Color.RED;           // dot notation for enum members
int64:r = Color.RED;
```

#### Turbofish — `::<T>`

Explicit type annotation for generic functions:

```aria
int32:val = parse::<int32>("42");
```

#### Safe Navigation

```aria
string:city = employee?.address?.city;  // NIL if any step is NIL
```


##### Pipe & Range Operators

#### Pipe Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `\|>` | Pipe forward | `data \|> transform \|> output` |
| `<\|` | Pipe backward | `output <\| transform <\| data` |

Pipe forward passes the left-hand value as the first argument to the right-hand function:

```aria
result = data |> filter |> transform |> output;
// Equivalent to: output(transform(filter(data)))
```

#### Range Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `..` | Inclusive range | `1..10` (1 through 10) |
| `...` | Exclusive range | `1...10` (1 through 9) |

Used in:
- Array slicing: `arr[1..5]`
- Loop bounds
- Pattern matching

#### Failsafe Shorthand — `!!!`

```aria
!!! err;     // equivalent to calling failsafe(err)
```

#### Address-Of — `@`

```aria
int32->:ptr = @value;    // get address of value
```

#### Iterator Variable — `$`

The `$` symbol is the automatic iteration variable in `loop` and `till`:

```aria
loop(0, 10, 1) {
    println($);       // prints 0 through 9
}
```


##### Result Operators

These operators control how `Result<T>` values are handled. Every function call in Aria
returns `Result<T>` — these operators let you unwrap, propagate, or discard results.

#### Operator Table

| Operator | Keyword | Meaning | Example |
|----------|---------|---------|---------|
| `?` | — | Safe unwrap with fallback | `val = func() ? 0;` |
| `??` | — | Null coalesce | `val = func() ?? default;` |
| `?!` | — | Emphatic unwrap (failsafe on error) | `val = func()?!;` |
| `?\|` | `defaults` | Scoped fallback for chains | `val = a() + b() ?| 0;` |
| `_?` | `drop` | Discard Result entirely | `_? func();` |
| `_!` | `raw` | Extract value, bypass error | `val = _! func();` |

#### Safe Unwrap — `?`

Returns the value on success, or the fallback on error:

```aria
int32:val = get_number() ? 0;    // 0 if get_number fails
```

#### Null Coalesce — `??`

```aria
string:name = get_name() ?? "unknown";
```

#### Emphatic Unwrap — `?!`

Calls `failsafe` if the Result is an error:

```aria
int32:val = must_succeed()?!;    // aborts via failsafe on error
```

#### Defaults — `?|` / `defaults`

Provides a scoped fallback for an entire expression chain:

```aria
// Any error in a(), b(), or c() uses 0 as fallback
int32:val = a() + b() + c() defaults 0;

// Shorthand operator form
int32:val = a() + b() + c() ?| 0;

// Nested defaults
int32:val = func(a() + 2 + c(2 + d() defaults 3) defaults 2);
```

Defaults scopes to the nearest enclosing expression, not the entire statement.

#### Drop — `_?` / `drop`

Silently discards the Result (no error check, no value extraction):

```aria
drop setup();         // keyword form
_? setup();           // shorthand form
```

Only works on expressions, NOT on named variables.

#### Raw — `_!` / `raw`

Extracts the value field, ignoring any error:

```aria
int32:val = raw risky_func();   // keyword form
int32:val = _! risky_func();    // shorthand form
```

#### Paren Syntax

All keywords support optional parentheses (v0.4.6+):

```aria
drop(func());    // same as: drop func();
raw(func());     // same as: raw func();
pass(value);     // same as: pass value;
fail(code);      // same as: fail code;
exit(0);         // same as: exit 0;
```

#### Related

- [types/result.md](../types/result.md) — Result<T> type documentation
- [functions/result_system.md](../functions/result_system.md) — pass/fail patterns


##### String Operators

#### Concatenation

The `+` operator concatenates strings:

```aria
string:full = ("Hello" + ", " + "world!");
string:name = (first + " " + last);
```

Long chains (7+ operands) work correctly.

#### Interpolation — Template Literals

Use backtick strings with `&{expression}` for interpolation:

```aria
string:msg = `Hello, &{name}! You are &{age} years old.`;
string:calc = `Result: &{a + b}`;
int32:x = 42;
println(`x = &{x}`);
```

#### Comparison

Strings support `==` and `!=`:

```aria
if (name == "Alice") {
    println("Found Alice");
}
```

#### Comments (related syntax)

| Syntax | Type |
|--------|------|
| `//` | Line comment |
| `/* ... */` | Block comment |

#### Escape Sequences (in `"..."` strings)

| Escape | Character |
|--------|-----------|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Backslash |
| `\"` | Double quote |

Raw strings (`` `...` `` without `&{}`) do not process escapes.

\newpage

## Functions


##### Async Functions

#### Declaration

```aria
async func:fetch_data = string(string:url) {
    // ... async operations
    pass data;
}
```

The `async` keyword marks a function as asynchronous. Async functions return
`Result<T>` like all other functions.

#### Await

```aria
string:data = raw fetch_data("https://example.com");
```

#### Related

- [advanced_features/concurrency.md](../advanced_features/concurrency.md) — threads, channels
- [result_system.md](result_system.md) — error handling in async context


##### Closures & Lambdas

#### Function Pointer Variables

Lambda expressions are assigned to typed variables using function pointer syntax:

```aria
(int32)(int32):identity = int32(int32:x) { pass x; };

(int32)(int32, int32):add = int32(int32:a, int32:b) {
    int32:sum = a + b;
    pass sum;
};
```

The variable type is `(ReturnType)(ParamTypes)` — return type first, then parameter types.

#### Passing as Arguments

Higher-order functions accept function pointers as parameters:

```aria
func:apply = int32((int32)(int32):f, int32:x) {
    int32:result = f(x) ? -1i32;
    pass result;
};

(int32)(int32):dbl = int32(int32:x) { int32:v = x + x; pass v; };
int32:r = apply(dbl, 21i32) ? -1i32;
```

**Note:** Lambdas must be assigned to a named variable first, then passed by name.
There are no anonymous inline lambdas at call sites.

#### Related

- [declaration.md](declaration.md) — named function syntax
- [generics.md](generics.md) — generic function syntax


##### Function Declaration

#### Syntax

Aria functions use the `func:name = return_type(params)` syntax:

```aria
func:add = int32(int32:a, int32:b) {
    pass (a + b);
}

func:greet = NIL(string:name) {
    println(`Hello, &{name}!`);
    pass NIL;
}
```

#### Key Rules

- Function name follows `func:` — e.g., `func:calculate`
- Return type comes before the parameter list
- Parameters use `type:name` syntax
- `NIL` return type for functions that return nothing
- NIL-returning functions must end with `pass NIL;`
- ALL functions return `Result<T>` implicitly (except extern functions)

#### Calling Functions

```aria
int32:sum = raw add(10, 20);           // raw unwrap
int32:safe = add(10, 20) ? 0;          // safe unwrap with fallback
drop greet("Alice");                    // discard Result
```

#### Visibility

```aria
pub func:public_func = int32() {      // visible to other modules
    pass 42;
}

func:private_func = int32() {          // file-private (default)
    pass 99;
}
```

#### Type Inference for Literals

Bare integer literals default to `int32`. Use suffixes for other widths:

```aria
func:process = NIL(int64:val) { pass(NIL); }
// WRONG: process(42);       // 42 is int32, function wants int64
// RIGHT: process(42i64);    // explicit int64 suffix
```


##### Design by Contract

#### Overview

Aria supports formal contracts on functions: `requires` (preconditions), `ensures`
(postconditions), and `invariant` (loop invariants). These are checked at compile time
where possible and at runtime otherwise.

#### Requires — Preconditions

```aria
func:divide = flt64(int32:a, int32:b)
    requires b != 0
{
    pass (a / b);
}
```

Multiple conditions are comma-separated:

```aria
func:clamp_to_byte = int32(int32:val)
    requires val >= 0, val <= 255
{
    pass val;
}
```

#### Ensures — Postconditions

```aria
func:check_order = int32(int32:lo, int32:hi)
    requires lo >= 0, hi > lo
    ensures hi > 0
{
    pass hi - lo;
}
```

Contract clauses go between the closing `)` of the signature and the opening `{` of
the body. Compiled with `--verify-contracts` flag (uses Z3 SMT solver).

#### Invariant

Loop invariants are specified in the language design but have limited compiler test
coverage.

#### Related

- [advanced_features/rules.md](../advanced_features/rules.md) — refinement types with `limit<>`
- [functions/declaration.md](declaration.md) — function syntax


##### Extern — FFI with C

#### Overview

`extern` blocks declare C functions callable from Aria. Extern functions do **not**
return `Result<T>` — they return raw values.

#### Syntax

```aria
extern "libm" {
    func:sin = double(double:x);
    func:cos = double(double:x);
    func:sqrt = double(double:x);
}
```

Or flat (no library block):

```aria
extern func:custom_func = int32(int32:a, int32:b);
```

#### Rules

1. **Extern blocks must be at file scope** — not inside functions
2. `extern "lib" { }` blocks have a limit of **≤7** declarations
3. Flat `extern func:` declarations have no per-file limit
4. Use `void` for no-return (not NIL): `func:exit = void(int32:code);`
5. Use `const` (not `fixed`) inside extern blocks

#### String ABI

| Direction | Aria Type | C Type |
|-----------|-----------|--------|
| Parameter | `string` | `const char*` |
| Return | `string` | `AriaString {char* data, int64_t length}` |

C shims for string-returning functions must return the AriaString struct.

#### Float ABI

Aria's `flt32` passes as `double` at the C ABI level. C shims must use `double` params:

```c
// C side
double my_func(double x) {    // NOT float!
    return (float)x * 2.0f;
}
```

#### Pointer Types in Extern

Pointer types in extern blocks may create `{i1, ptr}` optional wrappers that corrupt
struct fields. Workaround: use `int64` for handle/pointer types (ABI-compatible on x86-64).

#### Example

```aria
extern "libc" {
    func:printf = int32(string:fmt);
    func:malloc = int64(int64:size);    // use int64 instead of ptr
}

func:main = int32() {
    drop printf("Hello from C!\n");
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

#### Related

- [modules/use_import.md](../modules/use_import.md) — importing Aria modules
- [types/string.md](../types/string.md) — string ABI details


##### Generics

#### Generic Functions

```aria
func:identity = T(T:value) {
    pass value;
}

func:max = T(T:a, T:b) {
    if (a > b) {
        pass a;
    }
    pass b;
}
```

Generic type parameters are monomorphized at compile time — one specialized version
per concrete type used.

#### Turbofish Syntax

Explicitly specify type arguments with `::<T>`:

```aria
int32:val = identity::<int32>(42);
flt64:pi = identity::<flt64>(3.14);
```

#### Generic Constraints

Constraints limit which types can be used via trait bounds:

```aria
func<T: Addable>:sum = int32(T:a, T:b) {
    pass (a + b);
};
```

See [advanced_features/traits.md](../advanced_features/traits.md) for trait definitions.

#### Related

- [advanced_features/traits.md](../advanced_features/traits.md) — trait constraints
- [advanced_features/rules.md](../advanced_features/rules.md) — refinement types


##### main and failsafe — Mandatory Functions

Every Aria program **must** define both `main` and `failsafe`.

#### main

Program entry point. Must call `exit()` (not `pass`/`fail`):

```aria
func:main = int32() {
    println("Hello, world!");
    exit 0;
}
```

- Signature: `func:main = int32()` — takes no arguments (argc/argv via sys)
- Must call `exit(code)` where code is an `int32`
- `exit 0` = success, nonzero = error
- `pass`/`fail` are **not valid** in main

#### failsafe

Mandatory unrecoverable error handler. Called when `?!` unwrap fails:

```aria
func:failsafe = int32(tbb32:err) {
    println("Fatal error occurred");
    exit 1;
}
```

- Signature: `func:failsafe = int32(tbb32:err)` — receives error code
- Must call `exit(code)` where code > 0
- `pass`/`fail` are **not valid** in failsafe
- Called automatically on emphatic unwrap (`?!`) failure
- Called by `!!! err;` shorthand
- This is Aria's last line of defense — it must always exit

#### Required in Every Program

Both functions must be present. A program without either will not compile.

#### Paren Syntax

`exit` supports both keyword and paren syntax:

```aria
exit 0;      // keyword form
exit(0);     // paren form
```


##### Result System — pass and fail

#### Overview

Inside regular functions (not `main` or `failsafe`), use `pass` and `fail` to return
Results:

- `pass value` — return success Result containing value
- `fail errcode` — return error Result with tbb8 error code

#### Pass

```aria
func:square = int32(int32:x) {
    pass (x * x);
}

func:greet = NIL(string:name) {
    println(`Hello, &{name}!`);
    pass(NIL);           // NIL-returning functions must pass NIL
}
```

`pass` wraps the value in `Result<T>{value=val, error=NIL, is_error=false}`.

#### Fail

```aria
func:divide = flt64(int32:a, int32:b) {
    if (b == 0) {
        fail 1;          // error code 1 — division by zero
    }
    pass (a / b);
}
```

`fail` wraps the error code in `Result<T>{value=NIL, error=code, is_error=true}`.

#### Paren Syntax

Both support optional parentheses (v0.4.6+):

```aria
pass(42);        // same as: pass 42;
fail(1);         // same as: fail 1;
pass (x * x);   // parenthesized expression (always worked)
```

#### return Keyword

`return` is also available and expects a full `Result<T>` instance:

```aria
return Result{ error: err, value: NIL, is_error: true };
```

In practice, `pass`/`fail` are preferred — they're more concise.

#### Related

- [types/result.md](../types/result.md) — Result<T> type
- [operators/result_operators.md](../operators/result_operators.md) — ?, ?!, ?|, drop, raw
- [main_failsafe.md](main_failsafe.md) — exit instead of pass/fail

\newpage

## Control Flow


##### break and continue

#### break

Exit the innermost loop immediately:

```aria
loop(0, 100, 1) {
    if ($ == 42) {
        break;
    }
}
```

#### continue

Skip to the next iteration:

```aria
loop(0, 10, 1) {
    if ($ == 5) {
        continue;    // skip 5
    }
    println($);
}
```

#### Works With

- `for`
- `while`
- `loop` / `till`
- `when/then/end`


##### Error Flow

#### Layered Error Model

Aria has a three-tier error system:

| Tier | Mechanism | Scope | Recovery |
|------|-----------|-------|----------|
| Value-level | TBB ERR sentinel | Arithmetic | Check `== ERR` |
| Function-level | Result<T> pass/fail | Function calls | `?`, `?!`, `?|`, `drop`, `raw` |
| Program-level | failsafe | Fatal errors | Must `exit(code > 0)` |

#### Error Propagation Flow

```
TBB ERR (value) → Result<T> fail (function) → failsafe (program)
```

1. **TBB arithmetic** — overflow, div-by-zero → ERR sentinel, propagates through pipeline
2. **Function calls** — `fail errcode` → caller must handle Result
3. **Unrecoverable** — `?!` failure, `!!! err` → calls failsafe, program exits

#### The `ok` Keyword

Allows passing a variable that may have value `unknown` down the call chain:

```aria
ok maybe_unknown_value;
```

#### The `!!!` Operator — Failsafe Shorthand

The `!!!` operator immediately invokes `failsafe` with an error code. It is syntactic sugar:

```
!!! errcode;    →    failsafe(errcode);
```

##### Usage

```aria
func:validate = int32(int32:input) {
    if (input < 0) {
        !!! 42;     // calls failsafe(42) — program exits
    }
    pass input;
}

func:main = int32() {
    int32:val = raw validate(10);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    // err == 42 if triggered by !!! above
    exit err => int32;
}
```

##### When To Use

| Mechanism | When | Recovery |
|-----------|------|----------|
| `fail errCode` | Recoverable error in a function | Caller handles Result |
| `?!` (emphatic unwrap) | Unwrap Result, failsafe if error | failsafe receives error |
| `!!! errCode` | Unrecoverable error, bail immediately | failsafe receives errCode |

##### Interaction with `defer`

`defer` blocks still execute before `!!!` transfers control to failsafe:

```aria
func:cleanup_example = NIL() {
    defer {
        // This runs BEFORE failsafe is called
    };

    !!! 1;   // defer fires, then failsafe(1)
    pass NIL;
}
```

#### Related

- [types/tbb.md](../types/tbb.md) — TBB sticky error propagation
- [types/result.md](../types/result.md) — Result<T> system
- [functions/main_failsafe.md](../functions/main_failsafe.md) — failsafe function
- [operators/result_operators.md](../operators/result_operators.md) — error handling operators


##### for Loop

#### C-Style For

```aria
for (int32:i = 0; i < 10; i++) {
    println(i);
}
```

Standard three-part syntax: init; condition; update.

#### Notes

- No trailing semicolon after closing brace
- For simple iteration with a counter, prefer `loop()` or `till()`
- `for` is best for complex iteration patterns

#### Related

- [loop_till.md](loop_till.md) — loop/till with automatic `$` iterator
- [while.md](while.md) — condition-only loops


##### if / else

#### Syntax

`if` **requires parentheses** around the condition:

```aria
if (x > 0) {
    println("positive");
} else if (x == 0) {
    println("zero");
} else {
    println("negative");
}
```

#### Rules

- Parentheses around condition are **mandatory**
- No trailing semicolon after the closing brace
- Braces are required (no single-statement if)

#### Common Patterns

```aria
// Guard clause
if (input == NIL) {
    fail 1;
}

// With logical operators
if (age >= 18 && has_id) {
    println("Allowed");
}
```


##### loop and till — Counted Iteration

#### loop(start, limit, step)

Counted loop with automatic iteration variable `$`:

```aria
loop(0, 10, 1) {
    println($);       // prints 0, 1, 2, ..., 9
}

loop(0, 100, 2) {
    println($);       // prints 0, 2, 4, ..., 98
}
```

- `$` is the automatic iteration variable — no manual declaration needed
- `start` is the initial value
- `limit` is the exclusive upper bound
- `step` is the increment per iteration

#### till(limit, step)

Shorthand when starting from 0:

```aria
till(10, 1) {
    println($);       // prints 0, 1, 2, ..., 9
}
```

Equivalent to `loop(0, limit, step)`.

#### Using $ in Expressions

```aria
int32:sum = 0;
loop(1, 101, 1) {
    sum = (sum + $);   // sum of 1 to 100
}
println(sum);          // 5050
```

#### Notes

- No trailing semicolon after closing brace
- `$` is a **reserved symbol** — cannot use for other purposes inside loop body
- For complex iteration, use `for` instead

#### Related

- [for.md](for.md) — C-style for loops
- [while.md](while.md) — condition-only loops


##### pick — Switch/Case

#### Syntax

Aria's equivalent of switch/case:

```aria
pick (value) {
    (1i32) { println("one"); },
    (2i32) { println("two"); },
    (3i32) { println("three"); },
    (*) { println("other"); }
}
```

#### Rules

1. **`(*)` wildcard is required** for types with infinite domains (int32, string, etc.)
2. Arms use `(value) { body }` syntax — value in parentheses, then block
3. Arms are separated by commas
4. No implicit fallthrough (unlike C switch)

#### Fallthrough — `fall`

Explicit fallthrough to a labeled case:

```aria
pick (value) {
    (1i32) { println("one"); fall two; },
    two: (2i32) { println("two or fell from one"); },
    (*) { println("other"); }
}
```

Labels are placed before the match pattern with a colon: `label: (val) { }`.
Use `fall label;` to jump to a named case.

#### With Enums

Enums support exhaustiveness checking — if all variants are covered, `(*)` is optional:

```aria
enum:Color = { RED, GREEN, BLUE };
Color:c = Color.RED;

pick (c) {
    (Color.RED)   { println("Red"); },
    (Color.GREEN) { println("Green"); },
    (Color.BLUE)  { println("Blue"); }
}
```

#### Related

- [if_else.md](if_else.md) — simple branching
- [types/enum.md](../types/enum.md) — enums with exhaustiveness


##### when / then / end — State-Tracked Loop

#### Syntax

```aria
when (condition) {
    // body — executes while condition is true
} then {
    // runs when loop completed normally (condition became false)
} end {
    // runs when condition was initially false, or break was used
}
```

#### Example

```aria
int32:i = 0i32;
int32:result = 0i32;
when (i < 3i32) {
    i = i + 1i32;
} then {
    result = 1i32;     // loop completed normally
} end {
    result = 2i32;     // condition was initially false or break
}
```

#### vs while

`when/then/end` is semantically similar to `while` but also tracks *how* the loop
terminated. `then` runs on normal completion (condition became false). `end` runs if
the condition was initially false or `break` was used. Use `while` for simple loops,
`when` for state-driven loops where termination reason matters.

#### Notes

- The loop body goes in the `when (cond) { }` block
- `then { }` is the normal-completion handler
- `end { }` is the false-or-break handler

#### Related

- [while.md](while.md) — simple condition loops
- [loop_till.md](loop_till.md) — counted iteration


##### while Loop

#### Syntax

```aria
while (condition) {
    // body
}
```

#### Example

```aria
int32:count = 0;
while (count < 10) {
    println(count);
    count++;
}
```

#### Notes

- No trailing semicolon after closing brace
- For state-tracked loops, prefer `when/then/end`
- For counted iteration, prefer `loop()` or `till()`

#### Related

- [when.md](when.md) — when/then/end (state-tracked while)
- [loop_till.md](loop_till.md) — counted iteration

\newpage

## Memory Model


##### Borrow Semantics

#### Overview

Aria has borrow operators for safe reference passing:

| Operator | Meaning |
|----------|---------|
| `$$i` | Immutable borrow — read-only reference |
| `$$m` | Mutable borrow — read-write reference |

#### Usage

```aria
$$i int32:ref = value;        // immutable borrow
$$m int32:mut_ref = value;    // mutable borrow
```

Multiple immutable borrows are allowed simultaneously. Only one mutable borrow at a time.
The compiler enforces the 1-mut XOR N-immut rule.

#### Pin

The `#` operator pins a value, preventing it from being moved:

```aria
#value;                 // pin — cannot be moved after this
```

#### Related

- [overview.md](overview.md) — memory model overview
- [types/pointer.md](../types/pointer.md) — pointer types


##### Garbage Collected Allocation

#### Declaration

GC allocation is **implicit** — variables that are not marked `wild` or `stack` are
garbage collected:

```aria
Point:p1;
p1.x = 10i32;
p1.y = 20i32;
```

The `gc` keyword exists in the lexer but is not used as a prefix in practice.
Absence of `wild` = GC-managed.

#### When to Use

- Data that must outlive the creating function's scope
- Shared references across multiple parts of the program
- Complex object graphs with cycles

#### Notes

`gc` is a contextual keyword — it can be used as a variable name outside allocation contexts.

#### Related

- [overview.md](overview.md) — all allocation modes
- [stack.md](stack.md) — stack allocation (prefer when possible)
- [wild.md](wild.md) — unmanaged memory


##### Handle<T> — Safe Arena Pointers

#### Overview

`Handle<T>` is a generational index for arena-allocated data. It prevents
use-after-free (CWE-416) by tracking generation counters.

#### Structure

```
Handle<T> = { uint64:index, uint32:generation }
// 12 bytes data + 4 padding = 16 bytes, 8-byte aligned
```

#### Usage

```aria
// Create arena and allocate
Handle<Point>:h = arena.alloc(Point{x: 1.0, y: 2.0});

// Access (checks generation — returns ERR if stale)
Point:p = arena.get(h) ? default_point;

// Update
arena.set(h, Point{x: 3.0, y: 4.0});

// Free (increments generation — future gets return ERR)
arena.free(h);

// Grow arena (increments ALL generation counters)
arena.grow();
```

#### Why Not Raw Pointers?

| Problem | Raw Pointer | Handle<T> |
|---------|-------------|-----------|
| Use-after-free | Undefined behavior | ERR (generation mismatch) |
| Dangling reference | Silent corruption | Detected at access time |
| Double free | Undefined behavior | ERR on second free |

#### Related

- [wild.md](wild.md) — unmanaged memory (when Handles aren't appropriate)
- [overview.md](overview.md) — memory model overview


##### Memory Model Overview

#### Three Allocation Modes

Aria provides explicit control over memory allocation:

| Keyword | Mode | Management | Use Case |
|---------|------|------------|----------|
| `stack` | Stack | Automatic (scope-based) | Default, fast, most variables |
| `gc` | Garbage Collected | Automatic (GC runtime) | Shared data, complex lifetimes |
| `wild` | Unmanaged | Manual | FFI, OS-level, performance-critical |

#### Stack Allocation (Default)

```aria
int32:x = 42;           // stack-allocated (default)
stack int32:y = 99;      // explicit stack keyword
```

Stack variables are freed when their scope exits. This is the fastest allocation mode.

**Note:** `stack` and `gc` are **contextual keywords** — they can be used as variable
names outside allocation contexts.

#### GC Allocation

```aria
string:data = "hello";     // no wild/stack prefix = GC-managed
```

GC-allocated values are tracked by the garbage collector and freed when no longer
referenced. This is implicit — absence of `wild` or `stack` means GC mode.

#### Wild (Unmanaged)

```aria
wild int32:raw_val = 42;
wild int8->:buf = alloc(1024);
```

No automatic cleanup. Combine with `defer` for manual resource management.

#### Choosing a Mode

- **Default to stack** — fast, safe, no overhead
- **Use GC** when data must outlive its creating scope or be shared
- **Use wild** only for FFI, OS interaction, or performance-critical paths
- **Never use wild without defer** unless you have a specific reason

#### Related

- [stack.md](stack.md) — stack allocation details
- [gc.md](gc.md) — garbage collector
- [wild.md](wild.md) — unmanaged memory
- [handle.md](handle.md) — Handle<T> safe arena pointers
- [borrow.md](borrow.md) — borrow semantics


##### Stack Allocation

#### Overview

Stack allocation is the **default** in Aria. Variables are allocated on the function's
stack frame and automatically freed when the scope exits.

#### Declaration

```aria
int32:x = 42;           // implicitly stack
stack int32:x = 42;      // explicitly stack (same effect)
```

#### Scope Rules

Stack variables exist only within their declaring scope:

```aria
func:example = NIL() {
    int32:outer = 1;
    if (true) {
        int32:inner = 2;   // only exists in this if-block
    }
    // 'inner' is gone here
    pass NIL;
}
```

#### Performance

Stack allocation is essentially free — just a pointer bump. No garbage collection
overhead, no manual deallocation needed.

#### Related

- [overview.md](overview.md) — all allocation modes
- [gc.md](gc.md) — for data that must outlive scope


##### Wild — Unmanaged Memory

#### Declaration

```aria
wild int32:raw = 42;
wild int8->:buffer = alloc(1024);
wild int64->:ptr = alloc<int64>();
```

Wild memory is **not tracked** by stack cleanup or GC. You are responsible for cleanup.

#### Defer — Guaranteed Cleanup

Use `defer` to ensure cleanup runs when scope exits:

```aria
wild int8->:buffer = alloc(1024);
defer {
    free(buffer);
}

// ... use buffer ...
// free(buffer) runs automatically at scope exit
```

`defer` executes in LIFO (last-in, first-out) order for multiple defers.

#### Executable Memory — wildx

`wildx` allocates memory with execute permission for JIT compilation:

```aria
wildx uint8->:code = wildx_alloc(4096);   // 4KB executable page
defer {
    wildx_free(code);
}

// Write machine code into 'code' buffer
// Execute via function pointer cast
```

**Note:** `wildx` is specified but has no passing tests yet.

#### When to Use Wild

- FFI buffers passed to C functions
- OS-level operations (mmap, syscalls)
- JIT compilation (`wildx`)
- Performance-critical paths where GC pauses are unacceptable

#### Related

- [overview.md](overview.md) — all allocation modes
- [handle.md](handle.md) — safe alternative to raw pointers
- [types/pointer.md](../types/pointer.md) — pointer syntax

\newpage

## Modules


##### mod — Defining Modules

#### Module Declaration

```aria
mod math {
    pub func:add = int32(int32:a, int32:b) {
        pass (a + b);
    };

    func:helper = int32(int32:x) {   // private to module
        pass (x * 2);
    };
}
```

External module declaration (separate file):

```aria
mod crypto;
```

#### Visibility

- `pub` — public, visible to importers
- Default — private, visible only within the module/file

#### Related

- [use_import.md](use_import.md) — importing modules


##### Packages

#### Package Structure

Aria packages follow a standard layout managed by `aria-make`:

```
my-package/
├── src/
│   ├── main.aria
│   └── lib.aria
├── test/
│   └── test_main.aria
├── aria-make.toml
└── README.md
```

#### aria-make.toml

```toml
[package]
name = "my-package"
version = "0.1.0"

[dependencies]
aria-string = "0.1.0"
```

#### Building

```bash
aria-make build        # compile
aria-make test         # run tests
aria-make run          # build and run
```

#### Package Registry

Packages are hosted at `aria-packages` and `aria-packages-apt`:
- Source packages: `REPOS/aria-packages/packages/`
- APT packages: `REPOS/aria-packages-apt/`

#### Package Census (v0.11.3)

| Category | Count | % |
|----------|-------|---|
| Pure Aria | 17 | 24.6% |
| aria-libc backed | 9 | 13.0% |
| Direct extern FFI | 43 | 62.3% |
| **Total (with TOML)** | **69** | |
| Legacy/scaffold (no TOML) | 27 | |
| **Grand total** | **96** | |

##### New in v0.11.3

| Package | Category | Description |
|---------|----------|-------------|
| `aria-bitset` | Pure Aria (aria-libc mem) | Fixed-size bit sets with union, intersect, complement |
| `aria-result` | Pure Aria (aria-libc mem) | Extended Result combinators — unwrap, map_or, and/or |
| `aria-deque` | Pure Aria (aria-libc mem) | Double-ended queue with O(1) push/pop at both ends |

#### Related

- [use_import.md](use_import.md) — importing from packages
- [mod.md](mod.md) — module definitions


##### Standard Library Overview

The standard library contains **59 modules** in `REPOS/aria/stdlib/`.

#### Import Pattern

```aria
use "stdlib_file.aria".*;    // bare filename — compiler searches stdlib/
```

#### Available Modules

The standard library lives in `REPOS/aria/stdlib/` and provides:

##### String Utilities
- `string_length(s)` — character count (UTF-8 aware)
- `string_concat(a, b)` — concatenation
- `string_contains(s, substr)` — substring search

##### I/O
- `print(s)` — output without newline
- `println(s)` — output with newline
- `readFile(path)` — read file contents
- `writeFile(path, data)` — write file contents

##### Math
- Standard math functions via extern libm

##### Threading
- Thread pool, channels, atomics, mutexes, condvars, rwlocks, barriers, actors (via stdlib wrappers on aria-libc)

##### Concurrency Modules (v0.11.0)
- `thread.aria`, `mutex.aria`, `condvar.aria`, `rwlock.aria` — via aria_libc_process
- `channel.aria`, `actor.aria`, `thread_pool.aria` — built on above
- `shm.aria` — POSIX shared memory via aria_libc_posix

#### Related

- [modules/use_import.md](../modules/use_import.md) — import syntax
- [io_system/print.md](../io_system/print.md) — print/println details


##### use — Importing Modules

#### Syntax

```aria
use "path/to/module.aria".*;           // wildcard import (all pub symbols)
use "stdlib_file.aria".*;              // stdlib import (bare filename)
```

#### Path Resolution

- **Relative paths**: resolved relative to the importing source file
- **Bare filenames**: compiler searches the stdlib directory
- Wildcard `.*` imports all `pub` declarations

#### Example

```aria
// file: math_utils.aria
pub func:square = int32(int32:x) {
    pass (x * x);
}

// file: main.aria
use "math_utils.aria".*;

func:main = int32() {
    int32:val = raw square(5);
    println(val);    // 25
    exit 0;
}

func:failsafe = int32(tbb32:err) { exit 1; }
```

#### Notes

- Only `pub func:` declarations are importable
- `Rules<T>` are file-scoped — not importable
- Negative constants imported via `use` may be zeroed at runtime (known codegen issue)
  — compute inline as `0i64 - value` instead

#### Related

- [mod.md](mod.md) — defining modules
- [packages.md](packages.md) — package structure

\newpage

## Collections


##### Hash Tables — ahash

#### Overview

Per-scope hash tables mapping string keys to typed values. Heterogeneous (different
value types per key). Auto-cleanup on scope exit. Optional byte-budget capacity.

#### Builtins

| Builtin | Signature | Description |
|---------|-----------|-------------|
| `ahash(cap)` | `(int64) → int64` | Create hash table with byte budget. 0 = unbounded. Returns handle. |
| `ahset(h, key, val)` | `(int64, str, T) → int32` | Set key to value. Returns 0 on success, -1 on overflow. |
| `ahget(h, key)` | `(int64, str) → T` | Get value at key. NIL if not found. Type mismatch → failsafe. |
| `ahcount(h)` | `(int64) → int64` | Number of keys in table. |
| `ahsize(h)` | `(int64) → int64` | Current size in bytes of all stored entries. |
| `ahfits(h, val)` | `(int64, T) → int64` | 1 if val fits in remaining capacity, 0 otherwise. Always 1 if unbounded. |
| `ahtype(h, key)` | `(int64, str) → int32` | Type tag of value at key, -1 if key not found. |

#### Example

```aria
func:main = int32() {
    int64:ht = ahash(0);                 // unbounded hash table

    int32:rc1 = ahset(ht, "name", "Alice");
    int32:rc2 = ahset(ht, "age", 30);
    int32:rc3 = ahset(ht, "score", 98.6);

    string:name = ahget(ht, "name");
    int32:age = ahget(ht, "age");
    flt64:score = ahget(ht, "score");

    int64:count = ahcount(ht);           // 3

    println(name);
    println(age);
    exit 0;
}

func:failsafe = int32(tbb32:err) { exit 1; }
```

#### Fast-Path Mode

When the Z3 solver can prove all values in a hash table are the same type at compile
time, the compiler emits optimized **fast-path** code that skips runtime type checks.
This is automatic — no user action required.

#### Capacity

- `ahash(0)` — unbounded (grows as needed)
- `ahash(1024)` — 1KB byte budget; `ahset` returns -1 when full
- `ahfits(h, val)` — check before inserting into bounded tables

#### Type Safety

The receiving variable's type determines the expected type on `ahget`:

```aria
int32:val = ahget(ht, "age");     // expects int32 at "age"
string:s = ahget(ht, "age");      // type mismatch → failsafe!
```

Check with `ahtype(h, key)` before getting if unsure.

#### Auto-Cleanup

Hash tables are automatically freed when the scope exits.

#### Related

- [astack.md](astack.md) — user stack (LIFO)


##### User Stack — astack

#### Overview

Per-scope implicit LIFO (last-in, first-out) scratch pad. Stores heterogeneous typed
values. Auto-cleanup on scope exit. Fatal error model (exit(1) on error, not Result<T>).

#### Builtins

| Builtin | Signature | Description |
|---------|-----------|-------------|
| `astack(cap)` | `(int64?) → int64` | Initialize stack with byte budget (default 256). Returns handle. |
| `apush(h, val)` | `(int64, T) → void` | Push typed value. Fatal on overflow. |
| `apop(h)` | `(int64) → T` | Pop top value. Type inferred from assignment. Fatal on mismatch/underflow. |
| `apeek(h)` | `(int64) → T` | Non-destructive read of top value. Same typed semantics as apop. |
| `acap(h)` | `(int64) → int64` | Total capacity in bytes. |
| `asize(h)` | `(int64) → int64` | Bytes currently used. |
| `afits(h, val)` | `(int64, T) → bool` | True if val fits without overflow. |
| `atype(h)` | `(int64) → int32` | Type tag of top element. Useful to avoid mismatch. |

#### Example

```aria
func:main = int32() {
    int64:stk = astack 512;            // 512-byte stack (keyword form)
    // int64:stk = astack(512);        // paren form also works

    apush stk, 42;                      // push int32
    apush stk, "hello";                 // push string
    apush(stk, 3.14);                   // paren form

    string:s = apop stk;                // pop string: "hello" (LIFO)
    // string:s = apop(stk);            // paren form

    int32:n = apop(stk);                // pop int32: 42
    println(n);

    exit 0;
}

func:failsafe = int32(tbb32:err) { exit 1; }
```

#### Syntax Forms (v0.4.6+)

All astack builtins support both keyword and paren syntax:

```aria
// Statement-level (apush, astack)
apush stk, value;       // keyword
apush(stk, value);      // paren
astack 256;             // keyword
astack(256);            // paren

// Expression-level (apop, apeek, acap, asize, atype)
int32:v = apop stk;     // keyword
int32:v = apop(stk);    // paren

// Expression-level with argument (afits)
bool:ok = afits stk, value;   // keyword
bool:ok = afits(stk, value);  // paren
```

#### Error Behavior

Errors are **fatal** (not Result<T>):
- Stack overflow → `exit(1)`
- Stack underflow → `exit(1)`
- Type mismatch on pop/peek → `exit(1)`

Check before popping:
```aria
if (asize(stk) > 0) {
    int32:val = apop(stk);
}
```

#### Auto-Cleanup

The stack is automatically cleaned up when the scope exits (function return). No manual
deallocation needed.

#### Related

- [ahash.md](ahash.md) — hash tables

\newpage

## Io System


##### File I/O

#### Reading Files

```aria
string:content = raw readFile("data.txt");
```

#### Writing Files

```aria
drop writeFile("output.txt", "Hello, file!");
```

#### File Operations

| Function | Description |
|----------|-------------|
| `readFile(path)` | Read entire file as string |
| `writeFile(path, data)` | Write string to file |
| `fileExists(path)` | Check if file exists |
| `fileSize(path)` | Get file size in bytes |
| `deleteFile(path)` | Delete a file |

All file operations return `Result<T>`.

#### Streams

| Stream | FD | Purpose |
|--------|----|---------|
| `stdin` | 0 | Standard input |
| `stdout` | 1 | Standard output |
| `stderr` | 2 | Standard error |
| `stddbg` | 3 | Debug stream (Hexstream) |
| `stddati` | 4 | Data input stream (Hexstream) |
| `stddato` | 5 | Data output stream (Hexstream) |

#### Related

- [print.md](print.md) — print/println
- [stdin.md](stdin.md) — reading from stdin
- [sys.md](sys.md) — direct syscalls


##### Print & Output

#### print / println

```aria
print("no newline");
println("with newline");
println(42);                        // prints integer
println(`Value: &{expression}`);    // interpolation
```

- `print(s)` — output without newline
- `println(s)` — output with newline

#### Streams

| Stream | Purpose |
|--------|---------|
| `stdout` | Standard text output |
| `stderr` | Error output |
| `stddbg` | Debug output |

#### Related

- [types/string.md](../types/string.md) — string types and interpolation
- [file_io.md](file_io.md) — file operations


##### Standard Input

#### Reading from stdin

```aria
string:line = raw stdin_read_line();    // read one line
string:all = raw stdin_read_all();      // read until EOF
```

Both return `Result<string>`.

#### Related

- [print.md](print.md) — output
- [file_io.md](file_io.md) — file operations


##### sys() — Direct Syscalls

#### Three Tiers

Aria provides direct syscall access at three safety levels:

| Function | Tier | Syscalls Available | Return Type | TOS Layer |
|----------|------|-------------------|-------------|-----------|
| `sys(CONST, args...)` | Safe | ~55 curated whitelist | `Result<int64>` | Layer 1 |
| `sys!!(CONST, args...)` | Full | All syscalls | `Result<int64>` | Layer 2 |
| `sys!!!(expr, args...)` | Raw | Any int64 expression | `int64` (bare) | Layer 3 |

#### Safe Tier — sys()

Curated whitelist of ~55 safe syscalls. Returns `Result<int64>`:

```aria
int64:fd = sys(SYS_OPEN, path, flags, mode) ? -1i64;
```

#### Full Tier — sys!!()

All syscalls available, but still returns Result:

```aria
int64:result = sys!!(SYS_MMAP, addr, length, prot, flags, fd, offset) ? -1i64;
```

#### Raw Tier — sys!!!()

Accepts any int64 expression as syscall number. Returns bare int64 (no Result wrapper):

```aria
int64:result = sys!!!(231i64, 0i64);   // SYS_EXIT_GROUP
```

#### Known Issue

Bare filename string literals in `sys()` calls may fail. Store in a variable first:

```aria
// WRONG: sys(SYS_OPEN, "file.txt", 0i64, 0i64);
// RIGHT:
string:path = "file.txt";
sys(SYS_OPEN, path, 0i64, 0i64);
```

#### Related

- [file_io.md](file_io.md) — higher-level file operations
- [control_flow/error_flow.md](../control_flow/error_flow.md) — error handling tiers

\newpage

## Standard Library


##### Standard Library — Math

#### Functions

Standard math functions available via extern libm:

| Function | Description |
|----------|-------------|
| `sin(x)` | Sine |
| `cos(x)` | Cosine |
| `sqrt(x)` | Square root |
| `abs(x)` | Absolute value |
| `pow(x, y)` | Power |
| `log(x)` | Natural logarithm |
| `floor(x)` | Floor |
| `ceil(x)` | Ceiling |

#### Usage

```aria
extern "libm" {
    func:sin = double(double:x);
    func:cos = double(double:x);
    func:sqrt = double(double:x);
}

flt64:result = sin(3.14159);
```

#### Related

- [types/flt.md](../types/flt.md) — floating point types
- [types/fix256.md](../types/fix256.md) — deterministic math alternative


##### Standard Library Overview

#### What's in stdlib/

The standard library provides common functionality as importable Aria modules.
Located at `REPOS/aria/stdlib/`.

#### Import Pattern

```aria
use "module_name.aria".*;    // bare filename — compiler searches stdlib/
```

#### Available Categories

| Category | Modules | See |
|----------|---------|-----|
| String | string_utils | [string_utils.md](string_utils.md) |
| Math | math functions via libm | [math.md](math.md) |
| I/O | print, readFile, writeFile | [io_system/](../io_system/) |
| Threading | thread pool, channels | [advanced_features/concurrency.md](../advanced_features/concurrency.md) |

#### Built-in (No Import Needed)

These are available without `use`:
- `print()`, `println()`
- `astack`, `apush`, `apop`, `apeek`, `acap`, `asize`, `afits`, `atype`
- `ahash`, `ahset`, `ahget`, `ahcount`, `ahsize`, `ahfits`, `ahtype`
- `sys()`, `sys!!()`, `sys!!!()`

#### Related

- [modules/use_import.md](../modules/use_import.md) — import syntax
- [modules/packages.md](../modules/packages.md) — third-party packages


##### Standard Library — String Utilities

#### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `string_length(s)` | `(string) → int64` | Character count (UTF-8 aware) |
| `string_concat(a, b)` | `(string, string) → string` | Concatenate two strings |
| `string_contains(s, sub)` | `(string, string) → bool` | Check if substring exists |

#### Usage

```aria
use "string_utils.aria".*;

string:name = "Hello, World!";
int64:len = raw string_length(name);      // 13
bool:has = raw string_contains(name, "World");  // true
```

#### Related

- [types/string.md](../types/string.md) — string type details
- [operators/string_ops.md](../operators/string_ops.md) — string operators

\newpage

## Advanced Features


##### Concurrency

#### Overview

Aria's concurrency model provides four layers of abstraction:

| Layer | API | Use Case |
|-------|-----|----------|
| Threads | `aria_shim_thread_*` | Direct POSIX thread control |
| Thread Pool | `aria_shim_pool_*` | Task-parallel work distribution |
| Channels | `aria_shim_channel_*` | Typed message-passing between threads |
| Actors | `aria_shim_actor_*` | Self-contained message-processing entities |

All concurrency primitives are accessed via `extern` declarations to the runtime shim layer.

---

#### 1. Threads

##### Spawning a Thread

```aria
extern func:aria_shim_thread_spawn = int64(int64:func_ptr, int64:arg);
extern func:aria_shim_thread_join = int32(int64:handle);
extern func:aria_shim_thread_detach = int32(int64:handle);
extern func:aria_shim_thread_yield = NIL();
extern func:aria_shim_thread_sleep_ms = NIL(int64:ms);
extern func:aria_shim_thread_hardware_concurrency = int32();
extern func:aria_shim_thread_current_id = int64();

func:worker = int64(int64:arg) {
    // Worker body — arg is user-provided data
    pass arg * 2i64;
}

func:main = int32() {
    int64:handle = aria_shim_thread_spawn(@worker, 42i64);
    drop aria_shim_thread_join(handle);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

---

#### 2. Thread Pool

```aria
extern func:aria_shim_pool_create = int64(int32:num_workers);
extern func:aria_shim_pool_submit = int32(
    int64:handle, int64:func_ptr, int64:arg);
extern func:aria_shim_pool_shutdown = int32(int64:handle);
extern func:aria_shim_pool_wait_idle = int32(int64:handle);
extern func:aria_shim_pool_active_tasks = int64(int64:handle);
extern func:aria_shim_pool_pending_tasks = int64(int64:handle);

func:task = int64(int64:arg) {
    pass arg + 1i64;
}

func:main = int32() {
    int32:cores = aria_shim_thread_hardware_concurrency();
    int64:pool = aria_shim_pool_create(cores);

    // Submit work
    loop(0i64, 100i64, 1i64) {
        drop aria_shim_pool_submit(pool, @task, $);
    }

    // Wait for all tasks to complete
    drop aria_shim_pool_wait_idle(pool);
    drop aria_shim_pool_shutdown(pool);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

---

#### 3. Channels

Typed message-passing between threads. Three channel modes:

| Mode | Creation | Behavior |
|------|----------|----------|
| Buffered | `channel_create(capacity)` | Send blocks when full, recv blocks when empty |
| Unbuffered | `channel_create_unbuffered()` | Send blocks until receiver is ready (rendezvous) |
| Oneshot | `channel_create_oneshot()` | Single send, single recv, then auto-closes |

```aria
extern func:aria_shim_channel_create = int64(int32:capacity);
extern func:aria_shim_channel_send = int32(int64:handle, int64:value);
extern func:aria_shim_channel_recv = int64(int64:handle);
extern func:aria_shim_channel_try_send = int32(int64:handle, int64:value);
extern func:aria_shim_channel_try_recv = int64(int64:handle);
extern func:aria_shim_channel_close = int32(int64:handle);
extern func:aria_shim_channel_destroy = int32(int64:handle);
extern func:aria_shim_channel_count = int32(int64:handle);
extern func:aria_shim_channel_is_closed = int32(int64:handle);

func:producer = int64(int64:ch) {
    loop(1i64, 10i64, 1i64) {
        drop aria_shim_channel_send(ch, $);
    }
    drop aria_shim_channel_close(ch);
    pass 0i64;
}

func:consumer = int64(int64:ch) {
    int64:sum = 0i64;
    while (aria_shim_channel_is_closed(ch) == 0i32) {
        int64:val = aria_shim_channel_recv(ch);
        sum = sum + val;
    }
    pass sum;
}

func:main = int32() {
    int64:ch = aria_shim_channel_create(16i32);
    int64:prod = aria_shim_thread_spawn(@producer, ch);
    int64:cons = aria_shim_thread_spawn(@consumer, ch);

    drop aria_shim_thread_join(prod);
    drop aria_shim_thread_join(cons);
    drop aria_shim_channel_destroy(ch);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### Channel Select

Wait on multiple channels simultaneously:

```aria
extern func:aria_shim_channel_select2 = int32(
    int64:ch0, int64:ch1, int64:timeout_ms);

// Returns index of ready channel (0 or 1), or -1 on timeout
int32:ready = aria_shim_channel_select2(ch_a, ch_b, 5000i64);
```

---

#### 4. Actors

Self-contained entities that process messages from a mailbox (built on channels + threads):

```aria
extern func:aria_shim_actor_spawn = int64(int64:behavior_func);
extern func:aria_shim_actor_send = int32(int64:handle, int64:message);
extern func:aria_shim_actor_stop = int32(int64:handle);
extern func:aria_shim_actor_destroy = int32(int64:handle);
extern func:aria_shim_actor_is_alive = int32(int64:handle);
extern func:aria_shim_actor_pending = int32(int64:handle);
```

---

#### 5. Synchronization Primitives

##### Mutexes

```aria
extern func:aria_shim_mutex_create = int64(int32:type);
extern func:aria_shim_mutex_lock = int32(int64:handle);
extern func:aria_shim_mutex_trylock = int32(int64:handle);
extern func:aria_shim_mutex_unlock = int32(int64:handle);
extern func:aria_shim_mutex_destroy = int32(int64:handle);
```

##### Condition Variables

```aria
extern func:aria_shim_condvar_create = int64();
extern func:aria_shim_condvar_wait = int32(int64:cv, int64:mutex);
extern func:aria_shim_condvar_timedwait = int32(
    int64:cv, int64:mutex, int64:timeout_ns);
extern func:aria_shim_condvar_signal = int32(int64:handle);
extern func:aria_shim_condvar_broadcast = int32(int64:handle);
extern func:aria_shim_condvar_destroy = int32(int64:handle);
```

##### Read-Write Locks

```aria
extern func:aria_shim_rwlock_create = int64();
extern func:aria_shim_rwlock_rdlock = int32(int64:handle);
extern func:aria_shim_rwlock_wrlock = int32(int64:handle);
extern func:aria_shim_rwlock_unlock = int32(int64:handle);
extern func:aria_shim_rwlock_destroy = int32(int64:handle);
```

---

#### 6. Atomics

Low-level atomic operations with explicit memory ordering:

```aria
extern func:aria_shim_atomic_int64_create = int64(int64:initial);
extern func:aria_shim_atomic_int64_load = int64(int64:handle, int32:order);
extern func:aria_shim_atomic_int64_store = int32(
    int64:handle, int64:value, int32:order);
extern func:aria_shim_atomic_int64_fetch_add = int64(
    int64:handle, int64:value, int32:order);
extern func:aria_shim_atomic_int64_exchange = int64(
    int64:handle, int64:value, int32:order);
extern func:aria_shim_atomic_int64_destroy = int32(int64:handle);
```

Memory ordering constants (pass as `int32:order`):

| Keyword | Value | Guarantee |
|---------|-------|-----------|
| `relaxed` | 0 | No synchronization |
| `acquire` | 1 | Reads after this see prior writes |
| `release` | 2 | Writes before this visible to acquirers |
| `acq_rel` | 3 | Combined acquire + release |
| `seq_cst` | 4 | Sequentially consistent (strongest) |

---

#### Async/Await

The `async` keyword declares functions that can suspend and resume:

```aria
async func:fetch_data = string(string:url) {
    // Async operations
    pass data;
}
```

Use `await` to suspend until the async operation completes:

```aria
string:result = await fetch_data("https://example.com");
```

See [functions/async.md](../functions/async.md) for full async documentation.

---

#### Related

- [functions/async.md](../functions/async.md) — async function syntax
- [verification.md](verification.md) — `--verify-concurrency` for data race detection
- [safety_layers.md](safety_layers.md) — safety levels for concurrent code


##### Advanced FFI Patterns

#### Large Struct Passing (LBIM)

Types ≥128 bits are passed via `sret`/`byval` pointers, not registers:

```aria
// int128, int256, etc. are passed by pointer at ABI level
extern func:big_math = int128(int128:a, int128:b);
// C side receives: i128* sret, i128* byval, i128* byval
```

#### String Return from Extern

C functions that return strings must return `AriaString`:

```c
// C side
typedef struct { char* data; int64_t length; } AriaString;

AriaString my_string_func(const char* input) {
    AriaString result;
    result.data = strdup(input);
    result.length = strlen(input);
    return result;
}
```

#### Handle Pattern

Use `int64` instead of pointer types in extern blocks to avoid `{i1, ptr}` wrapper issues:

```aria
extern func:create_context = int64();        // returns handle as int64
extern func:destroy_context = NIL(int64:h);  // takes handle as int64
```

#### Store Before Pass

When passing extern function results directly to `pass`, store in a variable first:

```aria
// WRONG: pass(extern_func());
// RIGHT:
int32:val = extern_func();
pass val;
```

#### Related

- [functions/extern.md](../functions/extern.md) — basic extern usage
- [types/string.md](../types/string.md) — string ABI
- [types/int.md](../types/int.md) — LBIM types


##### JIT Compilation in Aria

#### Overview

Aria includes a built-in JIT assembler for runtime code generation. The JIT subsystem builds on two foundations:

- **WildX** (`wildx_alloc.cpp`) — W⊕X memory management with ASLR, guard pages, code signing, and quota enforcement
- **Assembler** (`assembler.cpp`) — x86-64 instruction encoder with label backpatching, register allocation, peephole optimization, and instruction selection

The JIT is accessible from Aria code through the `jit` stdlib package, which provides FFI bindings to the C++ assembler API.

#### Architecture

```
┌──────────────────────────────────────────────┐
│               Aria Source Code               │
│         use jit;  use wildx;                 │
├──────────────────────────────────────────────┤
│           jit.aria FFI Bindings              │
│    81 bindings + 17 helpers + constants      │
├──────────────────────────────────────────────┤
│           Assembler Pipeline                 │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ IR Queue │→ │ Peephole │→ │ Liveness  │  │
│  │ (lazy)   │  │ Optimizer│  │ Analysis  │  │
│  └──────────┘  └──────────┘  └───────────┘  │
│        ↓              ↓             ↓        │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ Linear   │→ │ Insn     │→ │ Machine   │  │
│  │ Scan RA  │  │Selection │  │ Code Emit │  │
│  └──────────┘  └──────────┘  └───────────┘  │
├──────────────────────────────────────────────┤
│        WildX — W⊕X Memory Manager            │
│  ASLR | Guard Pages | Code Signing | Quota   │
├──────────────────────────────────────────────┤
│              x86-64 Hardware                 │
└──────────────────────────────────────────────┘
```

#### Instruction Set (v0.7.2+)

The assembler supports 45+ x86-64 instructions across multiple categories:

##### Integer
- **Data movement:** `MOV r64, imm64` / `MOV r64, r64`
- **Arithmetic:** `ADD`, `SUB`, `IMUL` (r64,r64 and r64,imm32)
- **Bitwise:** `XOR`, `AND`, `OR`, `NOT`, `NEG`
- **Shifts:** `SHL`, `SHR`, `SAR` with imm8
- **Compare:** `CMP r64, r64` / `CMP r64, imm32`
- **Stack:** `PUSH`, `POP`
- **Flow:** `JMP`, `JE/JNE/JL/JLE/JG/JGE/JB/JBE/JA/JAE`, `RET`
- **Call:** `CALL r64`, `CALL label`, `CALL abs`

##### Floating-Point (SSE2)
- `MOVSD` (reg-reg, load, store), `ADDSD`, `SUBSD`, `MULSD`, `DIVSD`, `UCOMISD`
- XMM0–XMM15 registers

##### SIMD (SSE)
- `MOVAPS` (reg-reg, aligned load/store), `ADDPS`, `MULPS`
- Packed float32 (4x f32) operations

##### Memory
- `MOV r64, [base+offset]` (load), `MOV [base+offset], r64` (store)
- `LEA r64, [base+offset]` (address computation)
- `store_local` / `load_local` (RBP-relative stack frame)

#### Register Allocator (v0.7.3+)

The JIT includes a linear scan register allocator for automatic register assignment:

```aria
use jit;

fn jit_example() {
    let a = jit.asm_create();
    let v0 = jit.asm_vreg_new_gpr(a);  // virtual register
    let v1 = jit.asm_vreg_new_gpr(a);

    jit.asm_mov_r64_imm64(a, v0, 10);
    jit.asm_mov_r64_imm64(a, v1, 32);
    jit.asm_add_r64_r64(a, v0, v1);
    jit.asm_mov_r64_r64(a, jit.REG_RAX, v0);
    jit.asm_ret(a);

    let guard = jit.asm_finalize(a);
    let result = jit.asm_execute(guard);
    // result == 42
}
```

**Features:**
- 12 allocatable GPRs (RAX, RCX, RDX, RSI, RDI, R8, R9, RBX, R12-R15)
- 14 allocatable XMMs (XMM0-XMM13)
- Automatic spill/reload when registers are exhausted
- Auto prologue/epilogue when callee-saved registers are needed
- Mixed physical + virtual register support

#### Peephole Optimizer (v0.7.4)

The JIT runs a peephole optimization pass on the IR before register allocation:

| Pattern | Optimization | Bytes Saved |
|---|---|---|
| `MOV r, 0` | `XOR r, r` | 6–7 |
| `MOV r, r` | eliminated | 3–4 |
| `ADD r, 0` / `SUB r, 0` | eliminated | 7 |
| `SHL r, 0` / `SHR r, 0` | eliminated | 4 |
| `MOV r, X; MOV r, Y` | dead store eliminated | 10 |
| `MOV r, 2^n; IMUL d, r` | `SHL d, n` | ~7 |
| `XOR r, r; ADD r, s` | `MOV r, s` | 3–4 |

Statistics available via `aria_asm_peephole_stats()`.

#### Instruction Selection (v0.7.4)

During code emission, the allocator selects optimal machine encodings:

| IR Instruction | Selected Encoding | Bytes Saved |
|---|---|---|
| `MOV_IMM64` (value ≤ 0xFFFFFFFF) | `MOV r32, imm32` | 4–5 |
| `CMP r, 0` | `TEST r, r` | 4 |
| `ADD r, 1` | `INC r` | 4 |
| `SUB r, 1` | `DEC r` | 4 |
| `ADD/SUB r, imm8` | imm8 form | 3 |

Statistics available via `aria_asm_insn_sel_stats()`.

#### Profiling Integration (v0.7.4)

JIT code can be registered with Linux `perf` for profiling:

```aria
jit.asm_perf_map_register(code_ptr, code_size, "my_jit_function");
// Now visible in: perf record -p <pid> && perf report
```

This writes to `/tmp/perf-<pid>.map` in the format expected by `perf`.

#### WildX Security (v0.7.1)

All JIT code runs through WildX's security pipeline:
- **ASLR:** Random mmap hints for JIT pages
- **Guard pages:** `PROT_NONE` sentinels around executable regions
- **Code signing:** FNV-1a hash verified before every execution
- **W⊕X:** Strict WRITABLE → EXECUTABLE state machine (never both)
- **Quota:** Default 64MB, configurable via `aria_wildx_set_quota()`
- **Audit logging:** `--wildx-audit` flag for ALLOC/SEAL/EXEC/FREE events

#### Multi-Architecture (v0.7.4)

Architecture detection and abstraction:

```aria
let arch = jit.asm_get_arch();       // ASM_ARCH_X86_64 or ASM_ARCH_AARCH64
let ok = jit.asm_arch_supported(arch); // true on x86-64
```

AArch64 backend is stubbed for future implementation. The architecture abstraction layer supports querying the current target and checking support before code generation.

#### Safety

JIT compilation is a **Layer 3 (raw)** operation — it bypasses all safety guarantees. Executable memory is:
- Not bounds-checked
- Not type-checked
- A security risk if used with untrusted input

Always use WildX guards and code signing for JIT code.

#### Related

- [memory_model/wild.md](../memory_model/wild.md) — unmanaged memory
- [types/pointer.md](../types/pointer.md) — pointer operations
- [safety_layers.md](safety_layers.md) — safety layer definitions


##### Macros & Compile-Time Evaluation

#### Overview

Aria provides three mechanisms for compile-time code generation and evaluation:

1. **AST macros** — Template-based code generation invoked with `name!(args)`
2. **Compile-time evaluation** — `comptime(expr)` for constant folding at compile time
3. **Derive macros** — Auto-generate trait implementations (see [Derive Macros](#derive-macros))

---

#### AST Macros

##### Declaration

Macros use `macro:name = (params) { body; };` syntax, consistent with `func:name`:

```aria
macro:double_it = (x) {
    x + x;
};
```

##### Invocation

Invoke a macro with `name!(args)`:

```aria
func:main = int32() {
    int32:val = double_it!(5i32);
    // val == 10 (expanded to: 5i32 + 5i32)
    exit val;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### Multiple Parameters

```aria
macro:add3 = (a, b, c) {
    a + b + c;
};

func:main = int32() {
    int32:result = add3!(1i32, 2i32, 3i32);
    // result == 6
    exit result;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### How It Works

1. The parser sees `macro:name = (params) { body; };` and stores the AST template
2. When `name!(args)` is encountered, arguments are substituted into the template
3. Expansion happens at parse time — the macro body becomes inline code
4. Type checking runs on the expanded code, not the macro definition

##### Key Differences from C Macros

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

#### Compile-Time Evaluation

##### comptime Expression

Evaluate an expression at compile time:

```aria
func:main = int32() {
    int32:size = comptime(4 * 1024);
    // size == 4096, computed at compile time
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### comptime Function

Mark an entire function for compile-time evaluation:

```aria
comptime func:factorial = int64(int64:n) {
    if (n <= 1i64) {
        pass 1i64;
    }
    pass n * raw factorial(n - 1i64);
}

func:main = int32() {
    int64:val = comptime(raw factorial(10i64));
    // val == 3628800, computed entirely at compile time
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### pub comptime Function

Export a compile-time function for use in other modules:

```aria
pub comptime func:align_up = int64(int64:val, int64:align) {
    pass ((val + align - 1i64) / align) * align;
}
```

##### comptime Block

Execute a block of statements at compile time:

```aria
func:main = int32() {
    comptime {
        // All statements here execute at compile time
        int32:x = 10;
        int32:y = x * 2;
    };

    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

---

#### Derive Macros

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
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### Available Derive Traits

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

#### Attribute Macros

Attributes modify declarations at compile time:

```aria
#[inline]
func:hot_path = int32(int32:x) {
    pass x * 2;
}

#[noinline]
func:cold_path = int32(int32:x) {
    pass x / 2;
}

#[align(64)]
struct:CacheLine = {
    int64:data;
};
```

##### Available Attributes

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

#### See Also

- [MACRO_AUTHORING_GUIDE.md](../../reference/MACRO_AUTHORING_GUIDE.md) — Full derive and attribute reference
- [Traits](traits.md) — Trait system documentation
- [Generics](../functions/generics.md) — Generic functions and turbofish


##### Rules & Limit — Refinement Types

#### Rules<T>

Rules define compile-time and runtime constraints on types:

```aria
Rules<int32>:r_positive = {
    $ > 0
};

Rules<int32>:r_small_positive = {
    limit<r_positive>,
    $ < 100
};
```

Multi-type rules (value validated against multiple widths):

```aria
Rules<int32,int64>:r_positive_wide = {
    $ > 0
};
```

#### limit<Rules>

Apply Rules as a type annotation:

```aria
func:process = NIL(limit<r_positive> int32:count) {
    // 'count' is guaranteed > 0
    pass NIL;
}
```

#### Member Access in Rules — $.field

```aria
Rules<Person>:r_adult = {
    $.age >= 18
};
```

#### Array Rules

```aria
Rules<int32[]>:arr_first_positive = { $[0] > 0 };
Rules<int32[]>:arr_min_length = { $.length >= 4 };
```

#### Notes

- `$` is the value being checked
- `$.field` accesses struct members
- `$[n]` accesses array elements
- Rules are file-scoped — not importable via `use`

#### Related

- [traits.md](traits.md) — behavioral constraints
- [functions/design_by_contract.md](../functions/design_by_contract.md) — requires/ensures


##### Safety Layers — TOS (Terms of Safety)

#### Overview

Aria's safety model uses explicit layers. Each layer grants more power and less safety:

| Layer | Name | Access | Safety |
|-------|------|--------|--------|
| 0 | Safe | Default Aria code | Full — Result<T>, bounds checks, type safety |
| 1 | Controlled | `sys()` safe syscalls | Curated syscall whitelist |
| 2 | Supervised | `sys!!()` all syscalls | All syscalls, still returns Result |
| 3 | Raw | `sys!!!()`, `wild`, `wildx` | No safety net — you own it |

#### TOS Safety Vocabulary

Explicit bypass keywords that escalate safety level:

| Keyword | Action | Layer |
|---------|--------|-------|
| `raw` / `_!` | Extract value, ignore error | 1+ |
| `drop` / `_?` | Discard Result entirely | 1+ |
| `ok` | Pass potentially unknown value | 1+ |
| `?!` | Emphatic unwrap (failsafe on error) | 0 |
| `wild` | Unmanaged memory allocation | 3 |
| `wildx` | Executable memory allocation | 3 |
| `sys!!!()` | Raw syscall | 3 |

#### Philosophy

Every safety bypass is **visible in code**. There are no hidden undefined behaviors.
When reading Aria code, the `raw`, `wild`, `drop`, `sys!!!` keywords immediately
identify where safety guarantees are intentionally relaxed.

#### Related

- [control_flow/error_flow.md](../control_flow/error_flow.md) — error model overview
- [types/result.md](../types/result.md) — Result<T> safety
- [io_system/sys.md](../io_system/sys.md) — syscall tiers


##### Traits

#### Overview

Traits define shared behavior that types can implement. They serve as constraints
for generic type parameters.

```aria
trait:Describable = {
    func:describe = int32(int32:self);
};
```

#### Implementation

```aria
impl:Describable:for:int32 = {
    func:describe = int32(int32:self) {
        pass self + 100;
    };
};
```

For struct types:

```aria
trait:Measurable = {
    func:area = int32(Rect2D:self);
};

impl:Measurable:for:Rect2D = {
    func:area = int32(Rect2D:self) {
        pass self.w * self.h;
    };
};
```

#### As Generic Constraints

```aria
func<T: Addable>:apply_trait = int32(T:val) {
    pass val;
};
```

#### Dynamic Dispatch — `dyn`

```aria
func:process = int32(dyn Describable:item) {
    pass item.describe();
};
```

#### Borrow Receivers — `$$i` and `$$m`

Trait method parameters can use borrow qualifiers to express ownership semantics:

| Qualifier | Meaning | Aliasing Rule |
|-----------|---------|---------------|
| `$$i` | Immutable borrow (shared) | Multiple `$$i` borrows allowed simultaneously |
| `$$m` | Mutable borrow (exclusive) | Only one `$$m` borrow at a time, no concurrent `$$i` |

##### Syntax

Borrow qualifiers appear before the type in trait method parameter lists:

```aria
trait:Transformable = {
    func:scale = int32($$i Transformable:self, int32:factor);
    func:mutate = int32($$m Transformable:self, int32:value);
};
```

- `$$i Transformable:self` — borrows `self` immutably (read-only access)
- `$$m Transformable:self` — borrows `self` mutably (read-write access)

##### Implementation

```aria
impl:Transformable:for:Rect2D = {
    func:scale = int32($$i Rect2D:self, int32:factor) {
        // self is immutable — cannot modify fields
        pass self.w * factor;
    };

    func:mutate = int32($$m Rect2D:self, int32:value) {
        // self is mutable — can modify fields
        pass value;
    };
};
```

##### When To Use Each

| Use `$$i` when | Use `$$m` when |
|----------------|----------------|
| Method only reads data | Method modifies the receiver |
| Want to allow shared access | Need exclusive access |
| Pure computation / getters | Setters / state mutation |

##### Interaction with the Borrow Checker

The compiler enforces borrow rules at compile time:

```aria
// OK: multiple immutable borrows
$$i Rect2D:ref1 = ...;
$$i Rect2D:ref2 = ...;  // fine, both are $$i

// ERROR: mutable + immutable at same time
$$m Rect2D:ref1 = ...;
$$i Rect2D:ref2 = ...;  // compile error: cannot borrow while $$m exists
```

##### The `#` Pin Operator

The pin operator `#` guarantees a variable's memory address stays stable. This
prevents the GC from relocating pinned data — essential for zero-copy FFI:

```aria
#my_struct;  // pin: address will not move
```

Pinning is orthogonal to borrowing: a pinned value can still be borrowed with
`$$i` or `$$m`, but its address is guaranteed stable for the pin's lifetime.

#### Status

Traits are specified in the language design. Implementation status varies —
see the RFC at `META/ARIA/TRAITS_AND_BORROW_SEMANTICS_RFC.md`.

#### Related

- [rules.md](rules.md) — refinement types (`Rules<T>`, `limit<>`)
- [functions/generics.md](../functions/generics.md) — generic functions


##### Verification in Aria — Formal Proofs with Z3

#### Overview

Aria integrates the Z3 SMT solver to verify program properties at compile time.
Verification is opt-in via `--verify` flags and covers six domains: Rules/limit
constraints, function contracts, integer overflow, concurrency safety, memory safety,
and user-driven proofs.

When a property is **Proven**, the compiler can eliminate redundant runtime checks.
When **Disproven**, a compile-time diagnostic is emitted with a counterexample.
When **Unknown** (solver timeout), the runtime check is preserved.

#### Compiler Flags

| Flag | Purpose |
|------|---------|
| `--verify` | Enable all verification phases |
| `--verify-contracts` | Verify `requires`/`ensures` contracts |
| `--verify-overflow` | Prove arithmetic won't overflow, eliminate checks |
| `--verify-concurrency` | Detect data races and deadlocks |
| `--verify-memory` | Detect use-after-free bugs |
| `--smt-opt` | Enable SMT-guided optimizations (implies `--verify`) |
| `--smt-timeout=N` | Per-query Z3 timeout in milliseconds (default: 5000) |
| `--prove-report` | Proven/Disproven/Unknown per check |

Use `--verify` alone to enable everything, or combine individual flags:

```bash
ariac program.aria -o program \
    --verify-contracts --verify-overflow --prove-report
```

#### 1. Rules & Limit Verification

Rules define value constraints. The compiler uses Z3 to verify that every `limit<>`
binding satisfies its Rules at compile time.

```aria
Rules<int32>:r_positive = { $ > 0 };
Rules<int32>:r_bounded  = { $ >= 0, $ <= 1000 };

func:main = int32() {
    limit<r_positive> int32:x = 42;      // Proven: 42 > 0
    limit<r_bounded>  int32:y = 500;     // Proven: 500 >= 0 AND 500 <= 1000
    limit<r_positive> int32:z = -1;      // Disproven: -1 > 0 is false
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

Compile with `--verify --prove-report`:

```
[limit] Proven:  'x' satisfies r_positive
[limit] Proven:  'y' satisfies r_bounded
[limit] Disproven: 'z' violates r_positive — counterexample: $ = -1
```

##### Rules Propagation

When **all** callers of a function pass `limit<>`-constrained arguments, the compiler
propagates those constraints to the callee and eliminates redundant checks:

```aria
Rules<int32>:r_safe = { $ >= 0, $ <= 100 };

func:process = int32(int32:val) {
    pass val * 2;  // overflow check eliminated
               // if all callers use limit<r_safe>
}

func:main = int32() {
    limit<r_safe> int32:a = 50;
    int32:result = raw process(a);
    exit 0;
}

func:failsafe = int32(tbb32:err) {
    exit 1;
}
```

##### Rules Narrowing

If `r_child` constraints are a subset of `r_parent`, the compiler proves subsumption:

```aria
Rules<int32>:r_positive       = { $ > 0 };
Rules<int32>:r_small_positive = { limit<r_positive>, $ < 100 };

// Passing r_small_positive where r_positive is expected: Proven safe
```

#### 2. Function Contracts

`requires` (preconditions) and `ensures` (postconditions) are verified across call
sites using Z3.

```aria
func:safe_divide = flt64(int32:a, int32:b)
    requires b != 0
{
    pass (a / b);
};

func:clamp = int32(int32:val, int32:lo, int32:hi)
    requires lo <= hi
    ensures result >= lo
{
    if (val < lo) { pass lo; };
    if (val > hi) { pass hi; };
    pass val;
};
```

##### Cross-Function Contract Propagation

When function A calls function B, the compiler checks that A's context satisfies B's
`requires` clause:

```aria
func:make_positive = int32(int32:x)
    requires x > 0
    ensures result > 0
{ pass x; };

func:double_positive = int32(int32:y)
    requires y > 0
{
    int32:pos = raw make_positive(y);  // Proven: y > 0 satisfies requires x > 0
    pass pos * 2;
};
```

##### Loop Invariants

```aria
func:sum_to_n = int32(int32:n) requires n >= 0 {
    int32:s = 0;
    int32:i = 0;
    while (i < n) invariant s >= 0, i >= 0 {
        s = s + i;
        i = i + 1;
    };
    pass s;
};
```

#### 3. Overflow Elimination

With `--verify-overflow` or `--smt-opt`, the compiler uses Rules constraints to prove
arithmetic is within bounds and eliminates overflow checks:

```aria
Rules<int32>:SmallPos = { $ >= 0, $ <= 1000 };

func:safe_add = int32() {
    limit<SmallPos> int32:a = 500;
    limit<SmallPos> int32:b = 300;
    int32:result = a + b;    // Proven safe: max 1000+1000 = 2000 < INT32_MAX
    pass result;             // overflow check ELIMINATED
};

func:risky_mul = int32() {
    limit<SmallPos> int32:m = 999;
    limit<SmallPos> int32:n = 999;
    int32:big = m * n;  // Unknown: can't prove all
    pass big;           // overflow check KEPT
};
```

#### 4. Concurrency Verification

With `--verify-concurrency`, the compiler detects data races on shared variables and
deadlocks from inconsistent lock ordering.

##### Data Race Detection

```aria
int32:counter = 0i32;
extern func:aria_shim_mutex_lock = int32(int64:mtx);
extern func:aria_shim_mutex_unlock = int32(int64:mtx);

// UNSAFE — shared write without lock
func:unsafe_writer = int64(int64:arg) {
    counter = counter + 1i32;          // Disproven: data race
    pass 0i64;
};

// SAFE — mutex-guarded write
func:safe_writer = int64(int64:arg) {
    drop aria_shim_mutex_lock(global_mtx);
    counter = counter + 1i32;          // Proven: race-free
    drop aria_shim_mutex_unlock(global_mtx);
    pass 0i64;
};
```

##### Deadlock Detection

```aria
// UNSAFE — cyclic lock ordering
func:lock_ab = NIL() {
    lock(mtx_a); lock(mtx_b);
    unlock(mtx_b); unlock(mtx_a);
};
func:lock_ba = NIL() {
    lock(mtx_b); lock(mtx_a);
    unlock(mtx_a); unlock(mtx_b);
};
// Disproven: potential deadlock — A→B then B→A

// SAFE — consistent ordering
func:lock_ab2 = NIL() {
    lock(mtx_a); lock(mtx_b);
    unlock(mtx_b); unlock(mtx_a);
};
func:lock_ab3 = NIL() {
    lock(mtx_a); lock(mtx_b);
    unlock(mtx_b); unlock(mtx_a);
};
// Proven: deadlock-free
```

#### 5. Memory Safety

With `--verify-memory`, the compiler tracks allocation/free state of `wild` pointers
and detects use-after-free bugs.

```aria
// UNSAFE — use after free
func:test_uaf = NIL() {
    wild ?->:buf = malloc(64i64);
    free(buf);
    wild ?->:alias = buf;     // Disproven: buf is freed
};

// SAFE — alias before free
func:test_safe = NIL() {
    wild ?->:buf = malloc(64i64);
    wild ?->:alias = buf;     // Proven: buf is live
    free(buf);
};

// CONDITIONAL — path-dependent UAF
func:test_cond_uaf = NIL(int32:flag) {
    wild ?->:buf = malloc(64i64);
    if (flag > 0i32) { free(buf); };
    wild ?->:use = buf;       // Disproven: buf might be freed on flag > 0 path
};
```

#### 6. prove and assert_static

User-driven compile-time proof directives:

```aria
Rules<int32>:Percentage = { $ >= 0, $ <= 100 };

func:example = NIL() {
    limit<Percentage> int32:pct = 50;

    prove(pct >= 0);           // Proven at compile time → erased from binary
    prove(pct <= 100);         // Proven at compile time → erased from binary

    assert_static(pct > 10);   // Proven → erased
    assert_static(pct > 99);   // Unknown → warning, kept as runtime assert
};
```

| Directive | On Proven | On Disproven | On Unknown |
|-----------|-----------|--------------|------------|
| `prove(expr)` | Erased | Compile error | Compile error |
| `assert_static(expr)` | Erased | Warn+assert | Warn+assert |

#### SMT-Guided Optimizations (--smt-opt)

When `--smt-opt` is passed, the compiler uses Z3 proofs to perform additional
optimizations beyond safety checking:

- **Overflow check elimination** — proven-safe arithmetic skips runtime overflow guards
- **Null check elimination** — `Optional` from non-null sources skips nil checks
- **Loop-invariant hoisting** — expressions proven constant across iterations
- **Defaults fallback elimination** — infallible functions skip `?|` fallback code
- **Rules propagation** — inter-procedural constraint propagation

##### Performance Impact

Typical overhead and speedup from the benchmark suite:

| Metric | Value |
|--------|-------|
| Compile-time overhead (per flag) | 9–19% |
| Compile-time overhead (--verify-memory) | ~114% (small files) |
| Runtime speedup (--smt-opt, Rules) | ~3% |
| Runtime speedup (--smt-opt, overflow) | ~16% |
| Runtime speedup (--smt-opt, user stack) | ~25% |

#### Timeout Tuning

The default per-query timeout is 5000ms. Reduce for faster compilation at the cost
of more Unknown results; increase for complex proofs:

```bash
ariac program.aria -o program \
    --verify --smt-timeout=2000   # faster
ariac program.aria -o program \
    --verify --smt-timeout=10000  # slower
```

#### Notes

- Verification requires the compiler built with `-DARIA_HAS_Z3=ON`
- The Z3 context is created once per compilation and shared across all queries
- Each verification query creates a lightweight solver from the shared context
- `--prove-report` prints a summary of all Proven/Disproven/Unknown verdicts
- Rules, contracts, and `prove`/`assert_static` compose — use them together
- Disproven results include counterexamples when available

#### Best Practices

1. **Start with contracts** — Add `requires`/`ensures` to public-facing functions first.
   They document intent and catch bugs at call sites.

2. **Use `limit<Rules>` at boundaries** — Apply Rules constraints where data enters
   the system (user input, file reads, network data). The solver propagates them inward.

3. **Prefer `prove` for critical invariants** — If a property MUST hold, use `prove()`.
   A compile error on failure is better than a runtime surprise.

4. **Use `assert_static` for desirable invariants** — When a property should hold but
   isn't critical, `assert_static` gives a warning and keeps a runtime fallback.

5. **Build with `--smt-opt` in release** — Let proven-safe checks be eliminated.
   The ~3–25% runtime speedup is free after paying the compile-time cost.

6. **Tune timeouts per project** — Small projects: `--smt-timeout=2000`. Large projects
   with complex proofs: `--smt-timeout=10000` or higher.

7. **Use `--prove-report` during development** — See exactly what's Proven vs Unknown.
   Target Unknown results for optimization or stronger annotations.

8. **Compose constraints** — Rules, contracts, and prove/assert_static work together.
   A `limit<r>` binding inside a function with `requires` gives the solver more to work with.

#### Related

- [rules.md](rules.md) — Rules & Limit syntax
- [../functions/design_by_contract.md](../functions/design_by_contract.md) — requires/ensures
- [safety_layers.md](safety_layers.md) — Aria's safety model
- [concurrency.md](concurrency.md) — threading and mutexes


##### WebAssembly Target

#### Overview

Aria can target WebAssembly (WASM) via LLVM's wasm32 backend.

#### Limitations

Current WASM limitations:
- No direct syscall access (`sys()` not available)
- No `wildx` (executable memory)
- No POSIX threads (single-threaded only)
- Limited file I/O (browser sandbox)
- No FFI with native C libraries

#### Building for WASM

```bash
ariac --target wasm32 source.aria -o output.wasm
```

#### Related

- [functions/extern.md](../functions/extern.md) — FFI (not available in WASM)
- [io_system/sys.md](../io_system/sys.md) — syscalls (not available in WASM)

\newpage

## Debugging


##### Debugging

#### Debug Output

Use the `stddbg` stream (FD 3) for debug output that doesn't mix with stdout/stderr.

#### Print Debugging

```aria
println(`DEBUG: x = &{x}, y = &{y}`);
```

#### Type Inspection

- `atype(handle)` — type tag of top astack element
- `ahtype(handle, key)` — type tag of ahash entry

#### Common Issues

See `META/ARIA/KNOWN_ISSUES.md` for current known issues and workarounds.

##### Constant Arithmetic Bug
`fixed` with arithmetic expressions evaluates to 0. Compute in a variable:

```aria
// WRONG: fixed int64:NEG1 = 0i64 - 1i64;   // → 0
// RIGHT: int64:neg1 = (0i64 - 1i64);
```

##### String Literal in sys()
Store in variable first:

```aria
// WRONG: sys(SYS_OPEN, "file.txt", 0i64, 0i64);
// RIGHT: string:path = "file.txt";
//        sys(SYS_OPEN, path, 0i64, 0i64);
```

\newpage

# Part III: Safety Walkthrough

#### Safety-Critical Development in Aria: A Walkthrough

**Version:** 0.3.4  
**Date:** 2026-04-10  
**Status:** Living document  

---

#### Table of Contents

1. [Introduction](#1-introduction)
2. [The Application: Medical Infusion Pump](#2-the-application-medical-infusion-pump)
3. [Feature 1: failsafe — The Mandatory Safety Net](#3-feature-1-failsafe--the-mandatory-safety-net)
4. [Feature 2: Result\<T\> — No Silent Failures](#4-feature-2-resultt--no-silent-failures)
5. [Feature 3: TBB — Sticky Error Propagation](#5-feature-3-tbb--sticky-error-propagation)
6. [Feature 4: limit\<Rules\> — Z3 Verification](#6-feature-4-limitrules--value-constraints-with-z3-verification)
7. [Feature 5: Borrow Semantics](#7-feature-5-borrow-semantics--compile-time-memory-safety)
8. [Feature 6: Emergency Operators — ?! and !!!](#8-feature-6-emergency-operators---and-)
9. [Feature 7: wild — Controlled Unsafe Access](#9-feature-7-wild--controlled-unsafe-access)
10. [Feature 8: sys() — Tiered Syscall Safety](#10-feature-8-sys--tiered-syscall-safety)
11. [Feature 9: _? and _! Shorthand](#11-feature-9-_-and-_--ergonomic-drop--raw-shorthand)
12. [Complete Pump Controller](#12-putting-it-together-the-complete-pump-controller)
13. [Z3 Verification in Practice](#13-z3-verification-in-practice)
14. [Comparison with Other Approaches](#14-comparison-with-other-approaches)
15. [Summary](#15-summary)

---

#### 1. Introduction

Safety-critical software — code that controls medical devices, aircraft, nuclear
reactors, and autonomous vehicles — has a fundamental problem: **the consequences
of a bug are not a stack trace, they are injury or death.**

Traditional approaches to this problem fall into two camps:

1. **Restricted subsets** (MISRA C, SPARK/Ada): take an existing language, ban
   the dangerous parts, and add external analysis tools.
2. **Formal methods bolted on** (Frama-C, CBMC): write normal C, then attempt
   to prove properties after the fact with separate tools.

Both approaches share a weakness: **safety is an afterthought.** The language
itself does not enforce safety — external tools and coding standards do. A
developer who forgets to run the checker, or who disables a warning, is back to
writing unsafe code.

Aria takes a different approach: **safety is structural.** The compiler itself
refuses to produce a binary that lacks mandatory error handling, that uses
unchecked results, or that violates declared constraints. You cannot forget to
handle errors because the compiler will not let you. You cannot silently overflow
a safety-critical integer because the type system traps it.

This walkthrough demonstrates Aria's safety features through a realistic
application — a medical drug infusion pump controller — building up from
individual features to a complete, compilable program.

#### Who This Document Is For

- Safety engineers evaluating Aria for safety-critical domains
- Language researchers comparing error-handling and verification models
- Developers curious about Aria's safety guarantees and how they differ from
  Rust, Ada/SPARK, or traditional C/C++ approaches

#### Prerequisites

- The `ariac` compiler (v0.3.4+)
- Basic familiarity with imperative programming
- No prior Aria experience required

#### Companion Files

Each section references a standalone, compilable example in the `examples/`
directory:

| File | Feature |
|------|---------|
| `01_failsafe_basics.aria` | Mandatory failsafe handler |
| `02_result_handling.aria` | Result\<T\> with pass/fail |
| `03_tbb_propagation.aria` | TBB sticky error types |
| `04_limit_rules.aria` | limit\<Rules\> constraints |
| `05_borrow_safety.aria` | Borrow semantics ($$i/$$m) |
| `06_emergency_operators.aria` | ?! and !!! operators |
| `07_infusion_pump.aria` | Complete pump controller |
| `08_syscall_safety.aria` | sys() tiered syscall safety |

---

#### 2. The Application: Medical Infusion Pump

A drug infusion pump is a device that delivers fluids — typically medication —
into a patient's body at a controlled rate. Infusion pump software failures have
caused real patient deaths and are a frequent subject of FDA recalls.

Common failure modes include:

- **Overdose:** software calculates too high a flow rate
- **Silent sensor failure:** a disconnected sensor returns stale data
- **Arithmetic overflow:** a rate calculation wraps around to a plausible but
  wrong value
- **Memory corruption:** a pointer bug overwrites the dosage register
- **Missing error handling:** an error return is ignored, execution continues
  with garbage data

Our example pump controller will demonstrate how each of Aria's safety features
addresses one or more of these failure modes.

The controller performs a single infusion cycle:

```
1. Load and validate patient data
2. Read sensors (temperature, pressure, flow rate)
3. Calculate weight-based dosage using TBB arithmetic
4. Apply limit<Rules> to constrain the final dose
5. Deliver or enter safe mode based on error count
```

---

#### 3. Feature 1: failsafe — The Mandatory Safety Net

#### The Problem

In C, error-handling functions like `atexit()` or signal handlers are optional.
A developer can write a program with no error handling at all, and the compiler
will happily produce a binary. In safety-critical systems, this is unacceptable.

#### Aria's Solution

Every Aria program **must** define a `failsafe` function. The compiler checks
for its existence and refuses to compile if it is missing. This is enforced at
the analysis phase (BUG-005 fix, v0.3.4), not by convention or linting.

#### Syntax

```aria
func:failsafe = int32(tbb32:err) {
    // err: the TBB error code that triggered this handler
    // Body MUST call exit() — failsafe cannot "resume"
    exit(1);
};
```

#### Key Properties

| Property | Detail |
|----------|--------|
| **Mandatory** | Compiler error if missing |
| **Signature** | `int32(tbb32:err)` — fixed, not configurable |
| **Error code** | `tbb32` — uses TBB type so it cannot be accidentally corrupted |
| **Return** | Must call `exit(code)` — this is a terminal handler |
| **Invocation** | Called automatically on unhandled errors, or explicitly via `!!!` |

#### Example

*(See `examples/01_failsafe_basics.aria`)*

```aria
func:failsafe = int32(tbb32:err) {
    print("EMERGENCY: failsafe triggered\n");
    exit(1);
};

func:main = int32() {
    print("System online — failsafe registered.\n");
    int32:status = 0;
    if (status == 0) {
        print("All subsystems nominal.\n");
    }
    exit(0);
};
```

#### Why This Matters

In traditional safety-critical C (IEC 62304, DO-178C), external coding
standards mandate error handling — but the compiler itself does not enforce them.
Aria makes the guarantee structural: **no failsafe, no binary.**

---

#### 4. Feature 2: Result\<T\> — No Silent Failures

#### The Problem

In C, functions signal errors through return codes, `errno`, or null pointers.
All of these can be silently ignored:

```c
FILE *f = fopen("data.bin", "r");  // f might be NULL
fread(buf, 1, 100, f);             // UB if f is NULL — no compiler warning
```

In C++, exceptions can be unhandled. In Go, errors are values that can be
discarded with `_`.

#### Aria's Solution

Functions that can fail return `Result<T>` implicitly. The caller **must** check
`.is_error` before accessing `.value` — the compiler enforces this rule (called
"no-checky-no-val"). Accessing `.value` without a prior `.is_error` check is a
compile error.

#### Syntax

```aria
// Declaring a function that can fail:
func:read_sensor = int32() {
    pass(42i32);    // success — wraps 42 in Result<int32>
    fail(1i32);     // error   — wraps 1 as error code
};

// Calling and checking:
Result<int32>:r = read_sensor();
if (r.is_error) {
    int32:code = r.error;    // safe — checked first
} else {
    int32:val = r.value;     // safe — checked first
}
```

#### Error Codes

Aria defines standardized error code ranges:

| Range | Meaning | Examples |
|-------|---------|----------|
| `ERR` | Unknown/unclassified | TBB min sentinel |
| `0` | No error / success | — |
| `-1` to `-16` | System errors | -1 SYS_GENERAL, -3 SYS_IO, -7 SYS_OVERFLOW |
| `1` to `10` | User-defined errors | 1 USR_GENERAL, application-specific |

#### Example

*(See `examples/02_result_handling.aria`)*

```aria
func:read_temperature = int32() {
    pass(365i32);    // 36.5 °C in tenths
};

func:read_pressure = int32() {
    fail(1i32);     // sensor offline
};

func:main = int32() {
    Result<int32>:temp = read_temperature();
    if (temp.is_error) {
        print("Temperature sensor failed\n");
    } else {
        int32:t = temp.value;
        print("Temperature OK\n");
    }

    Result<int32>:pres = read_pressure();
    if (pres.is_error) {
        int32:code = pres.error;
        print("Pressure sensor offline\n");
    } else {
        int32:p = pres.value;
        print("Pressure nominal\n");
    }
    exit(0);
};
```

#### Why This Matters

The compiler guarantees at the type level that **every error path is explicitly
handled**. There is no mechanism to silently discard an error — no `_`, no
unchecked return codes, no unhandled exceptions.

---

#### 5. Feature 3: TBB — Sticky Error Propagation

#### The Problem

In IEEE 754 floating-point, `NaN` propagates through arithmetic: `NaN + 1 = NaN`.
This is valuable — it prevents corrupted data from producing plausible results.
But integers have no equivalent. In C, `INT_MAX + 1` is undefined behavior. In
most languages, integer overflow silently wraps to a negative number.

This is catastrophic in safety-critical systems. Imagine a dosage calculation:

```
dose = patient_weight * rate_per_kg
```

If `patient_weight` overflows to a small positive number due to a sensor bug,
the dose might appear valid but be dramatically wrong.

#### Aria's Solution

TBB (Trusted Balanced Bitfield) types are integers with a **sentinel error
value** at the type minimum. Once a TBB variable enters the ERR state — through
overflow, underflow, or division by zero — it **stays** in ERR through all
subsequent arithmetic.

| Type | ERR Sentinel |
|------|-------------|
| `tbb8` | -128 |
| `tbb16` | -32768 |
| `tbb32` | -2147483648 |
| `tbb64` | -9223372036854775808 |

#### Sticky Behavior

The critical property is that **errors never heal:**

```
ERR + 1     = ERR      (not a valid number)
ERR * 0     = ERR      (not zero — errors don't annihilate!)
ERR - ERR   = ERR      (not zero — errors don't cancel!)
1 + ERR     = ERR      (commutativity preserved)
```

This is fundamental. In C, `INT_MIN * 0 = 0`, which means an overflow error can
silently disappear. TBB guarantees this cannot happen.

#### Example

*(See `examples/03_tbb_propagation.aria`)*

```aria
func:main = int32() {
    // Normal arithmetic works as expected
    tbb32:sensor_a = 100;
    tbb32:sensor_b = 200;
    tbb32:sum = sensor_a + sensor_b;    // 300

    // Overflow produces ERR
    tbb32:big = 2147483647;
    tbb32:one = 1;
    tbb32:overflow = big + one;          // ERR (-2147483648)

    // ERR is sticky — it NEVER heals
    tbb32:still_bad = overflow + sensor_a;  // ERR
    tbb32:zero = 0;
    tbb32:mul_zero = overflow * zero;       // ERR (not zero!)
    tbb32:self_sub = overflow - overflow;   // ERR (not zero!)

    exit(0);
};
```

#### Application: Dosage Calculation

In the infusion pump controller, the dose calculation uses TBB arithmetic:

```aria
tbb32:w = weight_kg;
tbb32:base = rate_per_kg;
tbb32:dose = w * base;

tbb32:err_sentinel = -2147483648;
if (dose == err_sentinel) {
    fail(-7);   // SYS_OVERFLOW — propagate error up
}
```

If **any** input is corrupted, the output is guaranteed to be ERR. The pump
cannot deliver a corrupted dose because the ERR sentinel will be caught by the
check, or by the `limit<Rules>` constraint (which rejects ERR values since
they are negative and outside the valid dose range).

---

#### 6. Feature 4: limit\<Rules\> — Value Constraints with Z3 Verification

#### The Problem

Safety-critical systems have **value constraints** that must never be violated:
a drug dose must not exceed the maximum, a motor speed must stay below the
redline, a reactor temperature must remain within safe bounds.

In traditional languages, these constraints live in comments, documentation,
or runtime assertions that can be forgotten, disabled, or bypassed.

#### Aria's Solution

Aria's `Rules` declarations define named constraints. The `limit<RuleName>`
annotation on a variable means the compiler and runtime enforce those constraints
at every assignment. With Z3 verification (`--verify`), the compiler can
**prove** constraints at compile time — before the program ever runs.

#### Syntax

```aria
// Scalar constraint: dose must be 1–5000 mg
Rules:r_safe_dose = {
    $ > 0,
    $ <= 5000
};

// Struct field constraint: patient must be adult
Rules<Patient>:r_adult = {
    $.age_years >= 18,
    $.weight_kg >= 30
};

// Cascading: pediatric dose inherits safe dose + adds upper bound
Rules:r_pediatric = {
    limit<r_safe_dose>,
    $ <= 500
};

// Apply to a variable:
limit<r_safe_dose> int32:dose = 250;   // OK — 250 is in [1, 5000]
limit<r_safe_dose> int32:bad  = 9999;  // Runtime error → failsafe
```

#### Z3 Verification

When compiled with `--verify`, the compiler translates Rules constraints into Z3
SMT assertions and attempts to prove them statically:

```bash
ariac --verify 04_limit_rules.aria
```

If the compiler can prove a constraint is always satisfied, no runtime check is
needed. If it finds a counter-example, it reports the violating values. If it
can neither prove nor disprove, a runtime check is emitted as a safety net.

#### Example

*(See `examples/04_limit_rules.aria`)*

```aria
Rules:r_safe_dose_mg = {
    $ > 0,
    $ <= 5000
};

Rules:r_pediatric_dose = {
    limit<r_safe_dose_mg>,
    $ <= 500
};

struct:Patient = {
    int32:weight_kg;
    int32:age_years;
};

Rules<Patient>:r_adult_patient = {
    $.weight_kg >= 30,
    $.age_years >= 18
};

func:main = int32() {
    limit<r_safe_dose_mg> int32:dose = 250;              // OK
    limit<r_pediatric_dose> int32:child_dose = 100;       // OK (cascade)

    stack Patient:pat = Patient{weight_kg: 80, age_years: 35};
    limit<r_adult_patient> Patient:adult = pat;           // OK
    exit(0);
};
```

#### Application: Multi-Layer Dosage Protection

In the infusion pump, constraints form a multi-layer defense:

1. **Weight constraint** (`r_valid_weight`): patient weight 1–300 kg
2. **Dose constraint** (`r_safe_dose`): final dose 1–5000 mg
3. **Patient eligibility** (`r_eligible_patient`): age ≥ 18, weight ≥ 30 kg
4. **Flow constraint** (`r_safe_flow`): delivery rate 1–1000 mL/hr

A dosage that somehow passes the calculation check, passes the
patient-max-dose clamp, but falls outside 1–5000 mg would be caught by
`limit<r_safe_dose>` and trigger the failsafe handler.

#### Why This Matters

Unlike runtime assertions in C (`assert(dose <= 5000)`) which can be compiled
out with `NDEBUG`, Aria's constraints are part of the type system. They cannot
be disabled. With Z3, they can be verified at compile time — providing the
strongest possible guarantee without runtime cost.

---

#### 7. Feature 5: Borrow Semantics — Compile-Time Memory Safety

#### The Problem

Memory corruption is the leading cause of security vulnerabilities (per CISA,
NSA, and Microsoft). In safety-critical systems, a corrupted pointer can
overwrite a dosage register, crash a flight controller, or cause a reactor SCRAM.

C and C++ provide no structural protection against:
- Use-after-free
- Data races from concurrent aliased mutation
- Dangling pointers

#### Aria's Solution

Aria uses explicit borrow annotations with two rules enforced at compile time:

| Annotation | Meaning | Rule |
|-----------|---------|------|
| `$$i Type:ref = var;` | Immutable borrow | Any number may coexist |
| `$$m Type:ref = var;` | Mutable borrow | At most one; no $$i borrows on same variable |

This prevents the fundamental aliasing problem: you can never have a mutable
reference coexisting with any other reference to the same data.

#### Example

*(See `examples/05_borrow_safety.aria`)*

```aria
func:main = int32() {
    int32:sensor_reading = 42;

    // Multiple immutable borrows — ALLOWED
    $$i int32:view_a = sensor_reading;
    $$i int32:view_b = sensor_reading;
    int32:sum = view_a + view_b;   // 84

    // Single mutable borrow — write-through
    int32:actuator_position = 0;
    $$m int32:control = actuator_position;
    control = 100;
    // actuator_position is now 100

    exit(0);
};
```

#### Compile-Time Rejection

The compiler rejects conflicting borrows:

```aria
int32:x = 10;
$$i int32:reader = x;
$$m int32:writer = x;   // COMPILE ERROR: $$i borrow active on x
```

This is checked **at compile time** — no runtime overhead, no possibility of
the check being disabled.

#### Application: Safe Patient Data Access

In the infusion pump, patient data is accessed through immutable borrows:

```aria
$$i int32:pat_weight = safe_weight;
$$i int32:pat_max = checked_patient.max_dose_mg;
```

This guarantees that the dosage calculation cannot accidentally modify patient
data. If a future developer tries to write through these references, the
compiler will reject the code.

---

#### 8. Feature 6: Emergency Operators — ?! and !!!

#### The Problem

Error handling in safety-critical systems must be both **convenient** (so
developers actually use it) and **auditable** (so reviewers can trace every
error path). Exception-based languages make error handling convenient but
difficult to audit. Return-code languages make error handling auditable but
tedious.

#### Aria's Solution

Aria provides two operators that form a **two-level error escalation system**:

| Operator | Name | Level | Behavior |
|----------|------|-------|----------|
| `?!` | Emphatic unwrap | Local | Unwrap Result; use default on error |
| `!!!` | Direct failsafe | Global | Immediately invoke failsafe handler |

**Level 1 — `?!` (local recovery):** "I can handle this with a safe default."

```aria
Result<int32>:sensor = read_sensor();
int32:value = sensor ?! safe_default;
// On success: value = sensor.value
// On error:   value = safe_default
```

**Level 2 — `!!!` (global escalation):** "This is catastrophic. Emergency stop."

```aria
!!! -2;   // Immediately calls failsafe(-2), program terminates
```

#### Example

*(See `examples/06_emergency_operators.aria`)*

```aria
// Level 1: Local recovery with ?!
Result<int32>:ok_reading = read_sensor_ok();
int32:val_ok = ok_reading ?! 50i32;    // val_ok == 98 (unwrapped)

Result<int32>:bad_reading = read_sensor_fail();
int32:val_bad = bad_reading ?! 50i32;  // val_bad == 50 (default)

// Level 2: Escalate after exhausting local recovery
if (consecutive_failures >= 3) {
    !!! -2;   // trigger failsafe — emergency shutdown
}
```

#### Application: Sensor Fallback Cascade

In the infusion pump, ?! provides graceful degradation:

```aria
// Temperature: use body-temp default if sensor fails
int32:temperature = temp_result ?! 370i32;

// Pressure: use 100 mmHg default if sensor fails  
int32:pressure = pres_result ?! 100i32;

// Flow rate: use 0 (safe stop) if sensor fails
int32:flow = flow_result ?! 0i32;
```

Each sensor has a domain-specific safe default. If too many sensors fail, the
controller enters safe mode rather than delivering an uncertain dose.

#### Auditability

Both operators are syntactically distinctive — `?!` and `!!!` stand out in code
review. A security auditor can grep for these tokens to find every error recovery
and escalation point in the codebase.

---

#### 9. Feature 7: wild — Controlled Unsafe Access

#### The Problem

Safety-critical systems sometimes need to interact with hardware registers,
DMA buffers, or foreign-function interfaces that require raw pointer access.
In Rust, this is `unsafe {}`. In Ada, it is `pragma Import`. The challenge is
making unsafe code **visible and auditable** while still allowing it when needed.

#### Aria's Approach

The `wild` keyword declares raw, unmanaged pointers that bypass the borrow
checker:

```aria
wild void->:hw_reg = @extern("get_hardware_register", 0x40001000);
```

`wild` pointers are:
- **Explicitly marked** — they cannot be confused with safe references
- **Grep-able** — every use of `wild` in a codebase is an audit point
- **Unrestricted** — they support pointer arithmetic, casts, and FFI

The philosophy is the same as Rust's `unsafe`: make the dangerous parts visible
so reviewers know exactly where to focus.

#### When To Use wild

| Use Case | Example |
|----------|---------|
| Hardware register access | `wild uint32->:reg = ...;` |
| FFI with C libraries | `extern func:malloc = wild void->(int64:size);` |
| DMA buffer management | `wild uint8->:dma_buf = ...;` |
| Performance-critical paths | Manual memory layouts |

In the infusion pump controller, `wild` would be used for the actual hardware
interface layer — reading ADC registers, controlling valve GPIOs, and
communicating over CAN bus. The example code uses simulated functions instead
to keep the demonstrations portable and compilable without hardware.

---

#### 10. Feature 8: sys() — Tiered Syscall Safety

*(See `examples/08_syscall_safety.aria`)*

Direct syscalls bypass libc — one wrong argument can corrupt memory or kill a
process. Aria applies its TOS escalation model to make syscall danger visible
and intentional.

#### The Three Tiers

```aria
sys(WRITE, 1i64, "safe\n", 5i64);      // Safe tier — curated whitelist
sys!!(FORK);                            // Full tier — all syscalls allowed
sys!!!(1i64, 1i64, "raw\n", 4i64);     // Raw tier — bare int64, no wrapping
```

| Tier | Return | Safety | Use Case |
|------|--------|--------|----------|
| `sys()` | `Result<int64>` | Compile-time whitelist (~55 safe syscalls) | Library code |
| `sys!!()` | `Result<int64>` | All syscalls, named constants required | Process control |
| `sys!!!()` | `int64` | No validation, any expression | Bare metal |

#### Compile-Time Enforcement

The key innovation: `sys()` only accepts named constants from a curated
whitelist. You cannot sneak a dangerous syscall through with a variable:

```aria
sys(WRITE, fd, buf, len);    // OK — WRITE is in safe set
sys(FORK);                   // COMPILE ERROR: FORK not in safe set
int64:nr = 57i64;
sys(nr, 0i64);               // COMPILE ERROR: variables not allowed
```

The error message guides you to the right tier:
> `error: 'FORK' is not in the safe syscall set. Use sys!!('FORK', ...) for full access.`

#### Why This Matters for Safety-Critical Systems

1. **Auditable** — `aria-safety` flags every `sys!!` and `sys!!!` usage
2. **Visible** — `grep 'sys!!!'` finds all raw syscall paths in a codebase
3. **Intentional** — you cannot accidentally use the dangerous tiers
4. **Lazy-safe** — the easiest path (`sys()`) is also the safest

For the infusion pump: sensor reading and file logging use safe-tier `sys()`.
If hardware control needed `IOCTL` (which *is* in the safe set), no escalation
is required. Only truly dangerous operations like process spawning would need
`sys!!` — and that would be immediately visible in review.

---

#### 11. Feature 9: _? and _! — Ergonomic Drop & Raw Shorthand

*(See `examples/09_drop_raw_shorthand.aria`)*

Aria v0.4.0 adds prefix shorthand operators for the two TOS-bypass builtins:

| Shorthand | Desugars to | Operator Family |
|-----------|------------|------------------|
| `_?expr` | `drop(expr)` | `?` family (safe/default) |
| `_!expr` | `raw(expr)` | `!` family (danger/force) |

These are **pure parser sugar** — `_?` and `_!` are converted to `drop()` and `raw()`
call nodes during parsing. The type checker and codegen see the same IR as the
verbose forms. No new semantics, no new risks.

#### Why Both Forms?

Aria keeps verbose and terse forms so developers can choose the right one for context:

```aria
// In a long function where readability matters most:
drop(writeFile("audit.log", entry));
int32:val = raw(verified_result);

// In tight imperative code where the intent is obvious:
_?println("tick");
int32:x = _!safe_add(a, b);
```

#### The ? and ! Families

The shorthand operators fit naturally into Aria's existing operator families:

**? family** (safe / provides fallback / discards):
- `expr ? default` — unwrap with fallback
- `expr??` — null-coalesce
- `_?expr` — drop result entirely

**! family** (danger / force / bypass):
- `expr?!` — emphatic unwrap (calls failsafe on error)
- `expr!!` — force cast
- `!!!` — direct failsafe
- `_!expr` — raw extract without check

#### Safety-Critical Perspective

For the infusion pump, `_?` and `_!` are flagged by `aria-safety` just like their
verbose counterparts. The tool reports `_!` as `[RAW]` severity — the shorthand
does not let you sneak past code review:

```
[RAW]  Line 42: _! raw extraction shorthand — bypasses error check (desugars to raw())
[DROP] Line 58: _? drop shorthand — discards Result without checking (desugars to drop())
```

Both `drop()` and `_?` are fully interchangeable. Both `raw()` and `_!` are fully
interchangeable. The choice is purely stylistic.

---

#### 12. Putting It Together: The Complete Pump Controller

*(See `examples/07_infusion_pump.aria`)*

The complete infusion pump controller combines all features into a coherent
program:

#### Architecture

```
┌─────────────────────────────────────────────────┐
│                   failsafe()                     │
│            Last line of defense                  │
│     Closes valves, sounds alarm, logs error      │
└────────────────────┬────────────────────────────┘
                     │ invoked by !!! or unhandled error
┌────────────────────┴────────────────────────────┐
│                    main()                        │
│                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────┐│
│  │ Load Patient │→│ Read Sensors │→│ Calculate ││
│  │   Data       │  │  (Result<T>) │  │  Dosage  ││
│  │              │  │              │  │  (TBB)   ││
│  │ limit<Rules> │  │ ?! defaults  │  │          ││
│  │ $$i borrows  │  │              │  │          ││
│  └─────────────┘  └─────────────┘  └────┬─────┘│
│                                          │      │
│                                   ┌──────┴─────┐│
│                                   │  Constrain  ││
│                                   │ limit<Rules>││
│                                   │             ││
│                                   │ Deliver or  ││
│                                   │ Safe Mode   ││
│                                   └─────────────┘│
└──────────────────────────────────────────────────┘
```

#### Safety Layers in the Code

| Layer | Feature | What It Prevents |
|-------|---------|-----------------|
| 1 | `failsafe()` | Unhandled errors escaping the program |
| 2 | `Result<T>` | Silent sensor failures |
| 3 | `?!` operator | Sensor failure without safe default |
| 4 | TBB arithmetic | Overflow producing plausible wrong dosage |
| 5 | `limit<Rules>` | Out-of-range dosage reaching the pump |
| 6 | `$$i` borrows | Accidental mutation of patient data |
| 7 | Error counting | Delivering dose with too many unknowns |

#### Walkthrough of Key Sections

**Patient data validation:**
```aria
stack Patient:patient = Patient{
    id: 1042, weight_kg: 72, age_years: 45, max_dose_mg: 2000
};
limit<r_eligible_patient> Patient:checked_patient = patient;
limit<r_valid_weight> int32:safe_weight = checked_patient.weight_kg;

$$i int32:pat_weight = safe_weight;
int32:max_dose_val = checked_patient.max_dose_mg;
$$i int32:pat_max = max_dose_val;
```

Three constraints are applied before any calculation begins: the patient must
be eligible (age/weight), the weight must be in human range, and subsequent
access is read-only through immutable borrows.

**Sensor reading with local recovery:**
```aria
Result<int32>:temp_result = read_temperature();
int32:temperature = temp_result ?! 370i32;   // default: 37.0 °C

Result<int32>:flow_result = read_flow_rate();
int32:flow = flow_result ?! 0i32;            // default: stop delivery
```

Each sensor has a domain-specific default. The flow sensor defaults to 0 (no
delivery) — the safest possible fallback for a failed flow measurement.

**TBB-safe dosage calculation:**
```aria
tbb32:w = weight_kg;
tbb32:base = base_dose_per_kg;
tbb32:dose = w * base;

tbb32:err_sentinel = -2147483648;
if (dose == err_sentinel) {
    fail(-7);   // SYS_OVERFLOW
}
```

If either input is corrupted, the TBB multiplication produces ERR, which is
caught by the sentinel check. The error cannot propagate silently.

**Final constraint enforcement:**
```aria
limit<r_safe_dose> int32:final_dose = clamped;
```

Even after clamping to the patient's maximum, the dose must still pass the
global safe-dose rule (1–5000 mg). This is the last automated check before
delivery.

**Delivery decision:**
```aria
if (errors >= 2) {
    // Multiple sensor failures — do NOT deliver
    exit(0);   // safe mode
}
```

The controller counts errors throughout the cycle. Two or more failures trigger
safe mode — no drug is delivered.

---

#### 12. Z3 Verification in Practice

Aria v0.3.4 includes a built-in Z3 SMT solver integration that can verify
limit\<Rules\> constraints and function contracts at compile time.

#### Usage

```bash
#### Verify all constraints
ariac --verify 07_infusion_pump.aria

#### Generate a detailed verification report
ariac --verify --verify-report pump_report.txt 07_infusion_pump.aria

#### Verify specific categories
ariac --verify-contracts 07_infusion_pump.aria     # requires/ensures
ariac --verify-overflow  07_infusion_pump.aria      # integer overflow
```

#### What Z3 Proves

The Z3 verifier operates in three phases:

1. **Phase 1 — Constraint Verification:** Checks that every `limit<Rules>`
   assignment satisfies its constraints. For constant values, this is a direct
   proof. For computed values, Z3 searches for counter-examples.

2. **Phase 2 — Contract Verification:** Checks `requires` (preconditions)
   and `ensures` (postconditions) on function signatures.

3. **Phase 3 — Overflow Analysis:** Checks integer arithmetic for potential
   overflow, especially on safety-critical types.

#### Verification Report Example

```
=== Z3 Verification Report ===
File: 07_infusion_pump.aria

[PASS] limit<r_safe_dose> at line 156: dose ∈ [1, 5000] — PROVED
[PASS] limit<r_valid_weight> at line 134: weight ∈ [1, 300] — PROVED
[PASS] limit<r_eligible_patient> at line 131: age >= 18, weight >= 30 — PROVED
[PASS] limit<r_safe_flow> — no violations found

Constraints verified: 4/4
Counter-examples: 0
Verification time: 0.023s
```

#### Comparison with External Tools

| Approach | When | Integrated | Incremental |
|----------|------|-----------|-------------|
| SPARK/Ada | Compile time | [Y] | [Y] |
| Frama-C/WP | Post-hoc | [N] | [N] |
| CBMC | Post-hoc | [N] | [N] |
| **Aria + Z3** | **Compile time** | **[Y]** | **[Y]** |

Aria's Z3 integration runs as part of the normal compilation pipeline. No
separate tool invocation, no configuration files, no annotation language
different from the source language.

---

#### 14. Comparison with Other Approaches

#### How Aria Compares to Safety-Critical Alternatives

| Feature | C (MISRA) | Ada/SPARK | Rust | **Aria** |
|---------|-----------|-----------|------|----------|
| Mandatory error handler | [N] Convention | [N] Optional | [N] Optional | **[Y] Compiler-enforced** |
| Checked results | [N] Return codes | [Y] Exceptions | [Y] Result\<T\> | **[Y] Result\<T\> + no-checky-no-val** |
| Integer overflow safety | [N] UB | [Y] Constraint_Error | [Y] Panic (or wrap) | **[Y] TBB sticky sentinel** |
| Value constraints | [N] assert() | [Y] Subtypes | [N] (crate-level) | **[Y] limit\<Rules\> + Z3** |
| Memory safety | [N] Manual | [Y] (no pointers) | [Y] Borrow checker | **[Y] Borrow ($$i/$$m) + wild** |
| Formal verification | External (Frama-C) | SPARK prover | External (Kani) | **Built-in Z3** |
| Unsafe escape hatch | Entire language | `pragma Import` | `unsafe {}` | **`wild` keyword** |

#### Key Differentiators

1. **failsafe is mandatory, not optional.** No other mainstream language refuses
   to compile a program that lacks a top-level error handler.

2. **TBB has no equivalent.** Rust panics on overflow (in debug) or wraps (in
   release). Ada raises Constraint_Error (which can be caught and ignored). TBB
   errors are **sticky and inescapable** — they propagate through all arithmetic
   forever.

3. **limit\<Rules\> with Z3 is first-class.** Ada subtypes are similar in spirit
   but lack SMT verification. Rust has no equivalent — constraint checking
   requires external crates and runtime assertions.

4. **Two-level error escalation (?! / !!!) is distinct from try/catch.** The
   operators are syntactically loud, making error paths visible in code review.

---

#### 15. Summary

Aria's safety model is built on a simple insight: **safety features that can be
forgotten will be forgotten.** Every feature described in this walkthrough is
either mandatory (failsafe), compiler-enforced (Result checking, borrow rules),
or structurally integrated (TBB, limit\<Rules\>, Z3).

#### The Safety Stack

```
Layer 7:  limit<Rules> + Z3 ─── Provable value constraints
Layer 6:  $$i / $$m borrows ─── Compile-time memory safety
Layer 5:  TBB types ─────────── Sticky error propagation
Layer 4:  ?! operator ──────── Local recovery with safe defaults
Layer 3:  Result<T> ─────────── Mandatory explicit error handling
Layer 2:  !!! operator ──────── Global emergency escalation
Layer 1:  failsafe() ────────── Mandatory last line of defense
```

Each layer catches failures that escape the layer above it. A corrupted sensor
reading might be caught by `?!` (Layer 4); if not, by TBB propagation (Layer 5);
if not, by `limit<Rules>` (Layer 7); if not, by `failsafe` (Layer 1).

#### Compiling the Examples

```bash
#### Build the compiler
cd aria/build && cmake .. && make -j$(nproc)

#### Compile and run individual examples
./ariac ../../aria-docs/safety-walkthrough/examples/01_failsafe_basics.aria
./01_failsafe_basics

#### Compile with Z3 verification
./ariac --verify ../../aria-docs/safety-walkthrough/examples/04_limit_rules.aria

#### Compile the complete pump controller
./ariac ../../aria-docs/safety-walkthrough/examples/07_infusion_pump.aria
./07_infusion_pump
```

#### Next Steps

- **For safety engineers:** Review the pump controller code and compare it to
  your current development practices. Consider which failure modes Aria prevents
  that your current toolchain does not.
- **For language researchers:** The formal comparison in Section 12 is intended
  as a starting point. We welcome analysis from the verification and safety
  communities.
- **For developers:** Clone the [Aria repository](https://github.com/alternative-intelligence-cp/aria),
  build the compiler, and try compiling and modifying the example files. Break
  things. See what the compiler catches.

---

*This document is part of the Aria documentation suite. For language specification,
see `specs/aria_specs.txt`. For getting started, see `GETTING_STARTED.md`.*

\newpage

# Part IV: Appendix — Reference


#### The Aria Philosophy - In the Creator's Words

#### The Origin Story

"One thing that really inspired me a lot was my month or two self-teaching x86-64 asm and the nasm macro system. Sure, anything complex is a pain in the ass, BUT, the simplicity always drew me in. The instructions for the most part say exactly what they do: **MOV, RET, JMP, JNE, PUSH, POP, CMP**... I actually loved that part."

#### What I Wanted

#### From Assembly
- **Clarity**: Instructions say what they do. No hunting through docs to understand if `<*>?::<turboBobDiesel>++j` adds integers or fries your GPU
- **Simplicity**: The core should be simple and understandable
- **NASM macro power**: Metaprogramming that actually works

#### From the Ecosystem
- **C interop**: The entire world runs on C when you get down to it
- **No forced tradeoffs**: I didn't want to choose between convenience/safety and power/performance
- **Per-operation control**: Move back and forth as needed, from web app to kernel mod
- **Rust's borrow checker without the year-long learning curve or frustration spiral**
- **JavaScript's functional system without the rest of the insanity**

#### For My Brain (ADHD-Friendly)
- **Guided decisions**: Compiler steers you toward good choices
- **Override when needed**: When guidance doesn't match reality for the current task, you can choose
- **Clear symbols**: One symbol, one meaning (except `*` for C interop necessity)
- **Better I/O organization**:
  - Binary and text don't need to be mixed
  - Debug info goes to its own log
  - Errors go to their own stream
  - I/O goes to its destination
  - **stdOut should only have standard output in text form**

#### Better Foundations
- **More native types**: The ones we have are piss poor for many use cases
- **TBB types**: Reuse -128 and other sentinel values to handle errors AND enable cmov optimizations
- **Balanced types**: Ternary logic isn't just academic - it's useful
- **Arbitrary precision**: int1024, int2048, int4096 for real-world crypto/quantum needs

#### But Here's The Truth

"When it came down to it, **every decision got made based on safety for the model and those around it** and supporting the model performance wise as well."

#### Safety First, Everything Else is Gravy

The fact that TBB's -128 sentinel enables:
- Better error handling (sticky errors that don't cause UB)
- Performance optimization (cmov instead of branches)

That's **gravy**. A happy accident from good foundational decisions.

The fact that other features mesh well together? **More gravy**.

#### The Core Priority

**Safety for the AI model and the developers using it.**

That's not negotiable. That's not a tradeoff. That's the north star every decision orbits around.

#### Design Principles That Emerged

#### 1. Symbols Say What They Do
Like ASM instructions. No mystery operators. No context-dependent meanings (except `*` for C interop).

```aria
MOV → move
RET → return  
JMP → jump

// In Aria:
Result<T>  // It's a result that wraps T
.is_error  // Boolean: is this an error?
.value     // The value (if not error)
pass()     // Pass (return success)
fail()     // Fail (return error)
```

#### 2. No False Dichotomies
You don't choose between:
- Safety OR performance
- Convenience OR control
- Beginners OR experts
- High-level OR low-level

You get all of them. You choose per operation.

#### 3. ADHD-Friendly Guardrails
- Compiler guides you (default Result<T> wrapping)
- But lets you override (`@nochecky` if you know what you're doing)
- Clear error messages with examples
- No hunting through docs to understand behavior

#### 4. Better I/O Separation
Different streams for different purposes:
- Standard output → text only
- Standard error → errors only  
- Debug log → debug info
- Binary output → binary data

No mixing. No confusion.

#### 5. Safety Through Design, Not Restriction

**Bad approach:** "You can't do X because it's unsafe"  
**Aria approach:** "You can do X, but the compiler will make sure you do it safely, or ask you to explicitly bypass (auditability)"

Examples:
- Result<T> enforcement: Must check, but simple to do
- Borrow checker: Safe references without fighting the compiler
- wild modifier: Unsafe operations are visible and auditable

#### The Gravy List

Things that turned out great but weren't the primary goal:

#### TBB Types with Sentinel Values
- **Goal:** Better error handling (sticky errors)
- **Gravy:** Enables cmov optimizations, avoiding branches
- **More gravy:** -128 causes bugs in two's complement, so banning it improves safety too

#### One Symbol One Meaning
- **Goal:** Clarity (don't hunt through docs)
- **Gravy:** Makes the language easier to teach
- **More gravy:** Reduces cognitive load for ADHD
- **Even more gravy:** Better tooling (unambiguous parsing)

#### Result<T> Enforcement
- **Goal:** Make sure AI model checks errors
- **Gravy:** Teaches developers good error handling
- **More gravy:** Eliminates whole classes of bugs
- **Even more gravy:** Self-documenting code (you see the checks)

#### Generic System
- **Goal:** Type safety with reusable code
- **Gravy:** Result<T>, HashMap<K,V>, etc. just work
- **More gravy:** Zero runtime cost (monomorphization)
- **Even more gravy:** Clear syntax (no trait soup)

#### Why This Matters for AI Safety

When an AI is writing code:
1. **Simpler is safer**: Less ambiguity = fewer misunderstandings
2. **Explicit is verifiable**: Can check the AI's work
3. **Guided is consistent**: AI learns patterns that are already safe
4. **Auditable bypasses**: When AI goes unsafe, it's visible

This isn't about protecting developers from themselves.  
**This is about creating a language where AI-generated code is naturally safer.**

#### The Bottom Line

> "I wanted to not have to go hunt through man pages or online tutorials every five minutes to figure out what operators do. I wanted to know what was happening."

**Aria: The language where things mean what they say.**

- `MOV` moves
- `Result<T>` is a result
- `.is_error` is a boolean
- `pass()` passes
- `wild` means "I'm going off the rails, audit this"

Safety through clarity.  
Performance through good foundations.  
Everything else?

**Gravy.**

---

*This philosophy shapes every design decision in Aria. When in doubt, ask: "Does this make the AI model safer? Does this make meaning clearer? Does this reduce hunting through docs?" If yes to any, consider it. If yes to all, it's probably the right choice.*

**Remember: The goal is safety. The gravy is that safety turns out to be powerful, performant, and pleasant to use.**

\newpage

#### Aria Compiler Architecture

**Purpose**: The Aria language compiler - transforms Aria source code into executable binaries via LLVM.

---

#### Philosophy

#### Design Principles

1. **Errors Are Values** - No exceptions. Result<T,E> everywhere.
2. **Explicit Over Implicit** - No hidden behavior, no magic.
3. **Memory Safety Without Runtime Cost** - Compile-time borrow checking.
4. **Performance Matters** - Zero-cost abstractions, direct LLVM integration.
5. **Balanced Ternary Computing** - Native support for -1/0/+1 computation via TBB types.

#### Non-Negotiables

- **Spec Compliance** - aria_ecosystem/specs/ documents are law
- **No Simplification** - Difficulty is not a reason to deviate
- **No Bloat** - Every feature must serve a clear purpose
- **Test Coverage** - Features without tests don't merge
- **LLVM Integration** - No custom backends, LLVM is the target

---

#### High-Level Architecture

```
Aria Source Code (.aria)
    ↓
[Preprocessor] - Handles #include, conditional compilation
    ↓
[Lexer] - Tokenization (src/frontend/lexer/)
    ↓
[Parser] - AST construction (src/frontend/parser/)
    ↓
[Semantic Analysis] - Type checking, borrow checking (src/frontend/sema/)
    ├→ Type Checker
    ├→ Borrow Checker
    ├→ Exhaustiveness Analyzer
    ├→ Symbol Table
    └→ Module Resolver
    ↓
[IR Generation] - LLVM IR emission (src/backend/ir/)
    ↓
[LLVM Optimization Passes]
    ↓
[Code Generation] - Platform-specific assembly
    ↓
[Linker] - Final executable
```

---

#### Component Breakdown

#### Frontend (src/frontend/)

#### Lexer (lexer/)
- **Purpose**: Convert source text into tokens
- **Key Files**:
  - `lexer.cpp` - Main tokenization logic
  - `token.cpp` - Token definitions
- **Responsibilities**:
  - UTF-8 handling
  - Keyword recognition
  - Number literal parsing (including TBB values)
  - String literal processing (including template literals)
  - Position tracking for error messages

#### Parser (parser/)
- **Purpose**: Build Abstract Syntax Tree (AST) from tokens
- **Key Files**:
  - `parser.cpp` - Main parsing logic (~3000 lines)
  - Statement parsing (pick, when, defer, etc.)
  - Expression parsing (precedence climbing)
- **Responsibilities**:
  - Syntax validation
  - AST node construction
  - Error recovery
  - Operator precedence
  - Pattern matching for pick statements

#### AST (ast/)
- **Purpose**: Data structures representing program structure
- **Key Files**:
  - `ast_node.h` - Base AST node class
  - `expr.h` - Expression nodes
  - `stmt.h` - Statement nodes
  - `type.h` - Type nodes
- **Node Types**:
  - Literals, identifiers, binary/unary ops
  - Function calls, array indexing, member access
  - Variable declarations, function declarations
  - Control flow (when, pick, defer)
  - Pick patterns (ranges, wildcards, ERR)

#### Semantic Analysis (sema/)
- **Purpose**: Validate program correctness, infer types, check ownership

##### Type Checker (type_checker.cpp)
- Type inference for expressions
- Type compatibility validation
- Must-use result enforcement (nodiscard)
- Pick statement exhaustiveness checking
- Generic type resolution

##### Borrow Checker (borrow_checker.cpp)
- Ownership tracking (move semantics)
- Lifetime validation
- Pinning reference analysis
- Appendage Theory enforcement
- Use-after-move detection

##### Exhaustiveness Analyzer (exhaustiveness.cpp)
- **NEW** - Pattern coverage analysis for pick statements
- Type domain calculation (TBB ranges, bool values, enum variants)
- Interval arithmetic for gap finding
- ERR sentinel enforcement for TBB types

##### Symbol Table (symbol_table.cpp)
- Variable scoping
- Function signatures
- Type definitions
- Module imports

##### Module Resolver (module_resolver.cpp)
- Import path resolution
- Dependency graph construction
- Circular dependency detection

---

#### Backend (src/backend/)

#### IR Generation (ir/)
- **Purpose**: Emit LLVM IR from validated AST
- **Key Files**:
  - `ir_generator.cpp` - Main IR emission
  - `codegen_expr.cpp` - Expression codegen
  - `codegen_stmt.cpp` - Statement codegen
  - `tbb_codegen.cpp` - TBB-specific optimizations
  - `ternary_codegen.cpp` - Ternary operator lowering

#### Code Generation Strategy
1. **Function-at-a-time** - Process each function independently
2. **SSA Form** - Static Single Assignment via LLVM
3. **Type Lowering** - Aria types → LLVM types
4. **Optimization** - Leverage LLVM's optimization passes

---

#### Runtime (src/runtime/)

#### Memory Management
- **GC** - Garbage collector for default allocations
- **Arena** - Bump allocator for temporary data
- **Pool** - Fixed-size object pools
- **Slab** - Variable-size slab allocator
- **Wild** - Manual memory (opt-out of GC)

#### Streams (streams/)
- Six-stream I/O implementation
- FD 3-5 initialization (stddbg, stddati, stddato)
- Platform-specific stream handling

#### Async Runtime (async/)
- M:N threading scheduler (planned)
- Work-stealing queue (planned)
- Coroutine state management (planned)

---

#### Dependencies

#### External
- **LLVM 20.1.2** - IR generation and optimization
- **CMake 3.20+** - Build system
- **C++17** - Compiler implementation language

#### Internal (Aria Ecosystem)
- **aria_lang_specs** - Language specification (source of truth)
- **aria_ecosystem** - Architecture documentation
- **AriaBuild** - Build system integration (planned)
- **AriaX** - Package management (planned)

---

#### Build System

#### CMakeLists.txt Structure
```cmake
#### Phase 1: Lexer
LEX_SOURCES = lexer.cpp, token.cpp

#### Phase 2: Parser  
PARSER_SOURCES = parser.cpp

#### Phase 3: Semantic Analysis
SEMA_SOURCES = type_checker.cpp, borrow_checker.cpp, exhaustiveness.cpp, ...

#### Phase 4: IR Generation
IR_SOURCES = ir_generator.cpp, codegen_expr.cpp, codegen_stmt.cpp, ...

#### Phase 5: Runtime
RUNTIME_SOURCES = gc.cpp, streams.cpp, ...

#### Main Executable
ariac = LEX + PARSER + SEMA + IR + RUNTIME + LLVM
```

#### Build Commands
```bash
#### Configure
cmake -B build -DCMAKE_BUILD_TYPE=Debug

#### Build compiler
cmake --build build --target ariac

#### Run tests
cmake --build build --target test

#### Install
cmake --build build --target install
```

---

#### Code Organization

```
aria/
├── include/              # Public headers
│   ├── frontend/
│   │   ├── ast/         # AST node definitions
│   │   ├── lexer/       # Lexer interface
│   │   ├── parser/      # Parser interface
│   │   └── sema/        # Semantic analysis interfaces
│   ├── backend/
│   │   └── ir/          # IR generation interface
│   └── runtime/         # Runtime library headers
│
├── src/                 # Implementation files
│   ├── frontend/
│   │   ├── lexer/
│   │   ├── parser/
│   │   ├── ast/
│   │   └── sema/
│   ├── backend/
│   │   └── ir/
│   ├── runtime/
│   │   ├── gc/
│   │   ├── streams/
│   │   └── async/
│   └── main.cpp         # Compiler entry point
│
├── tests/               # Test suite
│   ├── unit/           # Unit tests (C++)
│   ├── integration/    # Integration tests (Aria code)
│   ├── safety/         # Safety feature tests
│   └── runtime/        # Runtime library tests
│
├── aria_ecosystem/      # Documentation (submodule)
│   ├── specs/          # Language specifications
│   └── MASTER_ROADMAP.md
│
└── docs/               # Generated documentation
```

---

#### Data Flow Example

```aria
func:main = int8() {
    tbb8:result = 42;
    pick(result) {
        (0..127) { stddbg << "Valid\n"; },
        (ERR) { stddbg << "Error\n"; }
    }
    pass(0);
};
```

#### Compilation Pipeline

1. **Lexer Output** (tokens):
   ```
   FUNC, COLON, IDENT(main), EQUAL, IDENT(int8), LPAREN, RPAREN, LBRACE,
   IDENT(tbb8), COLON, IDENT(result), EQUAL, NUMBER(42), SEMICOLON,
   PICK, LPAREN, IDENT(result), RPAREN, LBRACE, ...
   ```

2. **Parser Output** (AST):
   ```
   FuncDeclStmt
     ├─ name: "main"
     ├─ returnType: int8
     ├─ body: BlockStmt
     │   ├─ VarDeclStmt (tbb8:result = 42)
     │   └─ PickStmt
     │       ├─ selector: IdentifierExpr("result")
     │       └─ cases:
     │           ├─ PickCase (0..127)
     │           └─ PickCase (ERR)
   ```

3. **Type Checker**:
   - Infers `result` type as `tbb8`
   - Validates pick selector is tbb8
   - Calls ExhaustivenessAnalyzer
   - Verifies both 0..127 and ERR cover full tbb8 domain

4. **Borrow Checker**:
   - Tracks `result` ownership
   - No moves detected (value semantics for tbb8)
   - No lifetime issues

5. **IR Generation** (LLVM IR):
   ```llvm
   define i8 @main() {
   entry:
     %result = alloca i8
     store i8 42, i8* %result
     %0 = load i8, i8* %result
     %1 = icmp eq i8 %0, -128  ; ERR check
     br i1 %1, label %err_case, label %range_case
   
   range_case:
     call void @stddbg_print(i8* "Valid\n")
     br label %exit
   
   err_case:
     call void @stddbg_print(i8* "Error\n")
     br label %exit
   
   exit:
     ret i8 0
   }
   ```

6. **LLVM Optimization**:
   - Dead code elimination
   - Constant propagation (`%result` is always 42)
   - Branch elimination (never takes ERR path)

7. **Code Generation**:
   - Platform-specific assembly
   - Link against Aria runtime
   - Produce executable binary

---

#### Key Algorithms

#### Exhaustiveness Checking
1. Calculate type domain (min/max values, symbols)
2. Extract coverage from pick cases (ranges, literals, ERR)
3. Sort and merge overlapping intervals
4. Find gaps using sweep algorithm
5. Check ERR coverage if TBB type
6. Generate helpful error message

#### Borrow Checking (Appendage Theory)
1. Assign scope depth to each variable
2. Track moves and pinning references
3. Validate lifetime rules (child cannot outlive parent)
4. Detect use-after-move
5. Verify drop order (children first, parents last)

#### Type Inference
1. Bottom-up traversal of expression tree
2. Literal types from token (42 → int64, 3.14 → double)
3. Operator result types from operands
4. Function return types from declaration
5. Generic instantiation via unification

---

#### Testing Strategy

#### Unit Tests (C++)
- Test individual components (lexer, parser, etc.)
- Mock dependencies
- Fast feedback (~1 second)

#### Integration Tests (Aria)
- Test actual Aria programs
- Verify end-to-end compilation
- Check runtime behavior

#### Safety Tests
- Test error detection (must-use, exhaustiveness, borrow checking)
- Verify helpful error messages
- Ensure bugs are caught at compile-time

---

#### Performance Characteristics

| Component | Complexity | Bottleneck |
|-----------|------------|------------|
| Lexer | O(n) | UTF-8 decoding |
| Parser | O(n) | Memory allocation for AST |
| Type Checker | O(n·m) | Generic instantiation |
| Borrow Checker | O(n·d) | Scope depth tracking |
| Exhaustiveness | O(k·log k) | Interval sorting (k = cases) |
| IR Generation | O(n) | LLVM API calls |

---

#### Future Directions

#### v0.1.0 (Released February 2026)
- Complete core language features
- Stable ABI
- Basic standard library
- Documentation generator

#### v0.2.0 (Released March 2026)
- Self-hosting compiler (5 modules, 220 tests)
- Improved error diagnostics with suggestions
- License change to Apache 2.0
- 7 critical codegen bug fixes

#### v0.2.1+ Goals
- Full async/await runtime
- Incremental compilation
- Debugger integration (LLDB)
- Cross-compilation support

---

#### For Contributors

**Read This First**:
1. [CONTRIBUTING.md](../../aria/CONTRIBUTING.md) - Rules and expectations
2. [TASKS.md](../../aria/TASKS.md) - Available work
3. [aria_ecosystem/specs/](../../aria_ecosystem/specs) - Language specifications

**Key Questions**:
- **Where do I start?** Read TASKS.md for beginner-friendly work.
- **How do I test?** `cmake --build build && ./build/ariac tests/your_test.aria`
- **What's the code style?** Match existing code. We don't bikeshed formatting.
- **Can I add a feature?** Only if it's in the specs. Want a new feature? Discuss in aria_ecosystem first.

**Common Pitfalls**:
- Not reading the spec before implementing
- Simplifying because "it's easier this way"
- Adding dependencies without discussion
- Not testing edge cases
- Ignoring error handling

---

*This document evolves with the project. Last major update: 2024-12-24*

\newpage

#### Aria Type System Design
**Date**: 2025-02-07  
**Status**: Design Phase - Ready for Implementation

#### Philosophy

**"A little functional, a little OOP, a lot of imperative"**

#### What We Have
- [DONE] **Functional**: Closures, lambdas, first-class functions
- [DONE] **OOP-lite**: UFCS method syntax (zero-cost)
- [DONE] **Imperative**: Core strength - explicit, predictable control flow

#### What We're Adding
**Type system** - Organized, encapsulated composable units with zero overhead

**NOT OOP** - No inheritance, no vtables, no runtime dispatch, no magic

#### Core Principle: Zero Context Switching

**Aria's Consistency Law**: `type:name = value;` for EVERYTHING

```aria
// Variables
int32:x = 42;

// Functions  
func:add = int32(int32:a, int32:b) { pass(a + b); };

// Lambdas (just functions without names)
int32(int32:x) { pass(x * 2); };  // Can execute immediately with ()

// Structs
struct:Point = { int32:x; int32:y; };

// Types (new!)
Type:Counter = { /* regular Aria syntax inside */ };
```

**No special cases. No syntax changes. Just organization.**

#### Type Declaration Syntax

#### Complete Example

```aria
Type:Counter = {
    // Constructor - just a regular Aria function
    func:create = Counter(int32:initial) {
        Counter:c = { 
            secret_ptr = allocate_storage(),
            count = initial 
        };
        pass(c);
    };
    
    // Destructor - just a regular Aria function  
    func:destroy = void(Counter:self) {
        free(self.secret_ptr);
    };
    
    // Methods - just regular Aria functions with self parameter
    func:increment = void(Counter:self) {
        self.count = self.count + 1;
    };
    
    func:get_value = int32(Counter:self) {
        pass(self.count);
    };
    
    // Private members - only accessible within Type methods
    struct:internal = {
        int32->:secret_ptr;
        int64:internal_counter;
    };
    
    // Public members - accessible everywhere
    struct:interface = {
        int32:count;
        int8:flags;
    };
    
    // Static/type-level members
    struct:type = {
        int32:MAX_VALUE = 1000;
        int32:MIN_VALUE = 0;
    };
};
```

#### Usage

```aria
// Instantiate - desugars to Counter_create(0)
Counter:myCounter = instance<Counter>(0);

// Method calls - UFCS desugars to Counter_increment(myCounter)
myCounter.increment();
int32:val = myCounter.get_value();

// Access public fields
myCounter.count = 5;

// Access static members
int32:max = Counter.MAX_VALUE;

// Cleanup - desugars to Counter_destroy(myCounter)
myCounter.destroy();
```

#### Semantic Transformation (Compile-Time Only!)

#### Input: Type Declaration

```aria
Type:Counter = {
    func:create = Counter(int32:initial) { /* ... */ };
    func:destroy = void(Counter:self) { /* ... */ };
    func:increment = void(Counter:self) { /* ... */ };
    
    struct:internal = { int32->:secret_ptr; };
    struct:interface = { int32:count; };
    struct:type = { int32:MAX_VALUE = 1000; };
};
```

#### Output: Desugared Code

```aria
// Step 1: Merge internal + interface into actual struct
struct:Counter = {
    // From internal (private, but no runtime enforcement yet)
    int32->:secret_ptr;
    
    // From interface (public)
    int32:count;
};

// Step 2: Prefix all functions with TypeName_
pub func:Counter_create = Counter(int32:initial) {
    // Original body unchanged
};

pub func:Counter_destroy = void(Counter:self) {
    // Original body unchanged
};

pub func:Counter_increment = void(Counter:self) {
    // Can access both internal and interface fields
    // (Access control is semantic analysis concern)
};

// Step 3: Static members become module-level constants
pub func:Counter_MAX_VALUE = int32() { pass(1000); };

// Step 4: instance<Counter>(0) desugars to Counter_create(0)
Counter:myCounter = Counter_create(0);

// Step 5: myCounter.increment() uses existing UFCS
// Already implemented in codegen_expr.cpp:2727-2800!
```

#### Implementation Phases

#### Phase 1: Parser (Syntax Recognition)
**File**: `src/frontend/parser/parser.cpp`

Add case for `Type:` declarations:
```cpp
if (current_token.type == TOKEN_TYPE && peek().type == TOKEN_COLON) {
    return parseTypeDecl();
}
```

**New AST Node**:
```cpp
class TypeDeclStmt : public ASTNode {
public:
    std::string typeName;
    
    // Required functions
    ASTNodePtr createFunc;   // Constructor
    ASTNodePtr destroyFunc;  // Destructor (optional)
    
    // Struct definitions
    ASTNodePtr internal;     // struct:internal
    ASTNodePtr interface;    // struct:interface  
    ASTNodePtr typeMembers;  // struct:type
    
    // Additional methods
    std::vector<ASTNodePtr> methods;
};
```

#### Phase 2: Semantic Analysis (Desugaring)
**File**: `src/frontend/sema/type_checker.cpp`

```cpp
void TypeChecker::checkTypeDecl(TypeDeclStmt* stmt) {
    // 1. Validate required components
    if (!stmt->createFunc) {
        addError("Type '" + stmt->typeName +
                 "' missing required 'create' function");
    }
    
    // 2. Merge internal + interface into combined struct
    auto combinedStruct = mergeStructs(stmt->internal, stmt->interface);
    
    // 3. Register combined struct as actual type
    typeSystem->registerStruct(stmt->typeName, combinedStruct);
    
    // 4. Validate all methods
    for (auto& method : stmt->methods) {
        validateMethodSignature(method, stmt->typeName);
    }
    
    // 5. Export public symbols (interface + methods)
    exportTypeSymbols(stmt);
}
```

#### Phase 3: Code Generation (No Changes Needed!)
**Existing code handles it all**:
- Struct codegen already works [DONE]
- Function codegen already works [DONE]  
- UFCS already implemented [DONE]
- Just need to emit desugared AST nodes

#### Phase 4: Access Control (Future Enhancement)
**Optional**: Enforce `internal` vs `interface` at semantic analysis

```cpp
void TypeChecker::validateMemberAccess(MemberAccessExpr* expr) {
    // Check if accessing internal member from outside Type methods
    if (isInternalMember(expr->member) && !isWithinTypeMethod(currentFunction)) {
        addError("Cannot access internal member '" + expr->member + 
                 "' outside of Type methods");
    }
}
```

#### Naming Strategy: OOP Deflection

#### Terminology Mapping

| Traditional OOP | Aria Type System | Rationale |
|----------------|------------------|-----------|
| `class` | `Type:` | Different word = different thing |
| `constructor` | `func:create` | It's just a function you write |
| `destructor` | `func:destroy` | Explicit, not magic |
| `new Class()` | `instance<Type>()` | Shows it's calling `create` |
| `private` | `internal` | Implementation internals |
| `public` | `interface` | What you expose |
| `static` | `type` | Type-level, not instance-level |
| `this` | `self` | More functional-style |
| `method` | UFCS function | It's a function with prefix! |

#### Defense Mechanisms

**Critic**: "This is just OOP with different names!"

**Response**: "Not at all. Look at the generated code - it's structs and functions. No inheritance, no vtables, no dynamic dispatch. Just organized composition with zero overhead. If you want OOP, use C++ or Java. Aria gives you the ergonomics without the cost."

**Critic**: "Why not use `class` keyword?"

**Response**: "Because it's not a class. It's a Type declaration that organizes related functions and data. The word matters - it sets expectations correctly."

**Critic**: "You're missing inheritance!"

**Response**: "We left that out on purpose :-) Use composition. Put one Type as a field in another. Studies show composition is more maintainable anyway. Favor composition over inheritance, right?"

**Critic**: "This is incomplete OOP!"

**Response**: "Well of course it isn't. Who told you it was? We explicitly avoided OOP complexity while keeping the useful organizational patterns. Zero-cost abstractions over convenient abstractions."

#### Example: Type-Based Atomic Counter

```aria
// stdlib/concurrent/atomic_counter.aria

Type:AtomicCounter = {
    // Constructor
    func:create = AtomicCounter(int32:initial) {
        wild int8->:mem = malloc(4);
        int32->:ptr = mem;
        <- ptr = initial;
        AtomicCounter:ac = { storage = ptr, last_value = initial };
        pass(ac);
    };
    
    // Destructor
    func:destroy = void(AtomicCounter:self) {
        free(self.storage);
    };
    
    // Methods
    func:load = int32(AtomicCounter:self, int32:order) {
        int32:val = __atomic_load_4(self.storage, order);
        pass(val);
    };
    
    func:store = void(AtomicCounter:self, int32:val, int32:order) {
        __atomic_store_4(self.storage, val, order);
    };
    
    func:increment = int32(AtomicCounter:self) {
        int32:old = __atomic_fetch_add_4(self.storage, 1, 5);
        pass(old);
    };
    
    // Private - implementation detail
    struct:internal = {
        int32->:storage;
    };
    
    // Public - visible state
    struct:interface = {
        int32:last_value;
    };
    
    // Static members
    struct:type = {
        int32:RELAXED = 0;
        int32:ACQUIRE = 2;
        int32:RELEASE = 3;
        int32:SEQ_CST = 5;
    };
};

// Usage
use "stdlib/concurrent/atomic_counter.aria";

func:main = int32() {
    AtomicCounter:counter = instance<AtomicCounter>(0);
    
    // Method syntax (UFCS)
    counter.store(42, AtomicCounter.SEQ_CST);
    int32:prev = counter.increment();
    int32:current = counter.load(AtomicCounter.RELAXED);
    
    // Access public field
    counter.last_value = current;
    
    // Cleanup
    counter.destroy();
    
    pass(current);
};
```

#### Composition Over Inheritance

```aria
// No inheritance needed - just compose!

Type:Logger = {
    func:create = Logger() { /* ... */ };
    func:log = void(Logger:self, int8->:msg) { /* ... */ };
    struct:interface = { int8->:name; };
};

Type:Database = {
    func:create = Database(Logger:logger) {
        Database:db = { logger = logger };
        pass(db);
    };
    
    func:query = void(Database:self, int8->:sql) {
        self.logger.log("Executing query");  // Delegate!
        // ... execute query
    };
    
    struct:interface = {
        Logger:logger;  // Composition!
    };
};

// Usage
func:main = void() {
    Logger:log = instance<Logger>();
    Database:db = instance<Database>(log);
    
    db.query("SELECT * FROM users");
    
    db.destroy();
    log.destroy();
};
```

#### Advantages Over Traditional OOP

#### Zero-Cost Abstractions [DONE]
```c
// Your Aria code:
counter.increment();

// Compiles to (LLVM IR):
call @AtomicCounter_increment(%struct.AtomicCounter %counter)

// NOT (OOP with vtable):
%vtable = load ptr, ptr %counter.vtable
%fn_ptr = getelementptr %vtable, i64 3
%fn = load ptr, ptr %fn_ptr
call void %fn(ptr %counter)  // Indirect call!
```

#### Predictable Memory Layout [DONE]
```aria
Type:Point = {
    struct:interface = { int32:x; int32:y; };
};

// Memory layout (exactly):
// [x: 4 bytes][y: 4 bytes]
// Total: 8 bytes (no vtable pointer!)
```

#### FFI Compatible [DONE]
```c
// C code can use Aria Types directly
struct Point {
    int32_t x;
    int32_t y;
};

void use_point(struct Point* p) {
    // Works perfectly!
}
```

#### Compile-Time Resolution [DONE]
- All method calls resolved at compile time
- No runtime type checking
- LLVM can inline everything
- Cache-friendly (no pointer chasing)

#### Current Implementation Status

#### [DONE] Already Working
- Struct definitions
- Function definitions
- UFCS method syntax
- Module exports
- Public visibility

#### [WIP] Need to Implement
- `Type:` parser
- Type AST node
- Desugaring logic
- `instance<T>()` syntax
- Static member access (`Type.MEMBER`)

#### [NOTE] Future Enhancements
- Access control enforcement (`internal` checking)
- Better error messages for Type usage
- Type-aware editor completion
- Automatic `destroy()` calls (RAII)

#### Next Steps

1. **Draft parser changes** - Add Type: recognition
2. **Create AST node** - TypeDeclStmt structure
3. **Implement desugaring** - Transform to structs + functions
4. **Test with atomic example** - Verify zero-cost
5. **Document patterns** - Stdlib development guide

---

**Motto**: "Composition over inheritance, explicitness over magic, zero-cost over convenience"

**Tagline**: "It's not OOP, it's organized imperative programming :-)"

\newpage

#### Aria Function Specification

#### Declaration Syntax
All function declarations require a trailing semicolon.
```
func:nameOfFunc = <ReturnType>(<ArgType>:argName){
    // Always returns Result<ReturnType> implicitly — main and failsafe are exceptions
    // Can return Result object, Result literal, or use pass/fail sugar functions
    // exit() is NOT valid from normal functions
};
```

#### Examples: pass/fail/return usage
```aria
func:thisFails = uint8(int32:arg) {
    tbb8:err = 1;
    // stuff happens
    fail(err); // sugar for: return Result{error:err, value:NIL, is_error:true};
};

func:thisPasses = uint8(int32:arg) {
    uint8:retVal = 33;
    // stuff happens
    pass(retVal); // sugar for: return Result{error:NIL, value:retVal, is_error:false};
};

func:longForm = flt32() {
    flt32:retVal = 1.2;
    // do stuff
    return Result{error:NIL, value:retVal, is_error:false};
};
```

#### main and failsafe — Both MANDATORY to compile an executable
#### Examples: exit() usage
```aria
func:main = int32(int32:argc, int8[]->:argv) {
    // pass()/fail() are INVALID in main
    // exit() call required; normal control flow semantics apply otherwise
    exit(0);
};

func:main = int32() {
    // zero-argument form also valid
    exit(0);
};

func:failsafe = int32(tbb32:err) {
    // Handle graceful shutdown
    // pass()/fail() are INVALID in failsafe
    // exit() call required; err code must be > 0
    exit(1);
};
```
Note: `int8[]->` is Aria's equivalent to C's `char**` for argv compatibility.

#### Exit Code Conventions
- Traditional conventions apply: 0 = no error, > 0 = error

#### FAILSAFE ERROR CODES (for `err` in `failsafe(tbb32:err)`)

The failsafe parameter is tbb32, giving range [-2147483647, +2147483647] with ERR sentinel.

#### Special Values
| Code | Meaning |
|------|---------|
| ERR  | Unknown/unclassifiable error (tbb32 sentinel, min value) |
| 0    | No error / testing / forced failsafe with no fault |

#### System Error Codes (negative range: -1 to -2147483647)
Reserved for runtime, OS, and hardware-level errors. These are set by the
Aria runtime or OS signal handlers — user code should not emit these.

| Code | Name | Meaning |
|------|------|---------|
| -1   | SYS_GENERAL          | General system error (unspecified) |
| -2   | SYS_OUT_OF_MEMORY    | Memory allocation failed |
| -3   | SYS_STACK_OVERFLOW   | Stack overflow detected |
| -4   | SYS_SEGFAULT         | Segmentation fault / access violation |
| -5   | SYS_ABORT            | Abort signal (SIGABRT) |
| -6   | SYS_FPE              | Floating-point exception (SIGFPE) |
| -7   | SYS_BUS_ERROR        | Bus error (SIGBUS) |
| -8   | SYS_ILLEGAL_INSN     | Illegal instruction (SIGILL) |
| -9   | SYS_PIPE             | Broken pipe (SIGPIPE) |
| -10  | SYS_ALARM            | Alarm/timer expired (SIGALRM) |
| -11  | SYS_TERMINATED       | Process terminated by signal (SIGTERM) |
| -12  | SYS_IO_ERROR         | I/O error (disk, device) |
| -13  | SYS_PERMISSION       | Permission denied |
| -14  | SYS_DEADLOCK         | Deadlock detected |
| -15  | SYS_THREAD_PANIC     | Thread panic / unrecoverable thread error |
| -16  | SYS_RESOURCE_LIMIT   | System resource limit reached (file descriptors, etc.) |
| -17 to -99   | (reserved)  | Reserved for future Aria runtime system codes |
| -100 to -999 | (reserved)  | Reserved for OS/platform-specific system codes |
| -1000 to -2147483647 | (unassigned) | Available for future system-level extensions |

#### User Error Codes (positive range: 1 to 2147483647)
For application-level errors triggered by `!!! code` (failsafe invocation).

| Code | Name | Meaning |
|------|------|---------|
| 1    | USR_GENERAL          | General application error (unspecified) |
| 2    | USR_INVALID_CONFIG   | Configuration invalid or missing |
| 3    | USR_INIT_FAILED      | Initialization/startup failed |
| 4    | USR_DATA_CORRUPT     | Data corruption detected |
| 5    | USR_DEPENDENCY       | Required dependency unavailable |
| 6    | USR_NETWORK          | Network error (connection lost, timeout) |
| 7    | USR_AUTH             | Authentication/authorization failure |
| 8    | USR_HARDWARE         | Hardware device error (sensor, GPU, etc.) |
| 9    | USR_TIMEOUT          | Critical operation timed out |
| 10   | USR_ASSERTION        | Assertion / invariant violation |
| 11 to 49    | (reserved)  | Reserved for future Aria standard user codes |
| 50 to 99    | (reserved)  | Reserved for framework/library standard codes |
| 100+        | (user-defined) | Application-specific error codes |

#### Usage Notes
- System codes are **negative**, user codes are **positive** — easy to distinguish in handlers
- Runtime sets system codes automatically on signals; user code calls `!!! code` with positive values
- ERR (tbb sentinel) means "we don't know what happened" — always handle it
- Code 0 should never reach failsafe in normal operation, but handle it gracefully
- Libraries should document their error codes in the 100+ range and avoid collisions


#### User Stack Builtins (v0.4.3)

Per-scope implicit LIFO scratch pad for type-safe intra-function value passing.
Separate from the hardware call stack and the `stack` memory allocation keyword.

#### `astack(capacity?)`
Initialize one implicit stack per function scope.
- `capacity`: optional `int64`, default 256 slots if omitted
- No handle returned — the stack is implicit to the current scope
- Must be called before `apush`/`apop`/`apeek` in the same scope

```aria
astack(64i64);   // 64-slot stack
astack();        // 256-slot default
```

#### `apush(value)`
Push a typed value onto the implicit stack.
- Accepts: int8, int16, int32, int64, flt32, flt64, bool, string, pointer
- Type tag stored automatically (no manual tag management)
- Fatal `exit(1)` with diagnostic to stderr on overflow
- No return value

```aria
apush(42i64);
apush(3.14f64);
apush(true);
```

#### `apop()`
Pop the top value. Destination type inferred from assignment context.
- Zero arguments
- Runtime type-tag checking: fatal `exit(1)` on mismatch or underflow
- Returns value directly (not wrapped in `Result<T>`)

```aria
int64:n = apop();    // pops as int64
flt64:f = apop();    // pops as flt64
bool:b = apop();     // pops as bool
```

#### `apeek()`
Non-destructive read of the top value.
- Same context-typed semantics as `apop()`
- Value remains on stack

```aria
int64:top = apeek();  // reads top without removing
```

#### Error Model
Fatal errors (not Result<T>):
- Stack overflow on push → stderr diagnostic + `exit(1)`
- Stack underflow on pop/peek → stderr diagnostic + `exit(1)`
- Type mismatch on pop/peek → stderr diagnostic + `exit(1)`

#### Auto-Cleanup
Stack is automatically destroyed on function return (explicit return or fallthrough).
No manual cleanup needed.
\newpage

#### Aria Reserved Words — Complete List

> Generated from `src/frontend/lexer/lexer.cpp` keyword table (151 keywords)
> v0.16.11 — Authoritative reference

All words listed below are reserved by the Aria compiler and **cannot be used as
variable names, function names, or identifiers** (except `stack` and `gc` which are
contextual — see notes).

---

#### Mandatory Functions

```
func        main        failsafe
```

#### Control Flow

```
if          else        while       for         loop        till
when        then        end         pick        fall        break
continue    return      pass        fail        exit
```

#### Result & Error Handling

```
ok          drop        raw         defaults    Result      ERR
NIL         NULL        unknown
```

#### Memory Management

```
wild        wildx       stack*      gc*         move        defer
```

> `*` `stack` and `gc` are **contextual** — valid as variable names outside allocation contexts.

#### Module System

```
use         mod         pub         extern      in          as          cfg
```

#### Type Declarations

```
struct      enum        trait       impl        derive      opaque
Type        Rules       func        macro
```

#### Type Modifiers

```
const       fixed       limit       dyn         obj         any
```

#### Async & Concurrency

```
async       await       catch
```

#### Memory Ordering

```
relaxed     acquire     release     acq_rel     seq_cst
```

#### Borrow & Safety

```
is          requires    ensures     invariant   result
```

#### Compile-Time

```
comptime    inline      noinline    prove       assert_static
```

#### Integer Types

```
int1        int2        int4        int8        int16       int32
int64       int128      int256      int512      int1024     int2048     int4096
uint1       uint2       uint4       uint8       uint16      uint32
uint64      uint128     uint256     uint512     uint1024    uint2048    uint4096
```

#### Other Numeric Types

```
tbb8        tbb16       tbb32       tbb64
flt32       flt64       flt128      flt256      flt512
tfp32       tfp64
fix256
frac8       frac16      frac32      frac64
```

#### Boolean & String

```
bool        true        false       string
```

#### Ternary & Nonary

```
trit        tryte       nit         nyte
```

#### Vector, Matrix & Tensor

```
vec2        vec3        vec9
matrix      tmatrix
tensor      ttensor
```

#### Collection & I/O Types

```
array       binary      buffer      stream      pipe        process
```

#### Debug & Special

```
debug       apop        apush       apeek       astack      acap
asize       afits       atype       ahash       ahset       ahget
ahcount     ahsize      ahfits      ahtype
```

---

#### Not Reserved (Common Misconceptions)

| Word | Status | Notes |
|------|--------|-------|
| `sync` | **Not a keyword** | Only `async`/`await` exist. Use atomics for sync patterns. |
| `atomic` | **Not a keyword** | `atomic<T>` is a generic type, not a reserved word. |
| `complex` | **Not a keyword** | `complex<T>` is a generic type, not a reserved word. |
| `simd` | **Not a keyword** | `simd<T,N>` is a generic type, not a reserved word. |
| `Handle` | **Not a keyword** | `Handle<T>` is a generic type, not a reserved word. |
| `void` | **Not a keyword** | Used in `extern` blocks only as a type annotation. |
| `free` | **Not a keyword** | Use `_release` or `_destroy` — `free` is shadowed. |

---

#### Total Count: 151 reserved keywords

\newpage

#### main / failsafe / exit() — Canonical Reference
#### This is the source of truth for how the two program endpoints work.

#### Design Intent

Every Aria executable has exactly two code-path endpoints:

  main       — normal program entry and exit
  failsafe   — abnormal/error exit (runtime errors, !!!, constraint violations)

Both terminate the process via `exit(code)`. Neither uses `pass()/fail()`.
pass/fail are the Result<T> return system for regular functions — they have
no business in program endpoints.


#### Signatures

```aria
// main: normal entry point
// - returns int32 (C runtime compatibility)
// - no parameters (for now)
// - MUST contain at least one exit() call
func:main = int32() {
    // ... program logic ...
    exit(0);
};

// failsafe: error endpoint
// - returns int32 (C runtime compatibility)
// - takes tbb32:err — balanced ternary error code
//   (uses tbb32 so the ERR signal can represent "unknown error" from runtime)
// - MUST contain at least one exit(val) call where val > 0
func:failsafe = int32(tbb32:err) {
    // ... graceful shutdown / cleanup ...
    exit(1);
};
```


#### Why tbb32 for failsafe's error code?

- The runtime may invoke failsafe for errors that don't have a numeric code
- tbb32 has three states: positive integers, negative integers, and ERR
- ERR represents "unknown/unclassifiable error" — a state int32 cannot express
- This keeps the safety system coherent all the way to the final exit call
- User-invoked `!!! code` can pass a known tbb32 value
- Runtime-invoked failsafe passes ERR when the error category is not known


#### exit() rules

- Only valid inside `main` and `failsafe`
- Takes exactly one int32 argument (the process exit code)
- In main: any value is valid (`exit(0)` for success, `exit(1)` for failure, etc.)
- In failsafe: value MUST be > 0 (failsafe is an error path — exit(0) is wrong)
- exit() terminates the process immediately — it is noreturn
- At least one exit() call must be reachable in main and failsafe


#### What NOT to use in main/failsafe

- `pass(value)` — this is for regular functions, creates Result<T>
- `fail(code)` — this is for regular functions, creates error Result
- `return` — Aria doesn't use this keyword


#### Invocation patterns

```aria
// Normal program exit
func:main = int32() {
    // ... do work ...
    exit(0);    // success
};

// Error exit from main
func:main = int32() {
    if (bad_condition) {
        exit(1);    // error
    }
    exit(0);
};

// Failsafe with cleanup
func:failsafe = int32(tbb32:err) {
    // Log error, close resources, etc.
    exit(1);
};

// Failsafe with error-code-dependent exit
func:failsafe = int32(tbb32:err) {
    int32:ret = 1;
    // ... handle different error categories ...
    ret = 32;
    exit(ret);
};

// Runtime triggers failsafe via !!! operator
func:some_function = int32() {
    if (critical_failure) {
        !!! 42;    // desugars to failsafe(42) → calls failsafe with tbb32 val
    }
    pass(0i32);
};
```


#### Compiler enforcement summary

1. main MUST exist (unless -c library mode)
2. main MUST return int32
3. main MUST take no parameters
4. main MUST contain at least one exit() call
5. failsafe MUST exist (unless -c library mode)
6. failsafe MUST return int32
7. failsafe MUST take exactly one parameter of type tbb32
8. failsafe MUST contain at least one exit() call with value > 0
9. exit() is only valid inside main and failsafe
10. exit() takes exactly one int32 argument

\newpage

#### User Stack Reference

**Feature**: Per-Scope Implicit User Stack  
**Version**: v0.4.3 (March 2026)  
**Status**: Implemented, 22/22 tests passing

---

#### Summary

The user stack is a compiler-managed typed LIFO scratch pad for intra-function value passing. It provides type-safe push/pop operations with automatic type tagging, context-typed pop, and scope-based auto-cleanup.

---

#### API

#### `astack(capacity?)`

| Property | Value |
|----------|-------|
| Arguments | 0 or 1 (`int64` capacity, default 256) |
| Returns | Nothing meaningful (implicit handle) |
| Side Effects | Allocates mmap-backed storage |
| Errors | None (allocation failure is fatal) |
| Scope | One per function scope |

#### `apush(value)`

| Property | Value |
|----------|-------|
| Arguments | 1 (any supported type) |
| Returns | Nothing |
| Side Effects | Stores value + type tag at stack top |
| Errors | Fatal `exit(1)` on overflow |
| Supported Types | int8, int16, int32, int64, flt32, flt64, bool, string, pointer |

#### `apop()`

| Property | Value |
|----------|-------|
| Arguments | 0 |
| Returns | Value of destination type (context-inferred) |
| Side Effects | Removes top element |
| Errors | Fatal `exit(1)` on underflow or type mismatch |
| Type Inference | From assignment target: `flt32:f = apop()` → pop as flt32 |

#### `apeek()`

| Property | Value |
|----------|-------|
| Arguments | 0 |
| Returns | Value of destination type (context-inferred) |
| Side Effects | None (non-destructive) |
| Errors | Fatal `exit(1)` on underflow or type mismatch |
| Type Inference | Same as `apop()` |

---

#### Type Tag Scheme

| Tag | Type    | Storage Method |
|-----|---------|----------------|
| 0   | int8    | Zero-extended to int64 |
| 1   | int16   | Zero-extended to int64 |
| 2   | int32   | Zero-extended to int64 |
| 3   | int64   | Direct |
| 4   | flt32   | Bitcast float → int32, zero-extended to int64 |
| 5   | flt64   | Bitcast double → int64 |
| 6   | bool    | Zero-extended to int64 |
| 7   | string  | Pointer stored as int64 |
| 8   | pointer | Pointer stored as int64 |

---

#### Compiler Implementation

#### Files Modified (from v0.4.2 → v0.4.3)

| File | Changes |
|------|---------|
| `include/runtime/ustack.h` | Push changed to void return, removed error codes, added default capacity |
| `src/runtime/collections/ustack.cpp` | Push returns void with fatal exit on overflow |
| `include/backend/ir/codegen_expr.h` | Added `ustack_pop_dest_type` context variable |
| `include/backend/ir/ir_generator.h` | Added `ustack_pop_dest_type` for propagation |
| `src/frontend/sema/type_checker.cpp` | New signatures: astack(0-1 args), apush(1), apop/apeek(0 args, UnknownType return) |
| `src/backend/ir/codegen_expr.cpp` | Per-scope implicit stack via hidden `__aria_ustack_handle` alloca; type-aware pop/peek with bitcast chain |
| `src/backend/ir/codegen_stmt.cpp` | VarDecl sets dest type before initializer; auto-cleanup on return |
| `src/backend/ir/ir_generator.cpp` | VarDecl sets dest type; CALL dispatch propagates to fresh ExprCodegen |

#### Key Implementation Detail

The `ustack_pop_dest_type` context variable is the critical mechanism. When the compiler processes `int64:x = apop()`, the VarDecl handler sets `ustack_pop_dest_type = "int64"` before generating the initializer expression. The `apop()` codegen reads this context to know which type tag to validate and how to bitcast the returned value.

This required a fix in v0.4.3: `IRGenerator` creates a **fresh** `ExprCodegen` for each CALL node, so the context must be set on the `IRGenerator` and propagated to `ExprCodegen` in the CALL dispatch path.

---

#### Error Diagnostics

All errors print to stderr with the format:
```
[ustack] <error description>
```

Then call `exit(1)`.

Example messages:
- `[ustack] overflow: stack full (256/256)`
- `[ustack] underflow: stack empty`
- `[ustack] type mismatch: expected tag 3 (int64), got tag 5 (flt64)`

---

#### Tests

| Test File | Count | Coverage |
|-----------|-------|----------|
| `tests/test_ustack.aria` | 14 | LIFO ordering, peek, multi-push, typed pop (int8/int32/int64/flt32/flt64), mixed types |
| `tests/test_ustack_strings.aria` | 8 | Scope independence across function calls, mixed int/float types |

---

#### Breaking Changes from v0.4.2

| v0.4.2 (Handle-Based) | v0.4.3 (Implicit) |
|------------------------|-------------------|
| `int64:h = astack(256i64)` | `astack(256i64)` or `astack()` |
| `apush(h, value)` | `apush(value)` |
| `apop(h)` | `apop()` — type from context |
| `apeek(h)` | `apeek()` — type from context |
| Push returns error code | Push is void (fatal on overflow) |

---

#### Design Rationale

The user stack fills a specific niche in Aria's "safe by default, opt-in to danger" philosophy:

1. **Why not just use variables?** Variables are faster but require knowing the shape at compile time. The user stack supports dynamic depth and mixed types.

2. **Why not Result<T>?** Stack misuse (overflow, underflow, type mismatch) is a logic error, not a runtime condition. Fatal errors prevent silent corruption.

3. **Why implicit handles?** Explicit handles add ceremony without benefit when there's only one stack per scope. The compiler manages the handle internally.

4. **Why context-typed pop?** Requiring `apop::<int64>()` turbofish syntax would work but is verbose. Context typing from the assignment target is natural and catches mismatches at runtime.

---

#### See Also

- [Spec](../specs/aria_specs.txt) — Line 6109: USER STACK section
- [Guide](../guide/advanced_features/user_stack.md) — Tutorial with examples
- [Memory Model](../guide/memory_model/user_stack.md) — Memory perspective

\newpage

#### Macro & Derive Authoring Guide

> v0.8.4 — Aria Compiler Documentation

#### Overview

Aria's macro system provides two mechanisms for code generation:

1. **Derive macros** — Auto-generate trait implementations for structs
2. **Attribute macros** — Modify or annotate declarations at compile time

#### Derive Macros

#### Built-in Derives

Apply derive attributes to structs to auto-generate trait implementations:

```aria
#[derive(Eq, Ord, Clone, ToString, Debug, Hash)]
struct:Point = {
    int32:x;
    int32:y;
};
```

#### Available Traits

| Trait | Method | Signature | Description |
|-------|--------|-----------|-------------|
| `ToString` | `to_string` | `string(Self:self)` | String representation |
| `Debug` | `debug` | `string(Self:self)` | Debug representation with types |
| `Eq` | `eq` | `bool(Self:self, Self:other)` | Field-by-field equality |
| `Ord` | `less_than` | `bool(Self:self, Self:other)` | Lexicographic less-than |
| `Hash` | `hash` | `uint64(Self:self)` | FNV-1a hash |
| `Clone` | `clone` | `Self(Self:self)` | Shallow copy |

#### How Derive Works

1. The type checker expands `#[derive(Trait)]` into a synthetic `impl:Trait:for:StructName`
2. The generated impl contains methods that operate field-by-field
3. Synthetic impls are injected into the AST before IR generation
4. IR codegen generates the mangled function (e.g., `Point_less_than`)

#### Using Derived Methods

Derived methods are called via UFCS (Uniform Function Call Syntax):

```aria
Point:a = Point{ x: 1, y: 2 };
Point:b = Point{ x: 3, y: 4 };

bool:is_equal = raw a.eq(b);
bool:is_less = raw a.less_than(b);
string:repr = raw a.to_string();
```

Note: Derived methods return `Result<T>`, so use `raw` to unwrap.

#### Attribute Macros

#### Available Attributes

| Attribute | Target | Description |
|-----------|--------|-------------|
| `#[inline]` | Functions | Hint to inline the function |
| `#[noinline]` | Functions | Prevent inlining |
| `#[align(N)]` | Structs/Vars | Set alignment in bytes |
| `#[derive(...)]` | Structs | Generate trait impls |
| `#[gpu_kernel]` | Functions | Mark as GPU kernel |
| `#[gpu_device]` | Functions | Mark as GPU device function |
| `#[comptime]` | Functions | Evaluate at compile time |

\newpage

#### SMT Solver Reference — Aria Compiler

Working document. Will become a best practices guide later.

---

#### Architecture Overview

Z3 is statically linked into `ariac`. Users never interact with it directly.
The compiler uses it in two modes:

1. **Verification mode** (`--verify`) — prove constraints, contracts, overflow safety
2. **Optimization mode** (`--smt-opt`) — prove properties → emit faster codegen

#### Pipeline Location

All Z3 work happens in **Phase 3.25** of compilation, between type checking
and borrow checking, before IR generation.

```
Parse → Type Check → Z3 (3.25) →
  Borrow Check → Result Elision (3.75) →
  IR Gen → LLVM → Link
```

#### Files

| File | LOC | Role |
|------|-----|------|
| `include/analysis/z3_verifier.h` | 208 | Class definition, verification API |
| `src/analysis/z3_verifier.cpp` | 1353 | Core verification logic, Z3 queries |
| `src/main.cpp` (Phase 3.25) | ~750 | Orchestration, AST walks, optimization discovery |
| `src/backend/ir/ir_generator.cpp` | ~20 | Fast-mode flag activation per function |
| `src/backend/ir/codegen_expr.cpp` | ~140 | Fast-path code emission for ustack/uhash |

---

#### Compiler Flags

| Flag | Implies | Effect |
|------|---------|--------|
| `--verify` | — | Enable Z3 verification (Rules consistency, limit checks) |
| `--verify-contracts` | `--verify` | Verify requires/ensures function contracts |
| `--verify-overflow` | `--verify` | Verify integer arithmetic overflow safety |
| `--verify-report` | `--verify` | Emit detailed proof results (proven/disproven/unknown) |
| `--smt-opt` | `--verify` | Enable SMT-guided optimizations (fast paths) |

Defined in `CompilerOptions` struct at main.cpp:132-137.
Parsed at main.cpp:328-341.

---

#### Verification Passes (current as of v0.4.7)

#### 1. Rules Consistency Check
- Verifies each `Rules` declaration is not self-contradictory
- Creates Z3 bitvector/real variable, asserts all conditions, checks SAT
- If UNSAT → error: rules can never be satisfied

#### 2. Literal Limit Verification
- Walks AST for `limit<RulesName>` on variable declarations
- For integer/float literal initializers, proves value satisfies all conditions
- Uses `verifyLimitInt()` / `verifyLimitFloat()` with concrete values

#### 3. Function Contract Verification (`--verify-contracts`)
- Walks functions with `requires`/`ensures` clauses
- Currently Phase 1 only: checks requires and ensures are individually satisfiable
- **NOT YET IMPLEMENTED: Phase 2** — proving ensures from requires + body

#### 4. Overflow Verification (`--verify-overflow`)
- Walks binary ops (+, -, *) looking for literal-literal arithmetic
- Uses Z3 bitvector overflow builtins (`Z3_mk_bvadd_no_overflow`, etc.)
- **Limitation:** Only checks when both operands are known constants

#### 5. SMT Optimization Discovery (`--smt-opt`)
- **User Stack (astack):** Collects type tags from all `apush()` calls in a function.
  If all tags are identical, Z3 proves homogeneity → function added to `ustack_opt_funcs`.
  Codegen emits `aria_ustack_push_fast` / `pop_fast` (no runtime type check).
- **User Hash (ahash):** Same pattern for `ahset()` value tags → `uhash_opt_funcs`.
  Codegen emits `aria_uhash_set_fast` / `get_fast`.

---

#### Z3 API Usage Patterns

#### Core Pattern (Negation-Based Proof)
```
1. Create Z3 sort (bitvector for ints, real for floats)
2. Translate Aria AST condition → Z3_ast
3. Assert NEGATION of the property
4. solver.check()
5. L_FALSE (UNSAT) → property PROVEN for all inputs
   L_TRUE (SAT) → DISPROVEN, extract counterexample from model
   L_UNKNOWN → timeout or too complex
```

#### Z3 Context
- Single `Z3_context` lives for entire compilation
- Per-query timeout: **5 seconds** (set in Z3Verifier constructor)
- Fresh solver created per query with push/pop scope

#### Type Mapping
| Aria Type | Z3 Sort | Tag |
|-----------|---------|-----|
| int8 | BitVec(8) | 0 |
| int16 | BitVec(16) | 1 |
| int32 | BitVec(32) | 2 |
| int64/int | BitVec(64) | 3 |
| flt32 | Real | 4 |
| flt64/flt | Real | 5 |
| bool | — | 6 |
| string | — | 7 |

---

#### Data Flow: Verifier → Codegen

```
main.cpp Phase 3.25:
  AST walk → collect push/set type tags per function
  Z3 proves homogeneity
  → std::set<std::string> ustack_opt_funcs
  → std::set<std::string> uhash_opt_funcs

main.cpp (post-verification):
  ir_gen.setUStackOptimizedFuncs(ustack_opt_funcs)
  ir_gen.setUHashOptimizedFuncs(uhash_opt_funcs)

ir_generator.cpp (FUNC_DECL entry):
  ustack_fast_mode = ustack_optimized_funcs.count(funcName) > 0
  uhash_fast_mode = uhash_optimized_funcs.count(funcName) > 0

codegen_expr.cpp (call emission):
  if (ustack_fast_mode) emit _fast variant (no type tag arg)
  else emit standard variant (with type tag arg)
```

---

#### Known Gaps & Limitations (v0.4.7)

#### Contract Verification
- Only Phase 1 (sanity check) — no body analysis
- Cannot prove ensures from requires + function body
- Needs symbolic execution or weakest precondition calculus

#### Overflow Detection
- Only works with literal operands (both sides must be constants)
- No range inference through assignments or control flow
- Would need abstract interpretation or interval analysis

#### Type Tag Inference
- Binary expressions: walks leftmost chain heuristically
- Complex expressions (function calls, casts) → tagged as unknown (-1)
- Unknown tags prevent optimization for entire function

#### Cross-Function Analysis
- Each function analyzed in isolation
- No interprocedural flows (callee results don't propagate to callers)
- No module boundary crossing

#### No Incremental Solving
- Fresh solver per query — no context reuse between queries
- Could cache Z3 context for related queries in same function

---

#### Bugs, Workarounds, & Notes

(Log observations here as we work through v0.5.x)

#### v0.5.0 Work Log

#### Result<T> Elision — IMPLEMENTED

**What:** Static analysis identifies functions that can never fail (no `fail`, no `sys`,
all callees also infallible). These return raw T instead of `{T, ptr, i8}`, eliminating
wrapping/unwrapping overhead at every call site.

**Activation:** `--smt-opt` flag (no Z3 needed — pure static analysis)

**Phase:** 3.75 (between Borrow Checker and IR Generation)

**Analysis algorithm:**
1. Collect all user functions with bodies (skip main, failsafe, generics, externs)
2. Walk each function's AST recursively, recording:
   - `has_fail`: contains any FAIL node
   - `has_sys_call`: calls sys/sys!!/sys!!!
   - `called_user_funcs`: set of user-function callees
3. Fixed-point iteration: a function is infallible iff `!has_fail && !has_sys_call &&
   all callees are infallible`. Propagate until stable.

**Files modified:**
- `ir_generator.h` — added `result_elide_funcs` set, `result_elide_mode` flag, `setResultElideFuncs()` setter
- `main.cpp` — Phase 3.75 analysis pass (~200 lines), passes set to IR gen
- `ir_generator.cpp` — Pre-pass forward decl checks elision, FUNC_DECL sets elide flag,
  Result wrapping decision uses elide flag, pass() returns raw T for elided funcs

**Key insight:** Call-site unwrap code needed NO modification — it detects Result by
structural shape `{T, ptr, i8}`. Elided functions return raw T, so the struct check
naturally short-circuits.

**Bug found/fixed:** Pre-pass forward declaration was creating functions with Result
return type before main codegen. Added elision check to pre-pass too.

**Also fixed:** Default return for non-struct types used `ConstantInt::get(type, 0)` which
only works for integers. Changed to `Constant::getNullValue(type)` which works for all types.

---

#### Future: Best Practices Guide Topics

- When to use `--verify` vs `--smt-opt` vs `--verify-all`
- Writing solver-friendly code (how to make your code provable)  
- Understanding counterexamples
- Performance impact of verification (build time budgets)
- Dealing with UNKNOWN results
- Contract authoring patterns
- Rules design for verifiability

\newpage

#### Garbage Collection Tuning Guide

> v0.8.4 — Aria Compiler Documentation

#### Overview

Aria uses a **hybrid generational garbage collector** with two regions:

- **Nursery (Young Generation)**: A copying (Cheney-style) semi-space for short-lived allocations.
- **Old Generation**: A mark-sweep region for long-lived objects promoted from the nursery.

#### GC Modes

#### Stop-the-World (STW)
All mutator threads are paused during collection. Default for minor (nursery) collections.

#### Concurrent
Major collections use **Snapshot-At-The-Beginning (SATB)** marking with incremental sweeping. Mutator threads continue running during the mark phase.

#### Safepoints

Safepoints are injected automatically at:
- Loop back-edges (every iteration boundary)
- Function call sites

At safepoints, the runtime checks if a GC is pending and yields if needed.

#### Shadow Stack Root Scanning

The GC uses a **shadow stack** to track root references. Every function that allocates GC-managed objects pushes roots onto the shadow stack and pops them on exit.

#### Coroutine Integration (v0.8.4)

Suspended coroutine frames contain GC roots that must be scanned during collection. The `GCCoroAllocator::scan_frames()` method iterates all suspended frames and marks their roots during both minor and major GC cycles.

#### JIT Root Tracking (v0.8.4)

JIT-compiled code can register GC roots via:
- `aria_gc_register_jit_root(void** root_addr)` — Register a root location
- `aria_gc_unregister_jit_root(void** root_addr)` — Unregister when no longer needed

JIT roots are scanned during all GC phases — minor evacuation, STW major marking, and concurrent major marking.

#### Object Pinning (Wild Interop)

Objects passed to `wild` (unsafe FFI) blocks are **pinned** in place so external C code retains valid pointers. Pinned objects are:
- Skipped during nursery evacuation (not moved)
- Scanned in-place during old-gen marking

#### Write Barriers

A **card table** tracks cross-generational references. When an old-gen object is updated to point to a nursery object, the corresponding card is dirtied. During minor GC, dirty cards are scanned for additional roots.

#### Tuning Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Nursery size | 4 MB | Semi-space size for young generation |
| Promotion threshold | 2 | Survive N minor GCs before promotion |
| Major GC trigger | 75% | Old-gen occupancy threshold |
| Concurrent mode | Auto | STW for small heaps, concurrent for large |

\newpage

#### RFC: Traits & Borrow Semantics for Aria

**Status:** Design Proposal (v0.2.4)  
**Implementation Target:** v0.2.5 or v0.2.6  
**Author:** AI-assisted design  

---

#### Motivation

Two gaps in Aria's type system limit what real programs can express:

1. **No polymorphic dispatch.** You can't write a function that works on "anything with an `.encode()` method." Database backends, serialization formats, and generic algorithms all need this.

2. **No way to share handles.** Move-by-default consumes every variable on first use. Passing a DB handle to two functions requires the `last_db()` re-acquisition hack. Borrows exist in the checker but aren't user-facing.

Both are needed for Nikola: DB abstraction across SQLite/PostgreSQL/Redis, GPU handle passing, iterator patterns, and generic collections.

---

#### Part 1: Traits

#### Design Goals

- **Monomorphization by default** — zero runtime cost, the compiler stamps out a copy per concrete type
- **Vtable dispatch opt-in** — `dyn Trait` for cases where you truly need runtime polymorphism
- **No trait soup** — keep the common case simple, don't require 5 bounds on every function
- **Compose with UFCS** — `obj.method()` already desugars to `Type_method(obj)`, traits extend this naturally
- **Compose with Type declarations** — `Type:Person` already groups struct + methods, trait conformance plugs in

#### Syntax

#### Defining a trait

```aria
trait:Encodable = {
    func:encode = string(Self:self);
    func:size_hint = int32(Self:self);
};
```

- `Self` refers to the implementing type (resolved at compile time for monomorphization)
- Trait body contains function signatures — no default implementations in v1 (KISS)
- Traits can require other traits: `trait:Ordered = Equatable & { func:compare = int32(Self:a, Self:b); };`

#### Implementing a trait

```aria
impl Encodable for Vec3 {
    func:encode = string(Vec3:self) {
        pass("(" + int32_toString(self.x) + "," + int32_toString(self.y) + ")");
    };
    func:size_hint = int32(Vec3:self) {
        pass(20i32);
    };
};
```

- `impl Trait for Type { ... }` — explicit, clear, hard to confuse
- Each function must match the trait signature exactly (with `Self` replaced by the concrete type)
- An impl block can live in any file (not just where the type or trait is defined)

#### Integrating with Type declarations

```aria
Type:Person = {
    struct:internal = { int32:id; };
    struct:interface = { string:name; int32:age; };

    // Type methods (UFCS — already works)
    func:create = Person(string:name, int32:age) { ... };
    func:greet = NIL(Person:self) { print(self.name); pass(NIL); };

    // Trait conformance declared inline
    conforms Encodable;
    conforms Equatable;
};

// Implementations can be separate
impl Encodable for Person { ... };
```

- `conforms Trait;` inside a Type declaration is a compile-time assertion — the compiler checks that an `impl` block exists
- This keeps the Type declaration as a readable summary of what a type can do

#### Using trait bounds

```aria
func<T: Encodable>:write_all = NIL(T[]:items) {
    int32:i = 0i32;
    while (i < items.length) {
        print(items[i].encode());
        i = i + 1i32;
    }
    pass(NIL);
};
```

- Generic param constraints: `<T: Encodable>`, `<T: Encodable & Ordered>` (parser already handles this)
- The compiler monomorphizes: `write_all::<Vec3>` generates a concrete function with `Vec3_encode` calls
- UFCS glue: `items[i].encode()` → resolved to `Vec3_encode(items[i])` after monomorphization

#### Dynamic dispatch (opt-in)

```aria
func:write_any = NIL(dyn Encodable:item) {
    print(item.encode());
    pass(NIL);
};
```

- `dyn Trait` is a fat pointer (data ptr + vtable ptr)
- Runtime cost — only use when you need heterogeneous collections or plugin-style extensibility
- The compiler builds a vtable struct: `{ ptr encode_fn; ptr size_hint_fn; }`

#### Standard Traits (Proposed)

| Trait | Methods | Notes |
|-------|---------|-------|
| `Equatable` | `equals(Self, Self) -> bool` | `==` operator desugars to this |
| `Ordered` | `compare(Self, Self) -> int32` | `<`, `>`, `<=`, `>=` desugar |
| `Hashable` | `hash(Self) -> int64` | Required for HashMap keys |
| `Copyable` | `copy(Self) -> Self` | Opt-in value semantics (most types are move-only) |
| `Displayable` | `to_string(Self) -> string` | `print()` and string interpolation |
| `Addable` | `add(Self, Self) -> Self` | `+` operator, plus `Subtractable`, `Multipliable`, `Dividable` |
| `Defaultable` | `default() -> Self` | Zero-value construction |
| `Droppable` | `finalize(Self) -> NIL` | Custom cleanup (called by `drop()` or scope exit) |

#### Implementation Strategy

1. **Parser** — `trait:Name = { ... };` and `impl Trait for Type { ... }` node types (NT_TRAIT_DECL already reserved)
2. **Type checker** — resolve trait bounds on generic params, verify impl blocks match trait signatures
3. **IR codegen** — monomorphization: stamp out concrete function for each `<T>` instantiation with known type
4. **Vtable codegen** (later) — for `dyn Trait`: generate vtable struct, emit indirect calls

---

#### Part 2: Borrow Semantics

#### Design Goals

- **Make the existing borrow checker user-facing** — the infrastructure is already there (`$` operator, AccessPath, loan tracking)
- **Rust's safety without Rust's learning curve** — Aria should guide, not punish
- **Explicit borrows** — no implicit reference creation, you always see `$`
- **Compatible with move-by-default** — borrows are a controlled exception to the consume-on-use rule

#### Syntax

#### Immutable borrow

```aria
int32:x = 42i32;
int32$:ref = $x;          // borrow x (immutable)
print(ref);                // use the borrow — x is still alive
print(x);                  // x is still usable — it wasn't moved
```

- `$x` creates an immutable borrow of `x`
- `int32$` is the borrow type (already in the type system as `SafeRefType`)
- Multiple immutable borrows allowed simultaneously

#### Mutable borrow

```aria
int32:x = 42i32;
int32$:ref = $mut x;       // mutable borrow
ref = 99i32;                // mutate through the borrow
// x is now 99
// cannot use x while ref is alive (enforced by borrow checker)
```

- `$mut x` creates a mutable borrow
- Only one mutable borrow at a time (borrow checker already enforces this)
- Original variable is frozen while mutably borrowed

#### Passing borrows to functions

```aria
func:print_value = NIL(int32$:val) {
    print(val);
    pass(NIL);
};

func:increment = NIL(int32$ mut:val) {
    val = val + 1i32;
    pass(NIL);
};

func:main = int32() {
    int32:x = 10i32;
    print_value($x);       // pass immutable borrow — x survives
    increment($mut x);     // pass mutable borrow — x is modified
    print(x);              // prints 11
    pass(0i32);
};
```

- `int32$` in a parameter = accepts an immutable borrow
- `int32$ mut` in a parameter = accepts a mutable borrow
- The caller uses `$x` or `$mut x` at the call site — always explicit

#### Structs and handles

```aria
// The Nikola use case: pass a DB handle to multiple functions
int32:db = aria_sqlite_open(":memory:");
setup_tables($mut db);      // borrows db, doesn't consume it
insert_data($mut db);       // borrows db again — previous borrow ended
query_data($db);            // immutable borrow — read-only access
aria_sqlite_close(db);      // final use — db is moved/consumed here
```

- No more `last_db()` hack
- The borrow checker ensures `db` isn't used after `close()`
- Mutable borrows are sequential (one at a time), immutable borrows can overlap

#### Split borrows (already implemented)

```aria
struct:Pair = { int32:a; int32:b; };
Pair:p = Pair{a: 1i32, b: 2i32};
int32$:ref_a = $p.a;       // borrow field a
int32$:ref_b = $p.b;       // borrow field b — disjoint, allowed
// Both refs alive simultaneously — the borrow checker tracks access paths
```

#### Lifetime Rules

Aria uses **scope-based lifetimes** (no explicit lifetime annotations like Rust's `'a`):

1. A borrow cannot outlive the scope of the borrowed value
2. The borrow checker uses Appendage Theory: `Depth(Borrower) ≤ Depth(Owner)`
3. Returned borrows are forbidden in v1 — return owned values instead
4. Borrows in structs are forbidden in v1 — use owned values or pointers

These restrictions eliminate the need for lifetime parameters while covering 90% of use cases. The remaining 10% (self-referential structs, borrowed iterators) can use `wild` pointers with explicit unsafe opt-in.

#### Two-Phase Borrows (already implemented)

```aria
Vector<int32>:vec = Vector{};
vec.push(vec.len());    // Works! push($mut vec, vec.len($vec))
                        // Phase 1: reserve mutable borrow for push
                        // Phase 2: activate after len() completes
```

#### Pin Semantics (`#` operator — existing)

```aria
wild int8[1024]:buffer = malloc(1024);
int8#:pinned = #buffer;    // pin: GC won't move this memory
ffi_read(pinned, 1024);    // safe to pass to C — address is stable
// pinned released when scope exits
```

- `#` pins a value in memory (prevents GC relocation)
- Critical for FFI: C functions expect stable pointers
- Borrow checker tracks pin lifetime

---

#### Part 3: Interaction Between Traits and Borrows

#### Trait methods and self borrowing

```aria
trait:Printable = {
    func:display = NIL(Self$:self);      // takes immutable borrow of self
};

trait:Resettable = {
    func:reset = NIL(Self$ mut:self);    // takes mutable borrow of self
};
```

- Trait methods declare whether they borrow self immutably (`Self$`) or mutably (`Self$ mut`)
- This lets the compiler enforce at the call site: `obj.display()` auto-borrows immutably, `obj.reset()` auto-borrows mutably
- If a method needs to consume self (like `drop`), it takes `Self:self` (owned)

#### Generic bounds with borrow-friendliness

```aria
func<T: Printable>:print_twice = NIL(T$:item) {
    item.display();     // first call borrows the borrow (re-borrow)
    item.display();     // second call — fine, immutable borrow
    pass(NIL);
};
```

- Passing `T$` (borrowed) means the function doesn't consume the value
- The caller retains ownership: `print_twice($my_value);`

---

#### Summary of Syntax Additions

| Feature | Syntax | Example |
|---------|--------|---------|
| Trait definition | `trait:Name = { ... };` | `trait:Encodable = { func:encode = string(Self:self); };` |
| Trait implementation | `impl Trait for Type { ... };` | `impl Encodable for Vec3 { ... };` |
| Trait conformance | `conforms Trait;` | Inside `Type:` declaration |
| Trait bound | `<T: Trait>` | `func<T: Ordered>:sort = ...` |
| Multiple bounds | `<T: A & B>` | `func<T: Hashable & Equatable>:lookup = ...` |
| Dynamic dispatch | `dyn Trait` | `func:f = NIL(dyn Encodable:x)` |
| Immutable borrow | `$x` / `Type$` | `int32$:ref = $x;` |
| Mutable borrow | `$mut x` / `Type$ mut` | `int32$ mut:ref = $mut x;` |
| Pin | `#x` / `Type#` | `int8#:pinned = #buffer;` |

---

#### Open Questions

1. **Default trait methods?** Allowing `func:encode = string(Self:self) { ... };` with a body inside a trait definition. Useful but adds complexity. Propose: defer to v2.
2. **Associated types?** `trait:Iterator = { type Item; func:next = Item$(...); };` — needed for iterators but complex. Propose: defer to v2.
3. **Trait objects in collections?** `dyn Encodable[]:items` for heterogeneous arrays. Needs vtable + fat pointer arrays. Propose: defer to v2.
4. **Operator overloading via traits?** `+` → `Addable.add()`. Propose: yes, include in v1 (natural, expected).
5. **Auto-borrow at call sites?** Should `obj.method()` auto-create `$obj` if method takes `Self$`? Propose: yes, for ergonomics (Rust does this).

---

#### Implementation Phases

#### Phase 1 (v0.2.5): Core Traits + User-Facing Borrows
- `trait:` declaration parsing and AST
- `impl Trait for Type` parsing and AST
- Trait bound resolution in type checker
- Monomorphization in IR codegen
- User-facing `$x` / `$mut x` borrow syntax
- Standard traits: `Equatable`, `Displayable`, `Copyable`

#### Phase 2 (v0.2.6): Operator Traits + Dynamic Dispatch
- `Addable`, `Ordered`, `Hashable` with operator desugaring
- `dyn Trait` fat pointer support
- `conforms` assertion in Type declarations
- Generic collections (`Vector<T>`, `HashMap<K: Hashable, V>`)

#### Phase 3 (v0.2.7+): Advanced
- Default trait methods
- Associated types
- Trait objects in collections
- Derived traits (`#[derive(Equatable, Hashable)]`)

\newpage

#### Aria WebAssembly Compilation Guide

#### Overview

Aria v0.2.13 adds WebAssembly (WASM) as a compilation target. Compile Aria programs to `.wasm` files that run in WASI-compatible runtimes like `wasmtime`, `wasmer`, or `wasm3`.

#### Prerequisites

- **wasi-libc**: WASI sysroot with libc for wasm32
  ```bash
  sudo apt install wasi-libc
  ```
- **wasm-ld**: WebAssembly linker (part of LLVM/LLD)
  ```bash
  sudo apt install lld
  ```
- **wasmtime** (or wasmer): To run the compiled `.wasm` files
  ```bash
  curl https://wasmtime.dev/install.sh -sSf | bash
  ```

#### Usage

#### Basic Compilation

```bash
ariac program.aria --emit-wasm -o program.wasm
```

#### With Explicit Target

```bash
ariac program.aria --emit-wasm --target=wasm32-wasi -o program.wasm
```

#### Running

```bash
wasmtime program.wasm
```

#### Supported Features

| Feature | WASM Support | Notes |
|---|---|---|
| Integer arithmetic (int32, int64) | [DONE] Full | |
| Float arithmetic (flt32, flt64) | [DONE] Full | |
| Strings | [DONE] Full | concat, compare, trim, case, search, pad |
| Control flow (if/else, while) | [DONE] Full | |
| Functions & recursion | [DONE] Full | |
| Structs | [DONE] Full | |
| Arrays | [DONE] Full | |
| Pattern matching | [DONE] Full | |
| Error handling (Result types) | [DONE] Full | |
| File I/O | [WARN] WASI only | Needs WASI runtime with filesystem access |
| stdin/stdout | [DONE] Full | Via WASI fd_read/fd_write |
| Math functions | [DONE] Full | Software implementations |
| Generics | [DONE] Full | |

#### Unsupported Features (WASM Target)

These features are **not available** when compiling to WASM. The compiler will emit warnings if your code uses them:

| Feature | Reason |
|---|---|
| Async/await | No io_uring in WASM |
| Threading & thread pools | WASM is single-threaded (for now) |
| Mutex/atomics | No threading support |
| Process spawning (fork/exec) | WASM sandbox restriction |
| Signals | WASM has no signal model |
| mmap/mprotect | WASM uses linear memory |
| Raw syscalls | WASM sandbox restriction |
| FFI to native .so/.dll | WASM has its own import model |

The compiler's compatibility checker scans your code and warns about unsupported features before linking.

#### Architecture

```
                    ┌─────────────┐
 program.aria  ──>  │  Aria Parser │
                    └──────┬──────┘
                           │ AST
                    ┌──────▼──────┐
                    │  IR Generator│
                    └──────┬──────┘
                           │ LLVM IR
                    ┌──────▼──────┐
                    │ WASM Codegen │  (target: wasm32-wasi)
                    └──────┬──────┘
                           │ .o (WASM object)
              ┌────────────▼────────────┐
              │        wasm-ld          │
              │  + libaria_runtime_wasm │
              │  + wasi-libc            │
              └────────────┬────────────┘
                           │
                    program.wasm
```

The WASM backend:
1. Sets target triple to `wasm32-wasi`
2. Renames `main` → `__main_argc_argv` (WASI convention)
3. Emits a WASM object file via LLVM's WebAssembly backend
4. Links with `wasm-ld` against `libaria_runtime_wasm.a` and WASI libc

#### WASM Runtime

`libaria_runtime_wasm.a` is a stripped-down, WASI-compatible replacement for the native `libaria_runtime`. It provides:

- **String operations**: All `_simple` string functions (concat, trim, case, search, pad, substring, etc.)
- **I/O**: Print, file read/write (via WASI), stdin
- **Math**: All math functions (pow, sqrt, trig, abs, min/max)
- **Memory**: Arena, pool, slab allocators (simplified)
- **Maps**: Hash map with FNV-1a hashing
- **Compiler builtins**: `__multi3` (128-bit multiply for int64 arithmetic)

#### Examples

#### Hello World

```aria
func:main = int32() {
    out("Hello from WebAssembly!");
    return 0i32;
};

func:failsafe = void(int32:err_code) {};
```

```bash
ariac hello.aria --emit-wasm -o hello.wasm
wasmtime hello.wasm
#### Output: Hello from WebAssembly!
```

#### String Operations

```aria
func:main = int32() {
    str:greeting = "Hello, ";
    str:name = "World";
    str:message = greeting + name + "!";
    out(message);
    return 0i32;
};

func:failsafe = void(int32:err_code) {};
```

#### Integer Math & Control Flow

```aria
func:factorial = int64(int64:n) {
    if n <= 1i64 {
        return 1i64;
    };
    int64:sub = n - 1i64;
    int64:rec = factorial(sub);
    int64:result = n * rec;
    return result;
};

func:main = int32() {
    int64:fact5 = factorial(5i64);
    // fact5 == 120
    return 0i32;
};

func:failsafe = void(int32:err_code) {};
```

#### Binary Size

Typical WASM binary sizes:
- Hello world: ~15-20 KB
- String operations: ~25-35 KB
- Math-heavy programs: ~20-30 KB

These sizes include the WASM runtime and WASI libc stubs.

#### Troubleshooting

#### "undefined symbol" errors
Your program may use a feature not available in the WASM runtime. Check the unsupported features table above.

#### "function signature mismatch"
This typically indicates an ABI mismatch between your code and the WASM runtime. Ensure you're using the latest `libaria_runtime_wasm.a` built alongside the compiler.

#### wasmtime permission errors for file I/O
WASI sandboxes filesystem access. Grant directory access:
```bash
wasmtime --dir=. program.wasm
```

\newpage

#### Cross-Language Bindings

Aria can interoperate with C, Python, and any language that supports C ABI calling conventions.

---

#### Overview

Aria's compilation pipeline (Aria → LLVM IR → native code) produces symbols with standard C ABI linkage. This means Aria functions are callable from any language that can load shared libraries and call C functions.

**Three binding directions:**

| Direction | Mechanism | Status |
|-----------|-----------|--------|
| C → Aria | `extern` FFI blocks | Stable (v0.1.0+) |
| Aria → C | `--shared` flag produces `.so` | New in v0.2.6 |
| Aria → Python | `ctypes.CDLL()` on `.so` | New in v0.2.6 |

---

#### Aria → C (Exporting Aria Functions)

Compile Aria source to a shared library that C programs can link against.

#### Step 1: Write Aria Library

```aria
// mathlib.aria
func:add = int32(int32:a, int32:b) {
    pass(a + b);
};

func:multiply = int32(int32:x, int32:y) {
    pass(x * y);
};

func:square = int32(int32:n) {
    pass(n * n);
};
```

#### Step 2: Compile to Shared Library

```bash
ariac mathlib.aria --shared -o libmathlib.so
```

The `--shared` flag:
- Skips the `failsafe` requirement (library mode)
- Compiles with Position-Independent Code (PIC)
- Links as a shared library (.so)

#### Step 3: Use from C

```c
#include <stdio.h>

// Declare the Aria functions
extern int add(int a, int b);
extern int multiply(int x, int y);
extern int square(int n);

int main() {
    printf("add(3, 4) = %d\n", add(3, 4));
    printf("multiply(5, 6) = %d\n", multiply(5, 6));
    printf("square(9) = %d\n", square(9));
    return 0;
}
```

```bash
gcc main.c -L. -lmathlib -Wl,-rpath,. -o main
./main
```

#### Type Mapping (Aria → C)

| Aria Type | C Type | Notes |
|-----------|--------|-------|
| `int8` | `int8_t` / `char` | |
| `int16` | `int16_t` / `short` | |
| `int32` | `int32_t` / `int` | |
| `int64` | `int64_t` / `long long` | |
| `float32` | `float` | |
| `float64` | `double` | |
| `bool` | `int` | 0 = false, 1 = true |
| `string` | `char*` | Null-terminated C string |

---

#### C → Aria (Consuming C Libraries)

Use `extern` blocks to declare C functions, then call them from Aria.

#### Example: Using a C Math Library

```aria
extern "m" {
    func:sqrt = float64(float64:x);
    func:pow = float64(float64:base, float64:exp);
    func:sin = float64(float64:x);
    func:cos = float64(float64:x);
};

func:failsafe = NIL(int32:code) { pass(NIL); };

func:main = NIL() {
    var:result = sqrt(144.0);
    emit(result);
};
```

```bash
ariac main.aria -o main -lm
```

#### Using Custom C Shared Libraries

```aria
extern "mylib" {
    func:do_work = int32(int32:input);
    func:get_version = int32();
};
```

```bash
ariac main.aria -o main -L./libs -lmylib
```

The `extern "name"` string corresponds to the library name: `libname.so`.

---

#### Aria → Python (via ctypes)

Python can call Aria shared libraries using the built-in `ctypes` module.

#### Step 1: Compile Aria to Shared Library

```bash
ariac mathlib.aria --shared -o libmathlib.so
```

#### Step 2: Call from Python

```python
import ctypes

#### Load the Aria shared library
lib = ctypes.CDLL('./libmathlib.so')

#### Declare function signatures
lib.add.restype = ctypes.c_int
lib.add.argtypes = [ctypes.c_int, ctypes.c_int]

lib.multiply.restype = ctypes.c_int
lib.multiply.argtypes = [ctypes.c_int, ctypes.c_int]

lib.square.restype = ctypes.c_int
lib.square.argtypes = [ctypes.c_int]

#### Call Aria functions from Python
print(lib.add(10, 20))        # 30
print(lib.multiply(7, 8))     # 56
print(lib.square(9))          # 81
```

#### Python Type Mapping

| Aria Type | ctypes Type |
|-----------|-------------|
| `int8` | `ctypes.c_int8` |
| `int16` | `ctypes.c_int16` |
| `int32` | `ctypes.c_int` |
| `int64` | `ctypes.c_longlong` |
| `float32` | `ctypes.c_float` |
| `float64` | `ctypes.c_double` |
| `bool` | `ctypes.c_int` |
| `string` | `ctypes.c_char_p` |

---

#### Aria → Other Languages

Any language that supports C FFI can call Aria shared libraries:

| Language | Mechanism |
|----------|-----------|
| **Rust** | `extern "C"` + `#[link]` |
| **Go** | `cgo` with `// #cgo LDFLAGS: -lmathlib` |
| **Java** | JNI or JNA |
| **Node.js** | `ffi-napi` package |
| **Ruby** | `fiddle` or `ffi` gem |
| **C#** | `[DllImport]` P/Invoke |
| **Zig** | `@cImport` or `extern` |

Since Aria produces standard C ABI symbols, any language with C interop works.

---

#### Compiler Flags Reference

| Flag | Output | Use Case |
|------|--------|----------|
| (default) | Executable | Standalone programs |
| `-c` | Object file (.o) | Intermediate compilation |
| `--shared` | Shared library (.so) | Cross-language bindings |
| `--emit-llvm` | LLVM IR (.ll) | Debugging, inspection |
| `--emit-asm` | Assembly (.s) | Low-level inspection |

---

#### Symbol Naming

Aria functions use their declared name directly as the symbol name. There is no name mangling.

```aria
func:my_function = int32(int32:x) { pass(x); };
```

Produces symbol: `my_function` (visible via `nm -D libname.so`).

---

#### Limitations

- **Strings**: Aria string values passed across the boundary are C-style `char*` pointers. The caller is responsible for memory management of string arguments.
- **Complex types**: Structs and arrays are not yet supported across the FFI boundary. Use scalar types or pointers.
- **Async functions**: Async Aria functions cannot be called from C/Python directly (they return coroutine handles).
- **No header generation**: C header files must be written manually. Planned for future release.

\newpage

#### Undefined State Prevention: A Multi-Layered Approach

**Design Goal**: Never reach an undefined state. Ever.

**Key Insight**: No single mechanism can prevent all undefined states. Different kinds of "wrongness" require different safety layers.

#### The Three Safety Layers

#### Layer 1: Unknown (Computational Uncertainty)

**Purpose**: Handle indeterminate mathematical values

**Scope**: Operations that have no well-defined mathematical answer
- Division by zero: `5/0`, `0/0`
- Infinity arithmetic: `∞ - ∞`, `∞ / ∞`
- Complex number edge cases: `sqrt(-1)` (in real mode)
- Invalid operations on special numbers

**Behavior**: 
- Operation produces `Unknown` value instead of crashing
- Unknown propagates through calculations: `(5 + Unknown) → Unknown`
- Type system tracks: `ok(value)` returns `false` for Unknown
- Compiler enforces checking before use in critical contexts

**Philosophy**: These aren't errors - they're valid mathematical states representing "genuinely unknowable." The computation continues safely until the Unknown is checked.

**Example**:
```aria
func:safe_divide = int32(int32:a, int32:b) {
    int32:result = a / b;  // If b==0, result becomes Unknown
    
    if (!ok(result)) {
        pass(-1i32);  // Handle the Unknown case
    }
    
    pass(result);
}
```

**Implementation**: Division, modulo, and other unsafe operations inject runtime checks that produce Unknown sentinel values instead of undefined behavior.

---

#### Layer 2: Result<T> (Expected Failures)

**Purpose**: Handle operations that can legitimately fail

**Scope**: I/O, parsing, resource allocation, validation
- File operations: file not found, permission denied
- Network operations: timeout, connection refused
- Parsing: invalid format, out of range
- Optional computations: lookup not found

**Behavior**:
- Functions return `Result<T>` wrapper
- Caller must handle both success and error cases
- Unwrap operators provide ergonomic extraction:
  - `?` - casual unwrap with default
  - `??` - null coalesce for pointers/optionals
  - `?!` - emphatic unwrap (semantically stronger intent)
  - `_?` - shorthand for `drop()` — discard Result without checking
  - `_!` - shorthand for `raw()` — extract value without checking
- Compiler enforces "no checky no val" - can't access `.value` without checking `.is_error`

**Philosophy**: Failure is a normal part of the operation's contract, not an exceptional circumstance. The type system makes this explicit.

**Example**:
```aria
func:load_config = Result<Config>() {
    // Might fail - that's expected
    pass(readFile("config.json"));
}

func:main = int32() {
    // LOCAL handling: use default if loading fails
    Config:cfg = load_config() ?! getDefaultConfig();
    pass(0i32);
}
```

---

#### Layer 3: Failsafe (Catastrophic Failures)

**Purpose**: Emergency handler for unrecoverable errors

**Scope**: System-level failures that cannot be locally handled
- Out of memory
- Corrupted program state
- Assertion failures
- Stack overflow
- Hardware faults

**Behavior**:
- Every program **must** define `failsafe(int32:err_code)` function
- Compiler enforces presence - won't compile without it
- Explicit escalation via `!!!` operator
- Empty failsafe is valid - documents conscious decision
- Last line of defense before program termination

**Philosophy**: "Life preserver on the boat even if you can swim." You might never need it, but when the ship sinks, you'll be glad it's there. The requirement creates accountability - you cannot claim "I forgot."

**Example**:
```aria
// REQUIRED - every program must have this
func:failsafe = int32(int32:err_code) {
    // Log error, cleanup resources, notify operators
    // Even empty is valid - it documents your choice
    logCriticalError(err_code);
    cleanupResources();
    pass(err_code);
};

func:check_invariants = int32() {
    if (critical_system_invariant_violated) {
        // EMERGENCY: trigger global handler
        !!! INVARIANT_VIOLATION;
    }
    pass(0i32);
}
```

---

#### Why Multiple Layers?

**Single-layer approaches fail:**

1. **"Just use exceptions"**: 
   - Hidden control flow
   - Performance overhead
   - Doesn't distinguish expected vs catastrophic
   - Can be ignored

2. **"Just use Result everywhere"**:
   - Verbose for simple math
   - Wrong abstraction for "unknowable" values
   - Doesn't handle system-level failures

3. **"Just crash on errors"**:
   - Unsafe for production systems
   - No graceful degradation
   - Lost opportunity to handle recoverable cases

**Our approach**: Each layer handles what it's designed for

```
┌─────────────────────────────────────┐
│ Layer 3: Failsafe (catastrophic)   │  System-level, unrecoverable
│  !!!  operator, mandatory function  │  "Ship is sinking"
├─────────────────────────────────────┤
│ Layer 2: Result<T> (expected fail) │  Normal error conditions
│  ?, ??, ?! operators                │  "Operation might not work"
├─────────────────────────────────────┤
│ Layer 1: Unknown (indeterminate)   │  Mathematical uncertainty
│  ok() checker, propagation          │  "Answer is unknowable"
└─────────────────────────────────────┘
```

---

#### The `!` Connection

The exclamation mark semantically means "not right" across all uses:

- `!` - logical NOT: "this is not true"
- `?!` - emphatic unwrap: "this isn't right, but I have a fallback"
- `!!!` - failsafe call: "VERY MUCH NOT RIGHT - EMERGENCY"

This creates intuitive escalation from local to global error handling.

---

#### Design Guarantees

**No undefined states possible because:**

1. **Mathematical operations** → Unknown (never undefined)
2. **Expected failures** → Result<T> (type system enforces handling)
3. **Catastrophic failures** → failsafe() (compiler enforces presence)
4. **Null pointers** → Eliminated via optional types and null coalescing
5. **Uninitialized variables** → Compiler enforces initialization
6. **Type confusion** → Static type system prevents

**The programmer cannot:**
- Forget to implement failsafe (compiler error)
- Access Result value without checking (compile error)
- Use Unknown in critical context unchecked (type system)
- Dereference null (no null in type system)
- Reach undefined behavior (all paths covered)

---

#### Implementation Status

- [x] Layer 1: Unknown literal and propagation
- [x] Layer 2: Result<T> with unwrap operators (?, ??, ?!)
- [x] Layer 3: Mandatory failsafe() and !!! operator
- [ ] Divide-by-zero → Unknown conversion
- [ ] Overflow → Unknown conversion
- [ ] Integration testing across all layers

---

#### Usage Guidelines

**When to use Unknown:**
- Mathematical operations that can be indeterminate
- Computations that might not have answers
- Continue execution with "I don't know" values

**When to use Result<T>:**
- File I/O, network operations
- Parsing and validation
- Resource allocation
- Any operation where failure is a normal possibility

**When to use failsafe (!!!):**
- Assertion violations
- Invariant corruption
- Out of memory
- Unrecoverable system errors
- "This should never happen" scenarios

---

#### The Life Preserver Analogy

**Unknown** = Swim training
- Learn to handle rough conditions
- Part of normal operations
- Expected to encounter sometimes

**Result<T>** = Life jacket
- Comfortable to wear always
- Expected to need occasionally
- Normal safety equipment

**Failsafe** = Emergency life raft
- Mandatory on every vessel
- Hope to never use
- Critical when needed
- Required by regulation (compiler)

You can be an expert swimmer, but the boat still needs emergency equipment. The compiler enforces this because lives (or at least data) depend on it.

---

#### Direct Syscalls and Safety (v0.4.0)

The `sys()` builtin extends Aria's escalation model to kernel-level operations:

| Construct | Danger | Analogy |
|-----------|--------|---------|
| `sys(WRITE, ...)` | Safe | Life jacket — curated whitelist, Result-wrapped |
| `sys!!(KILL, ...)` | Medium | Emergency flare — all syscalls, still Result-wrapped |
| `sys!!!(nr, ...)` | High | Abandoning ship — bare int64, no safety net |

The safe tier cannot be bypassed with variables or expressions — only named constants from the compiler's whitelist. This prevents accidental escalation and makes dangerous syscall usage grep-able and auditable.

---

**Author's Note**: "I don't think one approach can cover it all. The unknown was the original idea, and I think it can cover most if we can somehow make things like divide by zero turn into unknown rather than undefined behind the scenes. Then they can actually handle that instead of crashing you know. But the failsafe was like... requiring people to have a life preserver on the boat even if they can swim. It's for when things don't go as planned."

\newpage

#### **Evaluation of the Aria Programming Language for Safety-Critical and Physics-Based AI Systems**

The engineering of safety-critical software—spanning domains such as aerospace flight control, nuclear reactor management, autonomous vehicular navigation, and the emerging field of Artificial General Intelligence (AGI)—demands programming paradigms that systematically eradicate undefined behavior, memory corruption, and non-deterministic execution.1 Historically, the software industry has addressed these stringent requirements by relying on heavily constrained subsets of legacy programming languages. For instance, C and C++ are frequently utilized under strict adherence to MISRA or AUTOSAR guidelines to mitigate their inherent vulnerabilities, while specialized languages such as Ada and its formally verifiable subset, SPARK, have long served as the gold standard for high-integrity systems.2 In recent years, Rust has emerged as a formidable alternative, introducing compile-time memory safety through its rigorous ownership model and borrowing semantics, thereby eliminating the need for garbage collection while preventing data races.5

The Aria programming language enters this highly rigorous ecosystem with a specialized focus on physics-based artificial intelligence models, wave-based computation, and AGI consciousness substrates.1 Aria attempts to reconcile the draconian safety requirements mandated by certification standards like DO-178C (for civil aviation) and ISO 26262 (for automotive functional safety) with the highly connected, graph-heavy, and mathematically intense memory patterns required by advanced physics simulations and neural architectures.1 By codifying a layered safety system, deterministic twisted floating-point arithmetic, generational memory handles, and compile-time dimensional analysis, Aria presents a novel and exhaustive architecture.1 If fully implemented and verified according to its specification, Aria represents a significant evolutionary branch in systems programming. It diverges from Rust’s static lifetime analysis in favor of structural isolation, emphasizes sticky error propagation over exception handling, and prioritizes absolute mathematical determinism across heterogeneous hardware.1 The following analysis evaluates the language's specified features against contemporary safety-critical standards and competing programming languages.

#### **The Epistemology of Error: Layered Safety and the Elimination of Undefined Behavior**

The defining characteristic of any programming language intended for safety-critical environments is its treatment of anomalous states and edge cases. In standard C and C++, operations such as signed integer overflow, division by zero, or out-of-bounds memory access result in undefined behavior (UB).1 Modern optimizing compilers, such as those based on the LLVM infrastructure, utilize undefined behavior as an aggressive optimization license; the compiler assumes UB will never occur and routinely eliminates safety branches or fabricates values to minimize the generated instruction count.17 In safety-critical contexts, this compiler behavior leads to catastrophic, silent divergences between the source code's logical intent and the executed machine code.16

Aria's design philosophy fundamentally rejects the concept of undefined behavior, operating under the strict doctrine that software must never reach an undefined state that causes an immediate, uncontrollable crash.1 To achieve this, the language constructs a defense-in-depth architecture composed of three distinct and non-negotiable safety layers: the Result\<T\> system for hard errors, the unknown sentinel state for soft errors, and the mandatory failsafe() routine for unrecoverable systemic failures.1

#### **The Result System and Explicit Accountability**

Aria’s primary error-handling layer mandates that all functions implicitly return a Result\<T\> struct, which physically encapsulates a success value, a void pointer to an error message, and a boolean error flag.1 This mechanism is conceptually analogous to Rust’s Result\<T, E\> enum, forcing the developer to explicitly acknowledge and handle the possibility of failure before the underlying data can be accessed.5 However, Aria streamlines the cognitive overhead by unifying the error type into a standardized structure, integrating it directly with language-level operators.1

The divergence from mainstream languages lies in Aria’s compiler-enforced accountability and syntactic integration. Languages like Java or Python rely on exceptions, which introduce hidden control flow paths, significant stack-unwinding overhead at runtime, and the persistent danger of unhandled exceptions crashing the host process.1 Conversely, C relies on integer return codes, which lack type safety and are effortlessly ignored by developers.1 By making the Result type pervasive and syntactically binding it to sticky error propagation, Aria ensures that recoverable errors—such as file I/O failures, network timeouts, or parsing errors—are handled explicitly at the boundary of their occurrence.1

Aria provides a suite of unwrap operators to interact with this system seamlessly. The safe unwrap operator (?) allows the developer to provide a default fallback value if the operation fails, ensuring continuous execution.1 The null coalescing operator (??) provides similar fallback functionality specifically for NIL optional values, differentiating between an explicit error state and a benign absence of data.1 Furthermore, Aria guarantees zero-overhead on the success path; checking the Result struct's boolean flag requires only a single branch instruction, which modern CPU branch predictors optimize to negligible latency.1

#### **The Unknown State: Enabling Fail-Operational Design**

The second layer of Aria’s safety architecture addresses operations that traditionally trigger hardware traps, panics, or undefined behavior, such as division by zero, invalid type conversions, or out-of-bounds array accesses.1 Instead of crashing the application or triggering a controlled panic (as Rust would in safe mode), Aria maps these anomalous operations to a defined unknown sentinel value.1

This design choice aligns flawlessly with the concept of "fail-operational" systems, a critical architectural requirement in modern aerospace design and highly automated driving frameworks.24 A fail-operational system must continue to function, albeit potentially in a degraded state, when a subsystem fails, rather than simply shutting down.26 By allowing an unknown value to propagate systematically through a chain of calculations—similar to how a Not-a-Number (NaN) value propagates in floating-point math, but highly controlled and type-checked across all data types—the software avoids abrupt termination.1

The developer is required to use the ok() function to explicitly verify the data's validity before it can be passed back into the strict Result\<T\> system or used in a critical control path.1 This mechanism provides the overall system a crucial operational window to apply analytical redundancy, execute sensor fusion algorithms, or trigger fallback logic before a complete functional collapse occurs.1

#### **The Mandatory Failsafe: Enforcing the Safe State**

When fail-operational mitigation strategies are exhausted, or when an error is fundamentally unrecoverable, safety-critical systems must seamlessly transition to a "fail-safe" mode, achieving a state where no harm can occur to human life or physical property.24 Aria codifies this transition at the compiler level by requiring every single program to define a global failsafe(int32:err\_code) function.1 This function acts as the ultimate, un-bypassable sink for unrecoverable errors, out-of-memory conditions, stack overflows, and assertions.1

This language-level mandate is a paradigm shift. While MISRA C guidelines and ISO 26262 standards strictly require systems to possess a safe state transition protocol—such as de-energizing industrial motors, dropping nuclear control rods, or severing fuel lines—these are traditionally implemented as architectural design patterns rather than compiler-enforced language semantics.10 Aria’s strict refusal to compile without a defined, non-empty failsafe() routine forces developers to codify their emergency termination protocols at the inception of the project.1

Aria introduces the emphatic unwrap operator (?\!) and the direct failsafe operator (\!\!\!) to interface with this layer. The ?\! operator attempts to unwrap a Result, but immediately invokes the failsafe() routine if an error is present, signaling that the software cannot safely continue without the requested data.1 The \!\!\! operator provides unconditional emergency termination, bypassing the Result system entirely to assert that an impossible state has been reached and the system must be halted.1

| Error Handling Paradigm | C / C++ | Rust | Ada / SPARK | Aria |
| :---- | :---- | :---- | :---- | :---- |
| **Expected Errors** | Integer Return Codes | Result\<T, E\> Enum | Exceptions / Out Params | Result\<T\> Struct (Mandatory) |
| **Math Anomalies / OOB** | Undefined Behavior | panic\! (Controlled Crash) | Exceptions / Formal Proof | unknown Sentinel (Fail-Operational) |
| **Terminal State** | abort(), exit() | panic\!, abort | Exception Propagation | failsafe() (Compiler Mandated) |
| **Safety Bypass** | Unrestricted | unsafe block | Restricted Features | Explicit TOS Operators (raw, drop) |

#### **Mathematical Determinism and Twisted Numeric Representation**

Aria’s primary target domain includes distributed Artificial General Intelligence substrates and hyper-accurate physics simulations. In these environments, identical mathematical operations must yield perfectly bit-exact results across heterogeneous hardware architectures (e.g., x86, ARM, RISC-V) to maintain network consensus and prevent the corruption of complex physical state manifolds.1 Standard floating-point arithmetic, governed by the IEEE 754 standard, catastrophically fails to provide this absolute guarantee.39

#### **The Rejection of IEEE 754**

IEEE 754 floating-point calculations are notoriously non-deterministic across different central processing units.39 Depending on the specific hardware and the compiler's optimization flags, instructions may be fused into Multiply-Add (FMA) operations, intermediate calculations may be temporarily stored in extended 80-bit precision registers, and rounding modes may be altered dynamically by shared mutable state within the floating-point environment.40 Furthermore, IEEE 754 introduces mathematically problematic artifacts such as negative zero (-0.0) and multiple binary representations of Not-a-Number (NaN).1 These artifacts can propagate silently through millions of operations, corrupting logical branching and destroying zero-neutrality.1 In the context of an AGI physics engine—where quantum states or neural topologies are iteratively calculated over sustained periods—even sub-atomic floating-point drift inevitably cascades into systemic divergence, fracturing the simulation.1

Aria completely abandons hardware-accelerated IEEE floats for critical calculations, providing its own strictly deterministic software-implemented types: fix256 (deterministic fixed-point) and tfp64 (Twisted Floating Point).1 The fix256 type utilizes a massive Q128.128 representation, allocating 128 bits for the integer component and 128 bits for the fractional component.1 This provides astronomical precision down to ![][image1] (significantly finer than the Planck length) while ensuring that all operations execute under the hood as pure, deterministic integer arithmetic.1 Fixed-point arithmetic avoids the non-associativity of floating-point math, making it the required standard for deterministic systems in aerospace telemetry and lockstep physics engines.39

For scientific applications requiring a massive dynamic range that fixed-point cannot provide, Aria introduces the tfp32 and tfp64 types. These construct a floating-point representation using Aria's specialized integer components for both the mantissa and the exponent, executing the arithmetic entirely in software.1 While this incurs a steep performance penalty (estimated at 10x to 50x slower than hardware FPU execution), it guarantees absolute cross-platform bit-exactness, collapses all mathematical error states into a single unified ERR sentinel, and entirely eliminates the \-0 anomaly.1

#### **Twisted Balanced Binary (TBB) and Sticky Error Propagation**

Aria addresses the vulnerabilities of standard integer arithmetic through its proprietary Twisted Balanced Binary (TBB) types, available in tbb8, tbb16, tbb32, and tbb64 variants.1 Standard two's complement integers exhibit a dangerous asymmetry; for example, an 8-bit signed integer ranges from \-128 to \+127. Because the absolute value of \-128 is \+128, which exceeds the maximum positive limit, attempting to negate the minimum value results in a silent wraparound or an overflow panic in languages like Rust.1

Aria’s TBB types structurally eliminate this vulnerability by sacrificing the absolute minimum value (e.g., \-128 in a tbb8) to serve as a hardware-level ERR sentinel.1 This yields a perfectly symmetric valid numeric range (-127 to \+127), making absolute value and negation operations mathematically flawless.1

More importantly, this architectural decision enables "sticky error propagation" at the bare-metal level.1 Whenever an overflow, underflow, or invalid operation occurs, the result collapses into the ERR sentinel.1 Any subsequent mathematical operation involving an ERR sentinel natively evaluates to ERR.1 This eliminates the need for the compiler to inject expensive branching checks after every single arithmetic instruction—a common and severe performance bottleneck in safe C++ implementations or Rust's checked-math modes.49 The application can execute a massive, vectorized pipeline of calculations and simply check for the ERR sentinel at the termination of the sequence, dramatically increasing throughput without sacrificing safety.1

#### **Exact Rational Arithmetic and LBIM Cryptographic Types**

Complementing its deterministic floats, Aria provides Fraction types (frac8 through frac64) to enable mathematically exact rational arithmetic.1 A frac struct maintains a whole number, a numerator, and a denominator, constantly reducing to canonical form via Greatest Common Divisor (GCD) algorithms to entirely prevent the accumulation of rounding errors common in recursive division.1

For high-security consensus algorithms and communications, Aria embeds Large BigInt Math (LBIM) types directly into the language primitives, scaling from int1024 up to a strictly enforced limit of int4096 and uint4096.1 These massive types are specifically designed for post-quantum and quantum-resistant cryptography (e.g., RSA-16384, lattice-based cryptography).1 By defining these cryptographic types as fundamental language primitives rather than relying on external, third-party libraries (which are historically frequent sources of vulnerabilities via side-channel attacks or memory mismanagement), Aria ensures that cryptographic operations inherit the exact same sticky error propagation, division-by-zero protection, and deterministic constraints as native integers.1

| Feature | IEEE 754 / Standard Int | Aria TBB / TFP / Fix | Safety & Physics Impact |
| :---- | :---- | :---- | :---- |
| **Symmetry** | Asymmetric (e.g., \-128 to 127\) | Symmetric (e.g., \-127 to 127\) | Prevents absolute value and negation overflow panics. |
| **Error State** | NaN (Float), Wraparound (Int) | Unified ERR Sentinel | Halts silent data corruption; explicitly tracked by compiler. |
| **Determinism** | Hardware-dependent, FMA drift | Bit-exact cross-platform | Ensures identical simulations across heterogeneous AGI nodes. |
| **Propagation** | NaN poisoning (silent) | Sticky ERR propagation | Allows batch processing without branch-per-instruction overhead. |

#### **Memory Architecture: Generational Handles vs. The Borrow Checker**

Memory safety remains the paramount challenge in systems programming. Vulnerabilities such as use-after-free (UAF), double-free, and buffer overflows account for the vast majority of critical security exploits and system crashes worldwide.21 Rust revolutionized this space by introducing the borrow checker, a compile-time mechanism that enforces strict single-mutable ownership and tracks lifetimes to guarantee memory safety without relying on a garbage collector.5

However, the strict, tree-based ownership hierarchy demanded by Rust's borrow checker is notoriously hostile to the highly connected, self-referential graph data structures required in AGI neural topologies, real-time physics simulations, and Entity Component Systems (ECS).57 To build complex graphs in Rust, developers are often forced to wrap data in runtime reference counting constructs (Rc or Arc), which introduce significant performance overhead and thread contention. Alternatively, they fall back to using integer indices into arrays, which circumvents the borrow checker but merely masks the memory management problem without guaranteeing temporal safety against stale indices.56

#### **Generational Handles and Arena Allocation**

Aria aggressively solves the graph-structure problem by adopting Generational Handles paired with Arena allocators as a fundamental language construct.1 A Handle\<T\> is a lightweight, 12-byte structure comprising an index (uint64) and a generation counter (uint32).1 It acts as a safe reference to memory, completely replacing raw pointers for dynamic, interconnected data.1

When an object—such as a wave node in Aria's physics engine or a neuron in its Sparse Hyper-Voxel Octree (SHVO)—is allocated in a typed arena, the arena issues a handle containing the slot's current generation number.1 If the arena is forced to reallocate (e.g., during exponential memory expansion or "neurogenesis"), or if the specific object is explicitly freed, the arena increments the generation counter for that memory slot.1 When the program subsequently attempts to access the data using the old handle, the generation mismatch is immediately detected.1 Instead of allowing a catastrophic use-after-free read or write, the arena.get() function safely returns an ERR via the Result\<T\> system.1

This paradigm fundamentally shifts memory safety from a static compile-time proof (as seen in Rust) to an ![][image2] dynamic runtime check.1 While this theoretically introduces runtime overhead, modern CPU branch predictors optimize the generation comparison check to near-zero latency on the hot path.1 Generational handles provide deterministic, rapid, and completely memory-safe graph representations, granting Aria a vital architectural advantage over Rust for building scalable AGI memory substrates without wrestling with lifetime annotations.1

#### **Explicit Allocation Modes and RAII Semantics**

To accommodate diverse real-time constraints and prevent hidden performance latency, Aria mandates explicit allocation modes, removing the dangerous ambiguity of implicit heap allocations found in many high-level languages.1

* **stack**: The default mode, offering ultra-fast ![][image2] allocation via pointer bumping, constrained strictly by lexical scope lifetimes.  
* **gc**: Opt-in garbage collection for non-critical code with complex shared ownership, though heavily discouraged in deterministic physics loops due to unpredictable pause times.  
* **wild**: Manual, unmanaged heap allocation (akin to C's malloc) designed for absolute real-time determinism where the developer takes full responsibility.  
* **wildx**: Executable memory allocation strictly isolated for Just-In-Time (JIT) compilation requirements.1

When utilizing wild memory, Aria mitigates the severe risk of memory leaks through the defer keyword.1 The defer statement guarantees that designated cleanup code executes in Last-In-First-Out (LIFO) order upon scope exit.1 Crucially, this execution is guaranteed regardless of whether the scope exits normally, via an early return, or through an error/failsafe trigger.1 This provides the safety benefits of C++ Resource Acquisition Is Initialization (RAII) without the immense complexity of hidden constructor/destructor control flows.1

#### **Borrow Semantics and Pinning**

While generational handles dominate dynamic graph allocations, later updates to the Aria specification (v0.2.35) incorporated an opt-in compile-time borrow checker to provide localized memory safety and prevent data races.1 Utilizing the $$m (mutable, exclusive) and $$i (immutable, shared) qualifiers, Aria enforces standard aliasing rules to ensure safe concurrent access during parallel physics computations.1 Furthermore, Aria introduces the pinning operator (\#), which guarantees that a variable's memory address remains absolutely stable. This prevents the garbage collector from relocating the data during optimization passes, an essential feature for seamless, zero-copy Foreign Function Interface (FFI) integration with external C libraries.1

| Memory Management Strategy | C / C++ | Rust | Aria |
| :---- | :---- | :---- | :---- |
| **Primary Safety Mechanism** | Manual / Smart Pointers | Borrow Checker / Lifetimes | Generational Handles / Arenas |
| **Temporal Safety (Use-After-Free)** | Unsafe (Developer Responsibility) | Statically Proven at Compile-Time | Dynamically Checked (Handles) |
| **Graph / Tree Structures** | Trivial but entirely unsafe | Difficult (Requires Rc / unsafe) | Trivial and fully safe (Handles) |
| **Cleanup Guarantees** | Manual / RAII | RAII / Drop trait | defer keyword (LIFO execution) |
| **Allocation Visibility** | Implicit / Library calls | Implicit (Box, Vec) | Explicit keywords (stack, gc, wild) |

#### **Semantic Correctness: Contracts, Limits, and Dimensional Analysis**

Beyond securing memory and preventing arithmetic overflow, safety-critical software must verify that its operations logically align with physical reality and business requirements.38 A structurally flawless program that attempts to add velocity to mass still produces a fundamentally flawed physics simulation, potentially leading to catastrophic real-world outcomes.1

#### **Compile-Time Dimensional Analysis**

Aria elevates physical units of measurement to the type system level, instituting strict, compile-time dimensional analysis.1 By parameterizing numeric types with physical units (e.g., fix256\<Joules\>, fix256\<Meters\>, tfp64\<Newtons\>), the Aria compiler mechanically verifies thermodynamic and physical equations prior to execution.1 Attempting to add time to energy, or equating mass to energy without explicitly applying the ![][image3] conversion coefficient, triggers an immediate compile-time error.1

While dimensional analysis is relatively rare in mainstream, general-purpose languages, it is highly prized in scientific computing and aerospace engineering. F\# successfully implements units of measure, verifying the logic and then stripping the metadata at compile time to incur zero runtime performance overhead.69 The Ada 2012 specification similarly introduced dimensionality checking specifically to prevent multi-million-dollar disasters akin to the Mars Climate Orbiter unit-conversion failure.72 By mandating this at the core of the language rather than relying on external libraries, Aria prevents entire categories of physics errors—a rigorous paradigm the language specification refers to as "Thermodynamic Constitutionalism".1

#### **Design by Contract and Refinement Types**

To further enforce semantic correctness, Aria integrates Design by Contract (DbC) natively into its function syntax using the requires (preconditions) and ensures (postconditions) clauses.1 If an input parameter violates a requires clause at runtime, the function immediately intercepts the execution and automatically returns an error Result, intertwining contractual constraints directly with the sticky error propagation system.1

Aria extends this philosophy with "Rules" and "Limits," effectively implementing Refinement Types.1 Developers can define logical boundaries—such as Rules:r\_safe\_temp \= { $ \>= \-40, $ \<= 120 }—and apply them to variable declarations using the limit keyword.1 If a literal value violates the limit, the compiler throws an error; if a dynamic runtime value breaches the boundary, it triggers the failsafe.1

It is critical to contrast Aria's approach to contracts with the formal methods used by Ada/SPARK. SPARK utilizes deductive formal verification, employing powerful SMT solvers (like Z3, Alt-Ergo, or CVC4) to mathematically *prove* the Absence of Runtime Errors (AoRTE) and verify contractual compliance before the code is ever compiled into an executable.65 SPARK can formally guarantee that a buffer overflow or contract violation is impossible across all conceivable execution paths.74

As of v0.3.4, Aria has integrated the Z3 SMT solver directly into its compiler pipeline, enabling static formal verification of `requires`/`ensures` contracts and integer arithmetic overflow at compile time.1 The `--verify-contracts` flag instructs the compiler to translate function preconditions and postconditions into Z3 assertions and mathematically prove their validity across all possible inputs. The `--verify-overflow` flag uses Z3's bitvector overflow intrinsics (`bvadd_no_overflow`, `bvsub_no_underflow`) to prove that integer arithmetic operations cannot overflow for bounded input ranges. A `--verify-report` flag emits detailed proof results per function (proven, unproven, or skipped).

This places Aria's contractual verification power in the same class as SPARK's formal methods—using the same Z3 solver backend—while integrating the proofs directly into Aria's existing Result and failsafe error propagation systems. Aria's Z3 integration also extends to its Rules/limit refinement types (since v0.2.45), where the solver verifies the consistency and satisfiability of compile-time constraints using bitvector-accurate proofs that match Aria's exact integer widths. Combined with the runtime enforcement of contracts and limits as a fallback for cases the solver cannot statically prove, Aria now provides both static and dynamic assurance for contractual compliance.65

#### **Concurrency, Polymorphism, and Performance**

High-performance physics engines and AGI substrates require massively parallel execution across multi-core architectures with minimal synchronization overhead.1 Aria addresses this by mandating Sequential Consistency (SeqCst) as the default memory ordering for all atomic operations.1 This prioritizes developer reasoning and state safety over the raw, unpredictable speed of relaxed memory models.1 The language provides robust, safe primitives for Lock-Free Queues, Seqlocks, and Transactional Locks, requiring developers to explicitly mark weaker memory orderings (using suffixes like \_acquire, \_release, \_relaxed) to alert security auditors to expert-level hardware optimizations.1

For hardware acceleration, Aria introduces explicit SIMD (Single Instruction, Multiple Data) vector types (e.g., simd\<fix256, 16\>) tailored for modern processor extensions like AVX-512 and ARM NEON.1 Crucially, SIMD operations in Aria utilize masked operations to handle sticky errors without relying on CPU branching.1 This ensures that safety checks do not destroy vectorization throughput—a pervasive limitation in auto-vectorizing C++ compilers where the introduction of a safety branch forces the compiler to abandon SIMD optimizations.1

#### **Monomorphization vs. Dynamic Dispatch**

Aria achieves zero-cost abstraction for generic programming via monomorphization, a technique identical to C++ templates and Rust generics.1 When a generic function or struct is utilized, the compiler generates a specialized, hard-coded copy of that function for the specific type being processed.1 This completely eliminates the need for virtual function tables (vtables) and dynamic dispatch overhead at runtime.1

While monomorphization drastically improves execution speed and allows the compiler to aggressively inline code, it risks "binary bloat"—a legitimate and significant concern for flash-constrained embedded systems in aerospace and automotive hardware.80 However, the sheer execution speed and cache-locality afforded by monomorphization are absolutely essential for Aria to achieve the strict 1-millisecond simulation timesteps demanded by advanced, multi-dimensional AGI physics grids.1

To complement its memory and concurrency models, recent updates to the Aria specification (v0.2.37) introduced Channels and Actors to facilitate thread-safe message passing.1 By providing buffered, unbuffered, and oneshot channels alongside isolated Actor entities, Aria embraces structured concurrency paradigms heavily inspired by Go and Erlang, preventing the catastrophic state corruption that occurs when multiple threads attempt to mutate shared global memory.1

#### **Expressive Paradigms and Architectural Syntax**

While heavily focused on bare-metal safety, Aria provides modern, expressive paradigms to maintain developer productivity without compromising its core tenets. The language treats functions as first-class citizens, supporting closures and lambda expressions.1 To maintain strict scoping safety, Aria differentiates between safe value capture (which copies data into the closure) and unsafe reference capture (which requires explicit pointer dereferencing and lifetime management by the developer).1

Aria's syntax is meticulously designed to reduce cognitive load and prevent ambiguity. It rejects the overloaded operators common in C++ in favor of distinct, blueprint-style operators that visually indicate the direction of data flow.1 For instance, int64-\>:ptr denotes a pointer pointing *to* an integer type, @var represents the address *at* a variable, and \<-ptr clearly indicates data being pulled *from* a pointer during dereferencing.1

Furthermore, Aria supports advanced string interpolation via template literals (e.g., \`Value: &{x}\`), allowing complex expressions and function calls to be evaluated safely within strings, complete with automatic type conversion and multiline support.1 The module system relies on explicit use, mod, and pub keywords, enforcing strict namespace isolation and facilitating clean dependency management in large-scale codebases.1

#### **Regulatory Compliance and the Explicit TOS Bypass**

When evaluating a programming language for commercial safety-critical deployment, technical features must translate into verifiable regulatory compliance capabilities, specifically targeting rigorous frameworks like RTCA DO-178C (Civil Aviation) and ISO 26262 (Automotive Functional Safety).9 Aria's design presents several major advantages for certification:

1. **Zero Implicit Conversion:** Aria strictly enforces literal type suffixes and completely eliminates implicit type coercions.1 This natively enforces at the compiler level one of the most critical and frequently violated rules in the MISRA C coding standard, drastically reducing the required static analysis burden prior to certification audits.1  
2. **Absence of Unintended Behavior:** DO-178C places a heavy emphasis on predictable data flow, control coupling, and hardware determinism.13 Aria’s systemic removal of undefined behavior, combined with the deterministic fixed-point (fix256) and twisted floating-point (tfp64) math, provides profound assurances that the software will execute exactly as modeled across all operational states.1  
3. **Safe State Transitions:** ISO 26262 requires systems to identify hazards and guarantee a transition to a defined Safe State upon failure.10 Aria’s language-mandated failsafe() function serves as the ultimate, un-bypassable conduit for this exact requirement, ensuring that Architectural Safety Integrity Level (ASIL) D targets for runtime monitoring and emergency response are structurally satisfied from day one.1

#### **The TOS Vocabulary vs. Rust Unsafe**

In systems programming, developers inevitably require mechanisms to bypass high-level safety constraints to interact directly with hardware or optimize critical paths. Rust utilizes the monolithic unsafe keyword to unlock raw pointer dereferencing, mutable statics, and unchecked array access.54 However, the broad scope of unsafe has drawn scrutiny from security researchers regarding poor developer risk assessment, the lack of granular security policies, and the difficulty of auditing precisely *which* safety rule is being bypassed within an unsafe block.54

Aria categorizes its explicit bypasses into a discrete, highly auditable "Terms of Service" (TOS) vocabulary.1 Instead of a single unsafe block, developers must use specific keywords for specific actions: raw() forces a Result unwrap without checking the error flag; drop() evaluates an expression but throws away the safety Result entirely; wild allocates unmanaged memory; and ok() deliberately strips an unknown sentinel taint from a variable.1 This extreme granularity heavily favors compliance certification, allowing auditors to precisely track and justify where error checking was bypassed versus where memory management was handled manually.1

Despite these immense architectural strengths, Aria currently lacks the decades of ecosystem maturity, certified compiler toolchains, and established developer familiarity that C, C++, and Ada possess.7 To achieve DAL A (DO-178C) or ASIL D (ISO 26262\) certification in commercial settings, the Aria compiler itself must undergo rigorous qualification, and structural coverage analysis tools (such as MC/DC analyzers) must be developed specifically for the language.10

#### **Conclusion**

The Aria programming language represents a highly specialized, intensely rigorous approach to safety-critical software architecture. By synthesizing the graph-friendly memory safety of generational handles with the structural accountability of a mandatory failsafe system, and underpinning it all with the absolute determinism of fixed-point and twisted-binary arithmetic, Aria addresses the precise failure modes that plague C/C++ and Rust in highly concurrent, physics-based AI simulations.1

While it presently lacks the static formal verification ecosystem of Ada/SPARK 74, its architectural refusal to allow undefined behavior, its fail-operational unknown state, and its strict thermodynamic constraints position Aria as an exceptionally potent language for its intended domain.1 By making safety an inescapable structural requirement rather than a developer convention, Aria possesses the foundational semantics necessary to become a dominant language in the development of fault-tolerant AGI, aerospace control systems, and high-fidelity physics substrates.

**Update (v0.3.4, March 2026):** Aria has since integrated the Z3 SMT solver into its compiler pipeline, adding static formal verification of `requires`/`ensures` contracts and integer arithmetic overflow proofs. This directly addresses the formal verification gap identified above, placing Aria alongside Ada/SPARK in providing compile-time mathematical correctness guarantees for safety-critical code.

#### **Works cited**

1. aria\_specs.txt  
2. Should I choose Ada, SPARK, or Rust over C/C++? (2024) | Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=45486829](https://news.ycombinator.com/item?id=45486829)  
3. Fixed-Point Arithmetic Unit with a Scaling Mechanism for FPGA-Based Embedded Systems, accessed March 28, 2026, [https://www.mdpi.com/2079-9292/10/10/1164](https://www.mdpi.com/2079-9292/10/10/1164)  
4. IAEA Safety Standards — Deterministic Safety Analysis for Nuclear Power Plants, accessed March 28, 2026, [IAEA PUB1851](https://www-pub.iaea.org/MTCD/Publications/PDF/PUB1851_web.pdf)  
5. Should I choose Ada, SPARK, or Rust over C/C++? \- AdaCore, accessed March 28, 2026, [https://www.adacore.com/blog/should-i-choose-ada-spark-or-rust-over-c-c](https://www.adacore.com/blog/should-i-choose-ada-spark-or-rust-over-c-c)  
6. Bringing Rust to Safety-Critical Systems in Space \- arXiv, accessed March 28, 2026, [https://arxiv.org/html/2405.18135v1](https://arxiv.org/html/2405.18135v1)  
7. What's The Best Language For Safety Critical Software? — Reddit, accessed March 28, 2026, [r/programming](https://www.reddit.com/r/programming/comments/te5rb/whats_the_best_language_for_safety_critical/)  
8. Programming Languages in Safety-Critical Applications \- adesso SE, accessed March 28, 2026, [https://www.adesso.de/en/news/blog/programming-languages-in-safety-critical-applications.jsp](https://www.adesso.de/en/news/blog/programming-languages-in-safety-critical-applications.jsp)  
9. Rust is DO-178C Certifiable \- Pictorus, accessed March 28, 2026, [https://blog.pictor.us/rust-is-do-178-certifiable/](https://blog.pictor.us/rust-is-do-178-certifiable/)  
10. Functional Safety in Automotive: ISO 26262 Testing Best Practices | QA Systems, accessed March 28, 2026, [https://www.qa-systems.com/blog/iso-26262-testing-best-practices/](https://www.qa-systems.com/blog/iso-26262-testing-best-practices/)  
11. Meeting ISO 26262 Guidelines \- Black Duck, accessed March 28, 2026, [https://www.blackduck.com/resources/white-papers/ISO26262-guidelines.html](https://www.blackduck.com/resources/white-papers/ISO26262-guidelines.html)  
12. Master Thesis — Comparison Ada C Rust, accessed March 28, 2026, [ILS Stuttgart](https://www.ils.uni-stuttgart.de/dokumente/2025-08-04_Masterarbeit_Comparison_Ada_C_Rust_V0.pdf)  
13. DO-178C Compliance in Static Analysis — Parasoft, accessed March 28, 2026, [Parasoft](https://www.parasoft.com/learning-center/do-178c/static-analysis/)  
14. The Most Memory Safe Native Programming Language, accessed March 28, 2026, [https://vale.dev/memory-safe](https://vale.dev/memory-safe)  
15. Ask HN: What if a language's structure determined memory lifetime? | Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=46488290](https://news.ycombinator.com/item?id=46488290)  
16. The Compiler as Sentinel: A Comprehensive Analysis of Compiler-Oriented Software Security | by Vipul Kumar | Medium, accessed March 28, 2026, [https://medium.com/@vipulkc/the-compiler-as-sentinel-a-comprehensive-analysis-of-compiler-oriented-software-security-f75428dbe820](https://medium.com/@vipulkc/the-compiler-as-sentinel-a-comprehensive-analysis-of-compiler-oriented-software-security-f75428dbe820)  
17. Taming Undefined Behavior in LLVM \- Microsoft Research, accessed March 28, 2026, [https://www.microsoft.com/en-us/research/publication/taming-undefined-behavior-llvm/](https://www.microsoft.com/en-us/research/publication/taming-undefined-behavior-llvm/)  
18. A production bug that made me care about undefined behavior \- Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=46423521](https://news.ycombinator.com/item?id=46423521)  
19. Failure is Required: Understanding Fail-Safe and Fail-Fast Strategies \- Medium, accessed March 28, 2026, [https://medium.com/javarevisited/failure-is-required-understanding-fail-safe-and-fail-fast-strategies-ac9112fe056d](https://medium.com/javarevisited/failure-is-required-understanding-fail-safe-and-fail-fast-strategies-ac9112fe056d)  
20. What's your opinion on exceptions? : r/ProgrammingLanguages \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/ProgrammingLanguages/comments/o1ye66/whats\_your\_opinion\_on\_exceptions/](https://www.reddit.com/r/ProgrammingLanguages/comments/o1ye66/whats_your_opinion_on_exceptions/)  
21. What Is the Most Secure Coding Language? Top Options in 2026, accessed March 28, 2026, [https://www.securityjourney.com/post/what-is-the-most-secure-coding-language](https://www.securityjourney.com/post/what-is-the-most-secure-coding-language)  
22. Software Error Incident Categorizations in Aerospace, accessed March 28, 2026, [https://arc.aiaa.org/doi/10.2514/1.I011240](https://arc.aiaa.org/doi/10.2514/1.I011240)  
23. Safety measures : r/AskProgramming \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/AskProgramming/comments/16pgxfy/safety\_measures/](https://www.reddit.com/r/AskProgramming/comments/16pgxfy/safety_measures/)  
24. On Safety-Critical Software Development \- Increment, accessed March 28, 2026, [https://increment.com/reliability/safety-critical-software-development/](https://increment.com/reliability/safety-critical-software-development/)  
25. Safe Software for Automated Driving \- Functional Safety for High Performance ECUs \- Vector, accessed March 28, 2026, [https://cdn.vector.com/cms/content/know-how/\_technical-articles/Safety\_Automated\_Driving\_ATZelektronik\_202007\_PressArticle\_EN.pdf](https://cdn.vector.com/cms/content/know-how/_technical-articles/Safety_Automated_Driving_ATZelektronik_202007_PressArticle_EN.pdf)  
26. A Taxonomy to Unify Fault Tolerance Regimes for Automotive Systems: Defining Fail-Operational, Fail-Degraded, and Fail-Safe, accessed March 28, 2026, [https://d-nb.info/1268108790/34](https://d-nb.info/1268108790/34)  
27. What Does It Mean to Fail Safe?. “Failure is inevitable, but surprise is… | by Tobe Onyema | Medium, accessed March 28, 2026, [https://medium.com/@tobe.onyema/what-does-it-mean-to-fail-safe-ba6e33db2479](https://medium.com/@tobe.onyema/what-does-it-mean-to-fail-safe-ba6e33db2479)  
28. Technical Report Fail-Operational and Fail-Safe Vehicle Platooning in the Presence of Transient Communication Errors \- Diva-portal.org, accessed March 28, 2026, [https://www.diva-portal.org/smash/get/diva2:1646357/FULLTEXT01.pdf](https://www.diva-portal.org/smash/get/diva2:1646357/FULLTEXT01.pdf)  
29. Software Reliability and Safety in Nuclear Reactor Protection Systems, accessed March 28, 2026, [https://www.nrc.gov/reading-rm/doc-collections/nuregs/contract/cr6101/cr6101.pdf](https://www.nrc.gov/reading-rm/doc-collections/nuregs/contract/cr6101/cr6101.pdf)  
30. Is it safe to „let it crash”? : r/programming \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/programming/comments/1mdlbw/is\_it\_safe\_to\_let\_it\_crash/](https://www.reddit.com/r/programming/comments/1mdlbw/is_it_safe_to_let_it_crash/)  
31. Reinforcement Learning for Fail-Operational Systems with Disentangled Dual-Skill Variables \- MDPI, accessed March 28, 2026, [https://www.mdpi.com/2227-7080/13/4/156](https://www.mdpi.com/2227-7080/13/4/156)  
32. The Safe State: Design Patterns and Degradation Mechanisms for Fail- Operational Systems \- Elektrobit, accessed March 28, 2026, [https://www.elektrobit.com/wp-content/uploads/2015/11/The\_safe\_state-Design\_patterns\_and\_degradation\_mechanisms\_for\_fail-operational\_systems.pdf](https://www.elektrobit.com/wp-content/uploads/2015/11/The_safe_state-Design_patterns_and_degradation_mechanisms_for_fail-operational_systems.pdf)  
33. Fail-safe \- Wikipedia, accessed March 28, 2026, [https://en.wikipedia.org/wiki/Fail-safe](https://en.wikipedia.org/wiki/Fail-safe)  
34. Understanding Fail-Safe PLC Systems: Hardware and Programming Guide, accessed March 28, 2026, [https://industrialmonitordirect.com/blogs/knowledgebase/understanding-fail-safe-plc-systems-hardware-and-programming-guide](https://industrialmonitordirect.com/blogs/knowledgebase/understanding-fail-safe-plc-systems-hardware-and-programming-guide)  
35. What to Expect with Version 3 of ISO 26262 \- UL Solutions, accessed March 28, 2026, [https://www.ul.com/sis/blog/what-to-expect-with-version-3-of-iso-26262](https://www.ul.com/sis/blog/what-to-expect-with-version-3-of-iso-26262)  
36. Functional Design Specifications (FDS) \- IACS Engineering, accessed March 28, 2026, [https://www.iacsengineering.com/functional-specifications/](https://www.iacsengineering.com/functional-specifications/)  
37. Multicore processors and critical embedded systems: WCET, interference research, and other challenges \- LDRA, accessed March 28, 2026, [https://ldra.com/capabilities/mcp/](https://ldra.com/capabilities/mcp/)  
38. SUMMARY OF TOPICS \- Electrical and Computer Engineering, accessed March 28, 2026, [https://users.ece.cmu.edu/\~koopman/pubs/191213\_UL4600\_VotingVersion.pdf](https://users.ece.cmu.edu/~koopman/pubs/191213_UL4600_VotingVersion.pdf)  
39. The most useful feature of fixed point calculations is its deterministic nature.... \- Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=10061759](https://news.ycombinator.com/item?id=10061759)  
40. What are the pros and cons of fixed-point arithmetics vs floating-point arithmetics?, accessed March 28, 2026, [https://langdev.stackexchange.com/questions/665/what-are-the-pros-and-cons-of-fixed-point-arithmetics-vs-floating-point-arithmet](https://langdev.stackexchange.com/questions/665/what-are-the-pros-and-cons-of-fixed-point-arithmetics-vs-floating-point-arithmet)  
41. How is the development process for mission critical software? : r/cpp \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/cpp/comments/5k2v3v/how\_is\_the\_development\_process\_for\_mission/](https://www.reddit.com/r/cpp/comments/5k2v3v/how_is_the_development_process_for_mission/)  
42. Does flight software use floating point or fixed points for its calculations? Why? \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/aerospace/comments/acsn0n/does\_flight\_software\_use\_floating\_point\_or\_fixed/](https://www.reddit.com/r/aerospace/comments/acsn0n/does_flight_software_use_floating_point_or_fixed/)  
43. Valori: A Deterministic Memory Substrate for AI Systems \- arXiv, accessed March 28, 2026, [https://arxiv.org/html/2512.22280v1](https://arxiv.org/html/2512.22280v1)  
44. Automated Fixed-Point Precision Optimization for FPGA Synthesis \- IEEE Xplore, accessed March 28, 2026, [https://ieeexplore.ieee.org/iel8/8784029/10830546/11039693.pdf](https://ieeexplore.ieee.org/iel8/8784029/10830546/11039693.pdf)  
45. Deterministic Atomic Buffering \- IEEE/ACM International Symposium on Microarchitecture, accessed March 28, 2026, [https://www.microarch.org/micro53/papers/738300a981.pdf](https://www.microarch.org/micro53/papers/738300a981.pdf)  
46. Impacts of floating-point non-associativity on reproducibility for HPC and deep learning applications This manuscript has been authored by UT-Battelle, LLC under Contract No. DE-AC05-00OR22725 with the U.S. Department of Energy. The United States Government retains and the publisher, by accepting the article for publication, acknowledges that the United \- arXiv, accessed March 28, 2026, [https://arxiv.org/html/2408.05148v1](https://arxiv.org/html/2408.05148v1)  
47. Arithmetic with binary-encoded balanced ternary numbers | Request PDF \- ResearchGate, accessed March 28, 2026, [https://www.researchgate.net/publication/269308357\_Arithmetic\_with\_binary-encoded\_balanced\_ternary\_numbers](https://www.researchgate.net/publication/269308357_Arithmetic_with_binary-encoded_balanced_ternary_numbers)  
48. Diff \- 49b07690a93a813585047d8265e326de8efd95c6^\! \- gofrontend \- Git at Google \- Google Git, accessed March 28, 2026, [https://go.googlesource.com/gofrontend/+/49b07690a93a813585047d8265e326de8efd95c6%5E%21/](https://go.googlesource.com/gofrontend/+/49b07690a93a813585047d8265e326de8efd95c6%5E%21/)  
49. Towards Smart Contract Fuzzing on GPUs \- About Me, accessed March 28, 2026, [https://chapering.github.io/pubs/sp24weimin.pdf](https://chapering.github.io/pubs/sp24weimin.pdf)  
50. The Architecture of Reliability: A Comprehensive Treatise on CUDA Error Handling and Debugging Methodologies | Uplatz Blog, accessed March 28, 2026, [https://uplatz.com/blog/the-architecture-of-reliability-a-comprehensive-treatise-on-cuda-error-handling-and-debugging-methodologies/](https://uplatz.com/blog/the-architecture-of-reliability-a-comprehensive-treatise-on-cuda-error-handling-and-debugging-methodologies/)  
51. Finding And Preventing Bugs In c++ \- Volt Software, accessed March 28, 2026, [https://volt-software.nl/posts/finding-and-preventing-bugs-in-cpp/](https://volt-software.nl/posts/finding-and-preventing-bugs-in-cpp/)  
52. Analysis of Error Propagation in Safety Critical Software Systems: An Approach Based on UGF | Request PDF \- ResearchGate, accessed March 28, 2026, [https://www.researchgate.net/publication/315066106\_Analysis\_of\_Error\_Propagation\_in\_Safety\_Critical\_Software\_Systems\_An\_Approach\_Based\_on\_UGF](https://www.researchgate.net/publication/315066106_Analysis_of_Error_Propagation_in_Safety_Critical_Software_Systems_An_Approach_Based_on_UGF)  
53. Comparative Analysis of Programming Languages in Cryptography \- Diva-portal.org, accessed March 28, 2026, [http://www.diva-portal.org/smash/get/diva2:1885509/FULLTEXT01.pdf](http://www.diva-portal.org/smash/get/diva2:1885509/FULLTEXT01.pdf)  
54. “I wouldn't want my unsafe code to run my pacemaker”: An Interview Study on the Use, Comprehension, and Perceived Risks of \- USENIX, accessed March 28, 2026, [https://www.usenix.org/system/files/usenixsecurity23-holtervennhoff.pdf](https://www.usenix.org/system/files/usenixsecurity23-holtervennhoff.pdf)  
55. I've heard that "Rust's borrow checker is necessary to ensure memory safety without a GC" usually also implying it's the only way, but I've done the same without the borrow checker. Am I just clueless/confused? \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/rust/comments/16gb7pc/ive\_heard\_that\_rusts\_borrow\_checker\_is\_necessary/](https://www.reddit.com/r/rust/comments/16gb7pc/ive_heard_that_rusts_borrow_checker_is_necessary/)  
56. Borrow checking, RC, GC, and the Eleven (\!) Other Memory Safety Approaches, accessed March 28, 2026, [https://verdagon.dev/grimoire/grimoire](https://verdagon.dev/grimoire/grimoire)  
57. r/gamedev on Reddit: Rant: Entity systems and the Rust borrow checker ... or something., accessed March 28, 2026, [https://www.reddit.com/r/gamedev/comments/9fq828/rant\_entity\_systems\_and\_the\_rust\_borrow\_checker/](https://www.reddit.com/r/gamedev/comments/9fq828/rant_entity_systems_and_the_rust_borrow_checker/)  
58. Jonathan Blow: Entity Systems and the Rust Borrow Checker... or something \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/rust/comments/9fqget/jonathan\_blow\_entity\_systems\_and\_the\_rust\_borrow/](https://www.reddit.com/r/rust/comments/9fqget/jonathan_blow_entity_systems_and_the_rust_borrow/)  
59. I like Odin | Brian Lovin, accessed March 28, 2026, [https://brianlovin.com/hn/32626543](https://brianlovin.com/hn/32626543)  
60. Memory safety without borrow checking, reference counting, or garbage collection | Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=36351415](https://news.ycombinator.com/item?id=36351415)  
61. I like Odin | Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=32626543](https://news.ycombinator.com/item?id=32626543)  
62. All | Search powered by Algolia, accessed March 28, 2026, [https://hn.algolia.com/?query=%E2%80%9CIt%20is%20never%20a%20compiler%20error%E2%80%9D\&type=story\&dateRange=all\&sort=byDate\&storyText=false\&prefix\&page=0](https://hn.algolia.com/?query=%E2%80%9CIt+is+never+a+compiler+error%E2%80%9D&type=story&dateRange=all&sort=byDate&storyText=false&prefix&page=0)  
63. Who Owns the Memory? Part 3: How Big Is your Type? | Luca Lombardo, accessed March 28, 2026, [https://lukefleed.xyz/posts/who-owns-the-memory-pt3/](https://lukefleed.xyz/posts/who-owns-the-memory-pt3/)  
64. RKOS: UNIKERNEL DESIGN FOR SAFETY AND PERFORMANCE \- Index of /, accessed March 28, 2026, [https://media.taricorp.net/rkos.pdf](https://media.taricorp.net/rkos.pdf)  
65. SPARK Overview \- learn.adacore.com, accessed March 28, 2026, [https://learn.adacore.com/courses/intro-to-spark/chapters/01\_Overview.html](https://learn.adacore.com/courses/intro-to-spark/chapters/01_Overview.html)  
66. Assuring Functional Safety in Open Systems of Systems \- kluedo, accessed March 28, 2026, [https://kluedo.ub.rptu.de/files/4422/\_Assuring+Functional+Safety+in+Open+Systems+of+Systems.pdf](https://kluedo.ub.rptu.de/files/4422/_Assuring+Functional+Safety+in+Open+Systems+of+Systems.pdf)  
67. Programming languages and dimensions \- Department of Computer Science and Technology | \- University of Cambridge, accessed March 28, 2026, [https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-391.pdf](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-391.pdf)  
68. Dimensional analysis and types, accessed March 28, 2026, [https://www.johndcook.com/blog/2015/12/01/dimensional-analysis-and-types/](https://www.johndcook.com/blog/2015/12/01/dimensional-analysis-and-types/)  
69. Is F\# suitable for Physics applications? \- Stack Overflow, accessed March 28, 2026, [https://stackoverflow.com/questions/167909/is-f-suitable-for-physics-applications](https://stackoverflow.com/questions/167909/is-f-suitable-for-physics-applications)  
70. Units of measure \- F\# for fun and profit, accessed March 28, 2026, [https://fsharpforfunandprofit.com/posts/units-of-measure/](https://fsharpforfunandprofit.com/posts/units-of-measure/)  
71. Units of Measure \- F\# | Microsoft Learn, accessed March 28, 2026, [https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/units-of-measure](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/units-of-measure)  
72. Physical Units Pass the Generic Test \- AdaCore, accessed March 28, 2026, [https://www.adacore.com/blog/physical-units-pass-the-generic-test](https://www.adacore.com/blog/physical-units-pass-the-generic-test)  
73. Dimensional Analysis in Programming Languages \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/programming/comments/8lwfis/dimensional\_analysis\_in\_programming\_languages/](https://www.reddit.com/r/programming/comments/8lwfis/dimensional_analysis_in_programming_languages/)  
74. SPARK: Formal Verification and Proving Program Correctness in Ada | by Jordan Rowles, accessed March 28, 2026, [https://jordansrowles.medium.com/spark-formal-verification-and-proving-program-correctness-in-ada-3105cc82694d](https://jordansrowles.medium.com/spark-formal-verification-and-proving-program-correctness-in-ada-3105cc82694d)  
75. The SPARK Programming Language \- AdaCore, accessed March 28, 2026, [https://www.adacore.com/languages/spark](https://www.adacore.com/languages/spark)  
76. 8\. Applying SPARK in Practice \- Documentation \- AdaCore, accessed March 28, 2026, [https://docs.adacore.com/spark2014-docs/html/ug/en/usage\_scenarios.html](https://docs.adacore.com/spark2014-docs/html/ug/en/usage_scenarios.html)  
77. Why isn't design by contract more common? : r/ProgrammingLanguages \- Reddit, accessed March 28, 2026, [https://www.reddit.com/r/ProgrammingLanguages/comments/l238o5/why\_isnt\_design\_by\_contract\_more\_common/](https://www.reddit.com/r/ProgrammingLanguages/comments/l238o5/why_isnt_design_by_contract_more_common/)  
78. Is Rust faster than C? \- Hacker News, accessed March 28, 2026, [https://news.ycombinator.com/item?id=46569175](https://news.ycombinator.com/item?id=46569175)  
79. Rust Performance Optimizations Compared to Other Programming Languages \- Medium, accessed March 28, 2026, [https://medium.com/@kaly.salas.7/rust-performance-optimizations-compared-to-other-programming-languages-c2e3685163e2](https://medium.com/@kaly.salas.7/rust-performance-optimizations-compared-to-other-programming-languages-c2e3685163e2)  
80. A Compiler-Driven Approach for Static Dependency Injection in Embedded Software, accessed March 28, 2026, [https://sol.sbc.org.br/index.php/sblp/article/download/36944/36729/](https://sol.sbc.org.br/index.php/sblp/article/download/36944/36729/)  
81. Tighten Rust's Belt: Shrinking Embedded Rust Binaries \- Stanford Sing, accessed March 28, 2026, [https://sing.stanford.edu/site/assets/publications/rust-lctes22.pdf](https://sing.stanford.edu/site/assets/publications/rust-lctes22.pdf)  
82. DO-178C Certification \- Your Complete Verification Journey \- LDRA, accessed March 28, 2026, [https://ldra.com/do-178/](https://ldra.com/do-178/)  
83. SEI CERT C Coding Standard: Rules for Developing Safe, Reliable, and Secure Systems (2016 Edition), accessed March 28, 2026, [https://abougouffa.github.io/awesome-coding-standards/sei-cert-c-2016.pdf](https://abougouffa.github.io/awesome-coding-standards/sei-cert-c-2016.pdf)  
84. From ASIL D to ASIL E: A Unified Framework for Driver-Out Functional Safety, accessed March 28, 2026, [https://www.researchgate.net/publication/395910500\_From\_ASIL\_D\_to\_ASIL\_E\_A\_Unified\_Framework\_for\_Driver-Out\_Functional\_Safety](https://www.researchgate.net/publication/395910500_From_ASIL_D_to_ASIL_E_A_Unified_Framework_for_Driver-Out_Functional_Safety)  
85. Improving Operating System Security, Reliability, and Performance through Intra-Unikernel Isolation, Asynchronous Out-of-kernel \- VTechWorks, accessed March 28, 2026, [https://vtechworks.lib.vt.edu/bitstreams/ae3b79d1-c54b-467d-be64-42d34dcd6b99/download](https://vtechworks.lib.vt.edu/bitstreams/ae3b79d1-c54b-467d-be64-42d34dcd6b99/download)  
86. Key Automotive Functional Safety Challenges: ISO 26262 | Synopsys Blog, accessed March 28, 2026, [https://www.synopsys.com/blogs/chip-design/auto-functional-safety-iso-26262-key-challenges.html](https://www.synopsys.com/blogs/chip-design/auto-functional-safety-iso-26262-key-challenges.html)  
87. ISL-ESRD-TR-14-03, "Evaluation of Guidance for Tools Used to Develop Safety-Related Digital Instrumentation and Control Sof \- Nuclear Regulatory Commission, accessed March 28, 2026, [https://www.nrc.gov/docs/ML1504/ML15043A206.pdf](https://www.nrc.gov/docs/ML1504/ML15043A206.pdf)

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACoAAAAXCAYAAAB9J90oAAABUklEQVR4Xu2WPy8FQRTFb/yJaEQo8QFUCtGSaHwMJYXv8D6AoFDoRKJWqESh1pJQ6XQahYQEEc6xMy93jnlPVmzeFvNLTjJz7tyZm92Z2TVrjmM1wCH0Cb1DkxK7C7ED8RuDxWxatahnAxoP7QVL40+uPQvtun7jaKHxiUXYXnNtz6n0E5j0alXSJTSahmujiyuMT4f2TejzTTx2R2TYhzquz1fBxHnn1aVfoSfQhXhvVuXci5/AASsZzy/GyaOXk5LzyDJ0Ll4cOwQ9Qw8u1mXK8ovlvDrkcicsvQ0WoRn7eXhyud/sQFvi/XehfFrX0Cq0Dm1DwyF2FQcFXqTfl78Wembplohz8JDkfLJk1d7cg26d/yvc6JxoTANtIl7GcxpoE9zsLFI/b62CG5tFjjjvyLVbQ+7gfKgxaPg3oydST+bA4YWrxUXxK1EoFAqF3nwBJstkcNSt7zkAAAAASUVORK5CYII=>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAZCAYAAABD2GxlAAABt0lEQVR4Xu2WvyvFYRTGj9/ERBab2WIRSlLKwqbIpIwyiBRlUP4Euz/BYDEwmqQoC4OJwSZJisQ5nffLex/nvefVRcr91NPtPud533u+99v7g+gf0oOGQwerBs1cGllrrHVWE9QslljzaGbwTF9scpH1ypoJ37tZ16yH98RnJHOBZkQX6ZwW0lyqVkIzafAAC4EX0qe1kHG14HWyrkKtUIpd1haaiExwiWbEJGlmAPwx1h14iNeg+y/ekBNg6kgzO+A/spbBQ7wGBamPoCmMkhb3sAC0k+bkYWLEqwcPyWnwlHWIpvBEOthbqdOkue3IawmeR06DK5TI5AwWTkhz45E3HDyPnN+YIiNTLP9PBQMrN2d4FtZYpJ+MTFsw77EAFK93E/zZ4HvkNNhHiUzO4FRmkGwfSY2PMV+xIJuvWQick9YbsEAfK9sjp8FVSmRaSQvH4MvmeUvp06Mg1XxMToNnrCM0Y+Q0kEn2w6doqCRhIzm5KFgU86AsxJ9A8zuQ/cs76jzco65SZHLvNCmHvDX3slAJsprLXTTKIQ8m5/mPs8FaQDMDucb9GnKyfIVe0ltSlSp/kjdlLX6Qyw11UQAAAABJRU5ErkJggg==>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAZCAYAAAA4/K6pAAAArklEQVR4XmNgIA6IAvF/KM5HkyMKLEVigwwJR+ITBIoMEE0wsBWNTzL4BMTH0AWJBVwMFNpOkeaPSOydSGwMEAvE24E4CknsIhCHQHEhEG9DkoMDYwbUKNoNxAeB2AgqjoxzoGrgAKaIF0kMxH+MxMcLQIqfoonpoPFxAn8GiAEgmiywjIHC6OlmwG0A0WkeZIA0mtgHIA5AE8MJBBgggQiLpg1AzIiiYhSMAloCAE0OKLoItFBVAAAAAElFTkSuQmCC>
\newpage
