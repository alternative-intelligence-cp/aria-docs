# Comptime Limitations

Comptime is the constant evaluator, not a sandboxed runtime. The list below
summarises what is **not** allowed in any `comptime(...)`, `comptime { ... }`,
or `comptime func:` body. Violations are reported at the comptime call site.

## Disallowed Operations

| Category | What is forbidden | Why |
|---|---|---|
| **I/O** | `print`, `println`, `read`, file/socket APIs | No host access at compile time |
| **GC allocation** | `gc T:x = ...`, anything that triggers the runtime GC | GC is a runtime service |
| **Wild allocation** | `wild T:x = ...`, `wildx T:x = ...`, `free` | Manual memory is runtime-only |
| **Extern calls** | Any `extern` C function | Determinism: no host symbols at build time |
| **Async** | `async func:`, `await`, async constructs | No scheduler at comptime |
| **Dynamic dispatch** | Function pointers stored in runtime values | Targets must be statically known |
| **Threading** | `Signal.register`, locks, mutexes | No threads at comptime |
| **Time / RNG** | `time_monotonic_ns`, `rand`, environment access | Determinism |
| **Pinning** | `#x` pinning for runtime borrow checking | Pin state is runtime-only |

## Bounded Operations

These are allowed but capped:

- **Recursion depth & step count** — bounded by the CTFE budget.
- **Loop iterations** — bounded by the same budget.
- **Memoization cache** — per-compilation-unit, bounded.

Exceeding any cap is a hard error, not a silent truncation.

## Allowed (For Reference)

- All pure arithmetic, comparisons, boolean logic
- `if`, `pick`, `loop(start, end, step)`
- Mutable locals (CTFE-private)
- `stack T:x = ...` (CTFE-private stack values)
- Struct construction, struct update syntax, field reads
- Array literals, fixed-size indexing
- All `@`-intrinsics (`@typeInfo`, `@sizeof`, `@fieldType`, ...)
- Calls into other `comptime func:` and pure regular `func:`

## "Why isn't X allowed?"

The single guiding rule is **determinism**: a Nitpick build must produce
identical output for identical input on any host. Anything that could
observe or perturb host state is therefore comptime-forbidden.

If you need behaviour that is forbidden at comptime, defer it to runtime
or generate the runtime code with a macro.
