# main / failsafe / exit() — Canonical Reference
# This is the source of truth for how the two program endpoints work.

## Design Intent

Every Aria executable has exactly two code-path endpoints:

  main       — normal program entry and exit
  failsafe   — abnormal/error exit (runtime errors, !!!, constraint violations)

Both terminate the process via `exit(code)`. Neither uses `pass()/fail()`.
pass/fail are the Result<T> return system for regular functions — they have
no business in program endpoints.


## Signatures

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


## Why tbb32 for failsafe's error code?

- The runtime may invoke failsafe for errors that don't have a numeric code
- tbb32 has three states: positive integers, negative integers, and ERR
- ERR represents "unknown/unclassifiable error" — a state int32 cannot express
- This keeps the safety system coherent all the way to the final exit call
- User-invoked `!!! code` can pass a known tbb32 value
- Runtime-invoked failsafe passes ERR when the error category is not known


## exit() rules

- Only valid inside `main` and `failsafe`
- Takes exactly one int32 argument (the process exit code)
- In main: any value is valid (`exit(0)` for success, `exit(1)` for failure, etc.)
- In failsafe: value MUST be > 0 (failsafe is an error path — exit(0) is wrong)
- exit() terminates the process immediately — it is noreturn
- At least one exit() call must be reachable in main and failsafe


## What NOT to use in main/failsafe

- `pass(value)` — this is for regular functions, creates Result<T>
- `fail(code)` — this is for regular functions, creates error Result
- `return` — Aria doesn't use this keyword


## Invocation patterns

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


## Compiler enforcement summary

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
