# sys() — Direct System Calls

**Category**: Standard Library → System Calls  
**Syntax**: `sys(CONST, args...)`, `sys!!(CONST, args...)`, `sys!!!(expr, args...)`  
**Purpose**: Direct Linux kernel syscall invocation without libc

---

## Overview

`sys()` is a compiler builtin that emits inline `syscall` instructions for direct kernel communication. Three tiers provide escalating access, integrated with Aria's TOS safety model.

---

## Three Tiers

| Tier | Syntax | Syscall Set | Return Type | Use Case |
|------|--------|-------------|-------------|----------|
| Safe | `sys(...)` | ~55 curated | `Result<int64>` | Library code, general I/O |
| Full | `sys!!(...)` | All ~370+ | `Result<int64>` | Process control, advanced ops |
| Raw | `sys!!!(...)` | All ~370+ | `int64` | Kernel modules, bare metal |

**Design principle:** Harder to type = more dangerous. `sys` is safe. `sys!!` requires deliberate escalation. `sys!!!` is raw access with no safety net.

---

## Setup

```aria
use "sys.aria".*;  // Import all syscall constants (SYS_READ, SYS_WRITE, etc.)
```

The `sys.aria` stdlib module provides 373 named constants auto-generated from Linux kernel headers.

---

## Safe Tier — `sys()`

The safe tier only accepts syscalls from a curated whitelist of ~55 operations that cannot compromise process integrity when used correctly.

```aria
// Write to stdout
Result<int64>:written = sys(WRITE, 1i64, "Hello, kernel!\n", 15i64);
int64:bytes = written ? 0i64;

// Get process ID
Result<int64>:pid_r = sys(GETPID);
int64:pid = pid_r ? 0i64;

// Read from file descriptor
Result<int64>:bytes_read = sys(READ, fd, buffer, 1024i64);
```

### What's in the safe set?

File I/O (`READ`, `WRITE`, `OPEN`, `CLOSE`, `LSEEK`, `OPENAT`, ...), memory (`MMAP`, `MUNMAP`, `MPROTECT`), directories (`GETCWD`, `MKDIR`, `RMDIR`, `RENAME`), process info (`GETPID`, `GETUID`, `GETTID`), time (`CLOCK_GETTIME`, `NANOSLEEP`), networking (`SOCKET`, `BIND`, `LISTEN`, `ACCEPT4`, `CONNECT`), polling (`EPOLL_CREATE1`, `EPOLL_CTL`, `POLL`), pipes (`PIPE2`, `DUP`, `DUP2`), and misc (`IOCTL`, `FCNTL`, `GETRANDOM`).

### Compile-time enforcement

```aria
// This will NOT compile:
Result<int64>:bad = sys(FORK);
// error: 'FORK' is not in the safe syscall set.
// Use sys!!('FORK', ...) for full access.

// Variables are also rejected — no sneaking past the whitelist:
int64:nr = 57i64;
Result<int64>:bad2 = sys(nr, 0i64);
// error: sys() safe tier requires a named constant, not a variable/expression.
```

---

## Full Tier — `sys!!()`

The full tier allows all Linux syscalls but still wraps the return in `Result<int64>`.

```aria
// Fork a process (not in safe set)
Result<int64>:child = sys!!(FORK);
int64:pid = child ? -1i64;

// Send a signal
Result<int64>:sig_r = sys!!(KILL, target_pid, 9i64);

// Execute a program
Result<int64>:exec_r = sys!!(EXECVE, "/bin/ls", argv_ptr, envp_ptr);
```

The first argument must still be a named constant — you can't use a variable expression with `sys!!`.

---

## Raw Tier — `sys!!!()`

The raw tier returns a bare `int64` with no Result wrapping. You are responsible for error checking.

```aria
// Raw write — returns bytes written or negative errno
int64:result = sys!!!(1i64, 1i64, "raw\n", 4i64);

// Check for error manually
if (result < 0i64) {
    // Handle error: result is -errno
}
```

The first argument can be any `int64` expression — literal, variable, or computed value. No compile-time validation of the syscall number.

---

## Result Wrapping

For `sys()` and `sys!!()`, the kernel return value is wrapped:

- **Non-negative** (>= 0) → `pass(value)` — success
- **Negative** (< 0) → `fail(abs(value))` — errno as error code

This means standard Aria error handling works:

```aria
Result<int64>:fd = sys(OPEN, "/etc/hostname", 0i64, 0i64);
int64:file = fd ? -1i64;  // unwrap with fallback

if (file < 0i64) {
    drop(println("Failed to open file"));
}
```

---

## Building Wrappers

The real power of `sys()` is building pure-Aria system wrappers with no C dependency:

```aria
use "sys.aria".*;

// Pure-Aria write wrapper
func:sys_write = Result<int64>(int64:fd, string:data, int64:len) {
    Result<int64>:r = sys(WRITE, fd, data, len);
    int64:val = r ? -1i64;
    if (val < 0i64) {
        fail(1t8);
    }
    pass(val);
};

// Pure-Aria getpid wrapper
func:sys_getpid = Result<int64>() {
    Result<int64>:r = sys(GETPID);
    int64:val = r ? -1i64;
    pass(val);
};
```

These wrappers are the building blocks for porting `aria-libc` from C shims to pure Aria.

---

## x86-64 Syscall ABI

The compiler emits inline `syscall` instructions using the standard x86-64 Linux calling convention:

| Register | Purpose |
|----------|---------|
| `rax` | Syscall number |
| `rdi` | Argument 1 |
| `rsi` | Argument 2 |
| `rdx` | Argument 3 |
| `r10` | Argument 4 |
| `r8` | Argument 5 |
| `r9` | Argument 6 |
| `rax` (return) | Result (negative = -errno) |

The `rcx` and `r11` registers are clobbered by the kernel.

---

## TOS Safety Integration

`sys!!` and `sys!!!` are TOS escalation constructs, visible in `aria-safety` audit output alongside `wild`, `raw()`, and `?!`. This makes unsafe syscall usage:

1. **Visible in code review** — grep for `sys!!` or `sys!!!`
2. **Auditable** — `aria-safety` flags full/raw tier usage
3. **Intentional** — you cannot accidentally use the full tier

---

## Limitations

- **x86-64 Linux only** — AArch64 support planned for v0.4.1+
- **Maximum 6 arguments** after the syscall constant (hardware limitation)
- **No struct passing** — syscall args must be integers, strings, or pointers
- **Constants are compiler builtins** — `WRITE`, `READ`, etc. are recognized by the compiler, not normal variables

---

## See Also

- [process_management.md](process_management.md) — Higher-level process operations
- [exec.md](exec.md) — `exec()` command execution
- [fork.md](fork.md) — `fork()` process forking
- `stdlib/sys.aria` — Syscall constant definitions
- `scripts/gen-sys-constants.sh` — Constant generator script
