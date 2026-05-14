# main and failsafe — Mandatory Functions

Every Nitpick program **must** define both `main` and `failsafe`.

## main

Program entry point. Must call `exit()` (not `pass`/`fail`):

```aria
func:main = int32() {
    println("Hello, world!");
    exit 0;
};
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
};
```

- Signature: `func:failsafe = int32(tbb32:err)` — receives error code
- Must call `exit(code)` where code > 0
- `pass`/`fail` are **not valid** in failsafe
- Called automatically on emphatic unwrap (`?!`) failure
- Called by `!!! err;` shorthand
- This is Nitpick's last line of defense — it must always exit

## Required in Every Program

Both functions must be present. A program without either will not compile.

## Paren Syntax

`exit` supports both keyword and paren syntax:

```aria
exit 0;      // keyword form
exit(0);     // paren form
```

## Note: OS-level exit-code truncation

Nitpick passes the full `int32` you give to `exit` straight to the operating
system, but on Unix-like systems the parent process (your shell, a test
runner, `wait()`/`waitpid()`, etc.) only observes the **low 8 bits** of the
exit status. This is the POSIX `WEXITSTATUS` contract — it is the same in C,
C++, Rust, Go, Python, Java, and every other language that exits via libc.

Concretely, `exit 1234` will be reported as `1234 & 0xff == 210` by the
shell. This is not a Nitpick bug; it is a property of the Unix process API.

If you need to communicate a value larger than 255 from a program, write it
to stdout and have the consumer parse it; reserve the exit code for a
small, well-known status (`0` = success, nonzero = error class).
