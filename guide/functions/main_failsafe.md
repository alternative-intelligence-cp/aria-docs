# main and failsafe — Mandatory Functions

Every Aria program **must** define both `main` and `failsafe`.

## main

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

## failsafe

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

## Required in Every Program

Both functions must be present. A program without either will not compile.

## Paren Syntax

`exit` supports both keyword and paren syntax:

```aria
exit 0;      // keyword form
exit(0);     // paren form
```
