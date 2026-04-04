# aria-retry

**Version**: 0.12.3
**Category**: resilience, networking
**License**: MIT

Retry with exponential backoff for Aria. Pure Aria implementation — tracks
attempt count, computes delays as `base_ms * 2^attempt` capped at `cap_ms`,
and manages success/failure/exhaustion state.

## Features

- Configurable max attempts, base delay, and delay cap
- Exponential backoff: `base_ms * 2^attempt`
- Automatic exhaustion detection
- Success/failure state tracking
- Delay calculation without actual sleeping (caller controls timing)

## API Reference

| Function | Description |
|----------|-------------|
| `retry_create(int64, int64, int64) → int64` | Create retry handle (max, base_ms, cap_ms) |
| `retry_next_delay(int64) → string` | Calculate delay for current attempt (ms) |
| `retry_record_fail(int64) → int32` | Record failure, advance attempt |
| `retry_record_ok(int64) → int32` | Record success |
| `retry_should_retry(int64) → int64` | Can retry? (1/0) |
| `retry_succeeded(int64) → int64` | Did it succeed? (1/0) |
| `retry_exhausted(int64) → int64` | All attempts used? (1/0) |
| `retry_get_attempt(int64) → string` | Current attempt number |
| `retry_attempts_used(int64) → string` | Total attempts used |

**Type:Retry** wrapper with methods: `init`, `delay`, `rec_fail`, `rec_ok`, `can_retry`, `is_ok`, `is_done`, `attempt`, `used`.

## Quick Start

```aria
use "aria_retry.aria".*;

func:main = int32() {
    int64:r = raw retry_create(3i64, 100i64, 5000i64);
    // Attempt loop
    int64:sr = raw retry_should_retry(r);
    // First attempt: delay = 100ms (100 * 2^0)
    string:d = raw retry_next_delay(r);
    // Simulate failure
    int32:_f = raw retry_record_fail(r);
    // Second attempt: delay = 200ms (100 * 2^1)
    string:d2 = raw retry_next_delay(r);
    // Simulate success
    int32:_ok = raw retry_record_ok(r);
    int64:done = raw retry_succeeded(r);
    println(string_from_int(done));
    exit(0i32);
};
```

## See Also

- [aria-lru](aria-lru.md) — LRU cache
- [aria-glob](aria-glob.md) — Glob pattern matching
- [aria-rate-limit](aria-rate-limit.md) — Rate limiting
