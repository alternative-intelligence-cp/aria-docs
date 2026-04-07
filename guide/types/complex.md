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

Standard complex arithmetic via stdlib functions:

```aria
use "complex.aria".*;

complex<flt64>:a = raw complex_new(1.0, 2.0);
complex<flt64>:b = raw complex_new(3.0, 4.0);
complex<flt64>:sum = raw complex_add(a, b);       // {4.0, 6.0}
complex<flt64>:prod = raw complex_mul(a, b);       // {-5.0, 10.0}
complex<flt64>:conj = raw complex_conjugate(a);    // {1.0, -2.0}
```

Functions: `complex_new`, `complex_add`, `complex_sub`, `complex_mul`, `complex_div`,
`complex_conjugate`, `complex_magnitude`, `complex_from_real`, `complex_from_imag`,
`complex_is_err`, `complex_err`.

## ERR Propagation

If either component is ERR (when using TBB component types), the entire complex
number is tainted.

## SIMD

Interleaved layout for SIMD: `[real₀, imag₀, real₁, imag₁, ...]`

## Status

Implemented in stdlib (`complex.aria`, 510 lines). Generic over all `Numeric` trait
types. 10+ test files covering basic operations, generics, and turbofish syntax.

## Related

- [flt.md](flt.md) — float component types
- [fix256.md](fix256.md) — deterministic component type
- [tbb.md](tbb.md) — error-propagating component type
