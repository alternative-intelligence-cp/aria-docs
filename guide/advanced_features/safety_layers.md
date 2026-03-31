# Safety Layers — TOS (Terms of Safety)

## Overview

Aria's safety model uses explicit layers. Each layer grants more power and less safety:

| Layer | Name | Access | Safety |
|-------|------|--------|--------|
| 0 | Safe | Default Aria code | Full — Result<T>, bounds checks, type safety |
| 1 | Controlled | `sys()` safe syscalls | Curated syscall whitelist |
| 2 | Supervised | `sys!!()` all syscalls | All syscalls, still returns Result |
| 3 | Raw | `sys!!!()`, `wild`, `wildx` | No safety net — you own it |

## TOS Safety Vocabulary

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

## Philosophy

Every safety bypass is **visible in code**. There are no hidden undefined behaviors.
When reading Aria code, the `raw`, `wild`, `drop`, `sys!!!` keywords immediately
identify where safety guarantees are intentionally relaxed.

## Related

- [control_flow/error_flow.md](../control_flow/error_flow.md) — error model overview
- [types/result.md](../types/result.md) — Result<T> safety
- [io_system/sys.md](../io_system/sys.md) — syscall tiers
