# aria-docs

Documentation, man pages, and examples for the [Aria programming language](https://github.com/alternative-intelligence-cp/aria) (v0.2.15).

## Contents

- **guide/** — Programming guide (362 files): types, functions, control flow, memory model, modules, advanced features, standard library
- **man/** — Man page sources and build scripts (`man ariac`, `man aria-pkg`, etc.)
- **examples/** — Example programs demonstrating Aria features (69 .aria files)
- **reference/** — Language reference and compiler architecture documentation
- **specs/** — Language specification (`aria_specs.txt` — 7,200-line spec covering the 3-layer safety system)
- **packages/** — Package-specific guides (raylib, GML, Tetris, OpenGL)

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
