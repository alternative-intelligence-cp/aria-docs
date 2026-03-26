# Contributing to Aria Documentation

Thank you for helping improve Aria's documentation!

## Getting Started

1. Fork the repository
2. Clone: `git clone https://github.com/<your-username>/aria-docs.git`
3. Create a branch: `git checkout -b docs/your-topic`
4. Make your changes
5. Push and open a Pull Request

## Repository Structure

- **guide/** — Programming guide (types, functions, control flow, memory model, etc.)
- **reference/** — Compiler architecture and ecosystem reference
- **examples/** — Example .aria programs
- **man/** — Man page sources and build system
- **specs/** — Language specification
- **packages/** — Package-specific guides

## Writing Guidelines

### Code Examples

Use correct Aria syntax in all code blocks:

```aria
func:example = int32(int32:x) {
    pass(x * 2);
};

failsafe {
    pass(1);
};
```

Key syntax rules:
- Functions: `func:name = ReturnType(params) { ... };`
- Variables: `Type:name = value;`
- Return values: `pass(value)` (success) / `fail(value)` (error)
- Loops: `till(limit, step) { ... }` with `$` as iterator variable
- Imports: `use module_name;`

### Unimplemented Features

If documenting a feature that is part of the language specification but not yet implemented in the compiler, add a warning banner at the top:

```markdown
> **⚠️ DESIGN DOCUMENT — NOT YET IMPLEMENTED IN COMPILER**
> This feature is part of the Aria language specification but is not yet available.
```

## Building Man Pages

```bash
cd man/
make          # Build all man pages
make clean    # Clean build artifacts
```

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
