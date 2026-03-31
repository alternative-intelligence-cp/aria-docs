# sys() — Direct Syscalls

## Three Tiers

Aria provides direct syscall access at three safety levels:

| Function | Tier | Syscalls Available | Return Type | TOS Layer |
|----------|------|-------------------|-------------|-----------|
| `sys(CONST, args...)` | Safe | ~55 curated whitelist | `Result<int64>` | Layer 1 |
| `sys!!(CONST, args...)` | Full | All syscalls | `Result<int64>` | Layer 2 |
| `sys!!!(expr, args...)` | Raw | Any int64 expression | `int64` (bare) | Layer 3 |

## Safe Tier — sys()

Curated whitelist of ~55 safe syscalls. Returns `Result<int64>`:

```aria
int64:fd = sys(SYS_OPEN, path, flags, mode) ? -1i64;
```

## Full Tier — sys!!()

All syscalls available, but still returns Result:

```aria
int64:result = sys!!(SYS_MMAP, addr, length, prot, flags, fd, offset) ? -1i64;
```

## Raw Tier — sys!!!()

Accepts any int64 expression as syscall number. Returns bare int64 (no Result wrapper):

```aria
int64:result = sys!!!(231i64, 0i64);   // SYS_EXIT_GROUP
```

## Known Issue

Bare filename string literals in `sys()` calls may fail. Store in a variable first:

```aria
// WRONG: sys(SYS_OPEN, "file.txt", 0i64, 0i64);
// RIGHT:
string:path = "file.txt";
sys(SYS_OPEN, path, 0i64, 0i64);
```

## Related

- [file_io.md](file_io.md) — higher-level file operations
- [control_flow/error_flow.md](../control_flow/error_flow.md) — error handling tiers
