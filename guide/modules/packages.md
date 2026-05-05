# Packages

## Package Structure

Nitpick packages follow a standard layout managed by `nitpick-build`:

```
my-package/
├── src/
│   ├── main.npk
│   └── lib.npk
├── test/
│   └── test_main.npk
├── nitpick-build.toml
└── README.md
```

## nitpick-build.toml

```toml
[package]
name = "my-package"
version = "0.1.0"

[dependencies]
aria-string = "0.1.0"
```

## Building

```bash
nitpick-build build        # compile
nitpick-build test         # run tests
nitpick-build run          # build and run
```

## Package Registry

Packages are hosted at `aria-packages` and `aria-packages-apt`:
- Source packages: `REPOS/aria-packages/packages/`
- APT packages: `REPOS/aria-packages-apt/`

## Package Census (v0.16.8)

| Category | Count |
|----------|-------|
| **Total packages** | **103** |

## Related

- [use_import.md](use_import.md) — importing from packages
- [mod.md](mod.md) — module definitions
