# aria-rate-limit

**Version**: 0.12.1
**Category**: middleware, security
**License**: MIT

Token-bucket rate limiter for Aria. Configure maximum tokens, refill rate,
and generate standard rate-limit HTTP headers.

## API Reference

| Function | Description |
|----------|-------------|
| `rate_configure(int64, int64) → int64` | Create limiter with max tokens and refill rate |
| `rate_set_max(int64, int64) → int32` | Update maximum token count |
| `rate_set_rate(int64, int64) → int32` | Update refill rate |
| `rate_check(int64) → int64` | Check if request is allowed (consumes 1 token) |
| `rate_check_cost(int64, int64) → int64` | Check with custom token cost |
| `rate_remaining(int64) → int64` | Get remaining tokens |
| `rate_retry_after(int64) → int64` | Get retry-after delay |
| `rate_headers(int64) → string` | Generate rate-limit HTTP headers |
| `rate_reset(int64) → int32` | Reset token bucket |
| `rate_test_set_tokens(int64, int64) → int32` | Set token count (testing) |

**Type:RateLimit** wrapper available.

## Quick Start

```aria
use "aria_rate_limit.aria".*;

func:main = int32() {
    int64:rl = raw rate_configure(100i64, 10i64);
    int64:allowed = raw rate_check(rl);
    int64:left = raw rate_remaining(rl);
    string:hdrs = raw rate_headers(rl);
    println(hdrs);
    exit(0i32);
};
```

## See Also

- [aria-server](aria-server.md) — HTTP server
- [aria-cors](aria-cors.md) — CORS middleware
