# Packages

## Package Structure

Aria packages follow a standard layout managed by `aria-make`:

```
my-package/
├── src/
│   ├── main.aria
│   └── lib.aria
├── test/
│   └── test_main.aria
├── aria-make.toml
└── README.md
```

## aria-make.toml

```toml
[package]
name = "my-package"
version = "0.1.0"

[dependencies]
aria-string = "0.1.0"
```

## Building

```bash
aria-make build        # compile
aria-make test         # run tests
aria-make run          # build and run
```

## Package Registry

Packages are hosted at `aria-packages` and `aria-packages-apt`:
- Source packages: `REPOS/aria-packages/packages/`
- APT packages: `REPOS/aria-packages-apt/`

## Package Census (v0.11.3)

| Category | Count | % |
|----------|-------|---|
| Pure Aria | 17 | 24.6% |
| aria-libc backed | 9 | 13.0% |
| Direct extern FFI | 43 | 62.3% |
| **Total (with TOML)** | **69** | |
| Legacy/scaffold (no TOML) | 27 | |
| **Grand total** | **96** | |

### New in v0.11.3

| Package | Category | Description |
|---------|----------|-------------|
| `aria-bitset` | Pure Aria (aria-libc mem) | Fixed-size bit sets with union, intersect, complement |
| `aria-result` | Pure Aria (aria-libc mem) | Extended Result combinators — unwrap, map_or, and/or |
| `aria-deque` | Pure Aria (aria-libc mem) | Double-ended queue with O(1) push/pop at both ends |

## Related

- [use_import.md](use_import.md) — importing from packages
- [mod.md](mod.md) — module definitions
