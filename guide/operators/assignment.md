# Assignment Operator

## Basic Assignment

```aria
int32:x = 42;            // typed declaration with assignment
x = 100;                  // reassignment
x := 200;                // type-inferred declaration (infers int32)
```

## Discard Assignment

```aria
_ = some_function();      // discard Result — acknowledges unused return
```

This is one way to handle nodiscard Results. See also `drop` and `raw`.

## Compound Assignment

See [arithmetic.md](arithmetic.md) for `+=`, `-=`, `*=`, `/=`, `%=`.
See [bitwise.md](bitwise.md) for `<<=`, `>>=`.

## Fixed Values

```aria
fixed int32:MAX = 100;    // Aria's const (use 'fixed', not 'const')
```

`const` is reserved for extern blocks. Use `fixed` for Aria constants.

**Known issue:** `fixed` with arithmetic expressions may evaluate to 0. Compute in a
variable first, then assign to `fixed`.
