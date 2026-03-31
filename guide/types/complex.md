# Complex Numbers — `complex<T>`

## Overview

Generic complex number type: `complex<T>` where T is the component type.

```
complex<T> = { T:real, T:imag }
```

Common instantiations: `complex<flt64>`, `complex<fix256>`, `complex<tbb64>`.

## Declaration

```aria
complex<flt64>:z = { real = 3.0, imag = 4.0 };  // 3 + 4i
```

## Arithmetic

Standard complex arithmetic: `+`, `-`, `*`, `/` with proper mathematical formulas.

```aria
complex<flt64>:a = { real = 1.0, imag = 2.0 };
complex<flt64>:b = { real = 3.0, imag = 4.0 };
complex<flt64>:sum = (a + b);  // {4.0, 6.0}
```

- `conjugate()` — implemented
- `magnitude()` — future
- `phase()` — future

## ERR Propagation

If either component is ERR (when using TBB component types), the entire complex
number is tainted.

## SIMD

Interleaved layout for SIMD: `[real₀, imag₀, real₁, imag₁, ...]`

## Status

**⚠️ Not yet implemented in the compiler.** Consider the `aria-entangled` package.

## Related

- [flt.md](flt.md) — float component types
- [fix256.md](fix256.md) — deterministic component type
- [tbb.md](tbb.md) — error-propagating component type
