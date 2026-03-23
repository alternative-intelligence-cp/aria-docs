# aria-docs

Documentation, man pages, and examples for the [Aria programming language](https://github.com/ailp/aria).

## Contents

- **guide/** — Language guide, specs, API reference, design documents
- **man/** — Man page sources and build scripts (`man ariac`, `man aria-pkg`, etc.)
- **examples/** — Example programs demonstrating Aria features
- **reference/** — Language reference (forthcoming)
- **specs/** — Formal language specifications (forthcoming)

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
