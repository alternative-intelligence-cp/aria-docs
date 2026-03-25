# Aria Packages Reference

This directory documents the official Aria packages available in `aria-packages`.

## v0.2.8 Packages (Game Development Track)

| Package | Description |
|---------|-------------|
| [aria-raylib](aria-raylib.md) | raylib 2D/audio/input bindings — the core game dev library |
| [aria-tetris](aria-tetris.md) | Full Tetris clone — reference game implementation |
| [aria-gml](aria-gml.md) | GameMaker Language compatibility layer over raylib |
| [aria-opengl](aria-opengl.md) | OpenGL 3.3 Core — custom shaders and GPU-accelerated 3D |

## All Packages

See individual files in this directory for full documentation of each package.

## Quick Start

```bash
# Clone the packages repo
git clone https://github.com/aria-lang/aria-packages
cd aria-packages/packages/aria-gml/shim
make

# Compile a program using aria-gml
ariac my_game.aria -o my_game \
  -L packages/aria-gml/shim \
  -laria_gml_shim -lraylib -lm
```
