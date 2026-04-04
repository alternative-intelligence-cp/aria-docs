# Aria Reserved Words — Complete List

> Generated from `src/frontend/lexer/lexer.cpp` keyword table (151 keywords)
> v0.13.4 — Authoritative reference

All words listed below are reserved by the Aria compiler and **cannot be used as
variable names, function names, or identifiers** (except `stack` and `gc` which are
contextual — see notes).

---

## Mandatory Functions

```
func        main        failsafe
```

## Control Flow

```
if          else        while       for         loop        till
when        then        end         pick        fall        break
continue    return      pass        fail        exit
```

## Result & Error Handling

```
ok          drop        raw         defaults    Result      ERR
NIL         NULL        unknown
```

## Memory Management

```
wild        wildx       stack*      gc*         move        defer
```

> `*` `stack` and `gc` are **contextual** — valid as variable names outside allocation contexts.

## Module System

```
use         mod         pub         extern      in          as          cfg
```

## Type Declarations

```
struct      enum        trait       impl        derive      opaque
Type        Rules       func        macro
```

## Type Modifiers

```
const       fixed       limit       dyn         obj         any
```

## Async & Concurrency

```
async       await       catch
```

## Memory Ordering

```
relaxed     acquire     release     acq_rel     seq_cst
```

## Borrow & Safety

```
is          requires    ensures     invariant   result
```

## Compile-Time

```
comptime    inline      noinline    prove       assert_static
```

## Integer Types

```
int1        int2        int4        int8        int16       int32
int64       int128      int256      int512      int1024     int2048     int4096
uint1       uint2       uint4       uint8       uint16      uint32
uint64      uint128     uint256     uint512     uint1024    uint2048    uint4096
```

## Other Numeric Types

```
tbb8        tbb16       tbb32       tbb64
flt32       flt64       flt128      flt256      flt512
tfp32       tfp64
fix256
frac8       frac16      frac32      frac64
```

## Boolean & String

```
bool        true        false       string
```

## Ternary & Nonary

```
trit        tryte       nit         nyte
```

## Vector, Matrix & Tensor

```
vec2        vec3        vec9
matrix      tmatrix
tensor      ttensor
```

## Collection & I/O Types

```
array       binary      buffer      stream      pipe        process
```

## Debug & Special

```
debug       apop        apush       apeek       astack      acap
asize       afits       atype       ahash       ahset       ahget
ahcount     ahsize      ahfits      ahtype
```

---

## Not Reserved (Common Misconceptions)

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

## Total Count: 151 reserved keywords
