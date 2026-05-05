# npkpkg — Nitpick Package Manager

`npkpkg` manages Nitpick libraries and tools. It can install packages from the
online registry or from local directories.

## Commands

### Update the registry

```bash
npkpkg update
```

Downloads the latest package registry from GitHub. Run this before searching
or installing remote packages. The cache lives at `~/.aria/cache/aria-packages/`.

### Search packages

```bash
npkpkg list --remote              # list all available packages
npkpkg list --remote json         # search by keyword
npkpkg list --remote http         # find HTTP-related packages
```

### Install a package

```bash
npkpkg install aria-json          # install from registry
npkpkg install ./my-package/      # install from local directory
npkpkg install my-package.npkpkg  # install from archive
```

When given a bare name (not a path), `npkpkg` looks up the package in the
cached registry and installs it. Run `npkpkg update` first if the cache
is empty.

### List installed packages

```bash
npkpkg list
```

### Get package info

```bash
npkpkg info aria-json
```

### Remove a package

```bash
npkpkg remove aria-json
```

### Search installed registry

```bash
npkpkg search json
```

### Create a new package

```bash
npkpkg init my-library
```

Creates a package skeleton with `aria-package.toml`, `src/`, and `tests/`.

### Pack a package

```bash
npkpkg pack ./my-library/
```

Creates a `.npkpkg` archive for distribution.

---

## Package Structure

Every Nitpick package has this layout:

```
my-package/
├── aria-package.toml    # metadata and dependencies
├── src/
│   └── my_package.npk  # source files
└── tests/
    └── test_my_package.npk
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
entry = "src/my_package.npk"
```

The `[build]` section is optional. If present, `type` defaults to `"library"`
and `entry` defaults to `""`.

---

## Using packages in your code

After installing a package, use it in your Nitpick source:

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
npkpkg update https://github.com/your-org/your-packages.git
```

The default remote is `https://github.com/alternative-intelligence-cp/aria-packages.git`.
