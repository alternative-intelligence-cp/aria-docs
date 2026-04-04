# aria-glob

**Version**: 0.12.3
**Category**: filesystem, pattern-matching
**License**: MIT

Glob pattern matching for file paths. Pure Aria implementation — supports
`*` (any non-/ chars), `**` (recursive, crosses /), `?` (single non-/ char),
and literal matching via a backtracking algorithm.

## Features

- `*` wildcard: matches any characters except `/`
- `**` recursive wildcard: matches any characters including `/`
- `?` wildcard: matches exactly one non-`/` character
- Pattern metadata: wildcard detection, recursive check, segment count
- Two-bookmark backtracking for complex patterns like `**/*.aria`

## API Reference

| Function | Description |
|----------|-------------|
| `glob_create(string) → int64` | Create glob handle with pattern |
| `glob_match(int64, string) → int64` | Match path against pattern (1/0) |
| `glob_pattern(int64) → string` | Get stored pattern |
| `glob_is_recursive(int64) → int64` | Pattern contains `**` (1/0) |
| `glob_has_wildcard(int64) → int64` | Pattern has `*`, `?` wildcards (1/0) |
| `glob_segment_count(int64) → string` | Number of `/`-separated segments |

**Type:Glob** wrapper with methods: `init`, `match_path`, `pat`, `is_rec`, `has_wild`, `segs`.

## Quick Start

```aria
use "aria_glob.aria".*;

func:main = int32() {
    int64:g = raw glob_create("**/*.aria");
    int64:m1 = raw glob_match(g, "src/pkg/main.aria");
    int64:m2 = raw glob_match(g, "test.aria");
    int64:m3 = raw glob_match(g, "readme.md");
    // m1=1, m2=1, m3=0
    int64:rec = raw glob_is_recursive(g);
    println(string_from_int(rec));
    exit(0i32);
};
```

## See Also

- [aria-lru](aria-lru.md) — LRU cache
- [aria-retry](aria-retry.md) — Retry with backoff
