# aria-pkg — Aria Package Manager

`aria-pkg` manages Aria libraries and tools. It can install packages from the
online registry or from local directories.

## Commands

### Update the registry

```bash
aria-pkg update
```

Downloads the latest package registry from GitHub. Run this before searching
or installing remote packages. The cache lives at `~/.aria/cache/aria-packages/`.

### Search packages

```bash
aria-pkg list --remote              # list all available packages
aria-pkg list --remote json         # search by keyword
aria-pkg list --remote http         # find HTTP-related packages
```

### Install a package

```bash
aria-pkg install aria-json          # install from registry
aria-pkg install ./my-package/      # install from local directory
aria-pkg install my-package.aria-pkg  # install from archive
```

When given a bare name (not a path), `aria-pkg` looks up the package in the
cached registry and installs it. Run `aria-pkg update` first if the cache
is empty.

### List installed packages

```bash
aria-pkg list
```

### Get package info

```bash
aria-pkg info aria-json
```

### Remove a package

```bash
aria-pkg remove aria-json
```

### Search installed registry

```bash
aria-pkg search json
```

### Create a new package

```bash
aria-pkg init my-library
```

Creates a package skeleton with `aria-package.toml`, `src/`, and `tests/`.

### Pack a package

```bash
aria-pkg pack ./my-library/
```

Creates a `.aria-pkg` archive for distribution.

---

## Package Structure

Every Aria package has this layout:

```
my-package/
├── aria-package.toml    # metadata and dependencies
├── src/
│   └── my_package.aria  # source files
└── tests/
    └── test_my_package.aria
```

### aria-package.toml

```toml
[package]
name        = "my-package"
version     = "0.1.0"
description = "A useful library"
license     = "MIT"

[dependencies]
aria-json = "0.2.0"

[build]
type  = "library"           # "library" or "executable"
entry = "src/my_package.aria"
```

The `[build]` section is optional. If present, `type` defaults to `"library"`
and `entry` defaults to `""`.

---

## Using packages in your code

After installing a package, use it in your Aria source:

```aria
use "aria-json".*;

func:main = int32() {
    // package functions are now available
    exit(0);
};
func:failsafe = int32(tbb32:err) { exit(1); };
```

---

## Directories

| Path | Purpose |
|------|---------|
| `~/.aria/packages/installed/` | Installed packages |
| `~/.aria/packages/registry/` | Local registry index |
| `~/.aria/cache/aria-packages/` | Cached remote registry |

---

## Custom registry URL

```bash
aria-pkg update https://github.com/your-org/your-packages.git
```

The default remote is `https://github.com/alternative-intelligence-cp/aria-packages.git`.
