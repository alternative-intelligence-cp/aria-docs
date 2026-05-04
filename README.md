# aria-docs

<p align="center">
	<img src="assets/nitpick_logo.png" alt="Nitpick logo: raccoon holding a magnifying glass" width="220">
</p>

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

> 🚧 **Rebrand in progress:** Aria is becoming **Nitpick**. This documentation
> repo still uses Aria names while the migration is underway. Existing docs,
> examples, man pages, and history are being preserved; source and file-extension
> renames will happen in later coordinated slices.

Documentation, man pages, and examples for the [Aria programming language](https://github.com/alternative-intelligence-cp/aria) (v0.17.3).

## Contents

- **guide/** — Programming guide (362 files): types, functions, control flow, memory model, modules, advanced features, standard library
- **guide/aria-pkg.md** — Package manager usage guide
- **man/** — Man page sources and build scripts (`man ariac`, `man aria-pkg`, etc.)
- **examples/** — Example programs demonstrating Aria features (69 .aria files)
- **reference/** — Language reference and compiler architecture documentation
- **specs/** — Language specification (`aria_specs.txt` — 7,200-line spec covering the 3-layer safety system)
- **packages/** — Package-specific guides (raylib, GML, Tetris, OpenGL)

## Installation

See **[INSTALL.md](https://github.com/alternative-intelligence-cp/aria/blob/main/INSTALL.md)** in the main aria repo for all installation methods (install script, .deb, .rpm, build from source).

## Getting Started

See [GETTING_STARTED.md](GETTING_STARTED.md) for setup instructions.

## Man Pages

```bash
cd man/
make          # Build man pages
sudo make install  # Install to /usr/share/man/
```

## Examples

Each example can be compiled with:
```bash
ariac examples/<name>.aria -o <name>
./<name>
```

Some examples have subdirectories with their own build scripts.

## License

AGPL-3.0 — see [LICENSE.md](LICENSE.md)
