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

## Package Census (v0.16.8)

| Category | Count |
|----------|-------|
| **Total packages** | **103** |

## Related

- [use_import.md](use_import.md) — importing from packages
- [mod.md](mod.md) — module definitions
