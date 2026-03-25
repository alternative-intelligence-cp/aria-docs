# aria-tetris

**Version**: 0.2.8
**Category**: games, gamedev, examples
**License**: Apache-2.0 WITH Runtime-Library-Exception

A complete Tetris clone written entirely in Aria. Serves as a reference
implementation demonstrating real-world use of the Aria language for game
development — game state, rendering, audio, input, persistence, and animation
all implemented in pure Aria source.

## Features

- **7-bag randomizer** — fair piece distribution, no long droughts
- **Ghost piece** — shows where the falling piece will land
- **Hold piece** — swap current piece to hold slot (C / B-button)
- **Hard drop** — instant placement with 2× score bonus (Space / Y-button)
- **Soft drop** — faster fall with 1× score bonus
- **High score persistence** — saved to `aria_tetris_best.txt` (survives restarts)
- **Gamepad support** — full D-pad + face-button mapping (xinput-compatible)
- **Procedural sound effects** — 7 synthesized tones, no WAV files needed
- **Line-clear flash animation** — 18-frame white blink before removal
- **Level progression** — level increases every 10 lines, gravity accelerates
- **Title / Game Over / Pause screens** — complete game flow

## Controls

| Action | Keyboard | Gamepad |
|--------|----------|---------|
| Move left/right | ← → | D-pad L/R |
| Rotate CW | ↑ or X | A-btn or D-pad Up |
| Rotate CCW | Z | X-btn |
| Soft drop | ↓ (hold) | D-pad Down (hold) |
| Hard drop | Space | Y-btn |
| Hold piece | C | B-btn |
| Pause / Resume | P | Start |
| Start game | Enter | Start |

## Dependencies

- `aria-raylib` ≥ 0.3.0
- raylib system library
- C compiler (to build shim)

## Build

```bash
cd packages/aria-raylib/shim
cc -O2 -shared -fPIC -Wall -o libaria_raylib_shim.so \
   aria_raylib_shim.c $(pkg-config --cflags --libs raylib) -lm

cd ../../aria-tetris
/path/to/ariac src/aria_tetris.aria -o aria_tetris \
  -L ../aria-raylib/shim \
  -laria_raylib_shim -lraylib -lm
```

## Run

```bash
export LD_LIBRARY_PATH=../aria-raylib/shim
./aria_tetris
```

## Scoring

| Lines cleared | Points |
|--------------|--------|
| 1 | 100 × level |
| 2 | 300 × level |
| 3 | 500 × level |
| 4 (Tetris!) | 800 × level |
| Soft drop | 1 per row |
| Hard drop | 2 per row |

## Sound Design

All sounds are generated procedurally at startup using `aria_rl_gen_beep()` —
no audio files are bundled with the package.

| Sound | Frequency | Duration | Wave | Event |
|-------|-----------|----------|------|-------|
| Move | 220 Hz | 40 ms | Square | Piece moved |
| Rotate | 330 Hz | 40 ms | Square | Piece rotated |
| Lock | 150 Hz | 70 ms | Square | Piece locked |
| Line clear | 440 Hz | 120 ms | Square | 1–3 lines |
| Tetris | 880 Hz | 200 ms | Triangle | 4 lines |
| Level up | 660 Hz | 160 ms | Sine | Level increase |
| Game over | 110 Hz | 450 ms | Square | Top-out |

## Architecture

The entire game is a single Aria source file (`src/aria_tetris.aria`, ~930 lines).

### State machine

```
0 = Title screen → 1 (Enter/Start)
1 = Playing      → 3 (P/Start), 2 (top-out)
2 = Game over    → 0 (Enter/Start)
3 = Paused       → 1 (P/Start)
```

### Line-clear animation

When a piece locks and fills complete rows:
1. `flash_timer = 18` (18 frames at 60fps = 300ms animation)
2. Input and gravity are suspended
3. Marked rows blink white (3-frame on/off alternation)
4. When `flash_timer` reaches 0, rows are removed, score is awarded, next piece spawns

### High score persistence

```aria
// Load on startup
string:hs_content = raw(readFile("aria_tetris_best.txt"));
if (string_length(hs_content) > 0i64) {
    int64:hs_val = string_to_int(hs_content);
    if (hs_val > 0i64) { high_score = @cast<int32>(hs_val); }
}

// Save on game over if new record
if (score > high_score) {
    high_score = score;
    raw(writeFile("aria_tetris_best.txt",
        string_from_int(@cast<int64>(high_score))));
}
```

## Source Structure

```
aria-tetris/
├── aria-package.toml       # Package manifest
├── aria_tetris             # Compiled binary (not committed)
└── src/
    └── aria_tetris.aria    # Complete game source (~930 lines)
```

## See Also

- [From GML to Native Code](../guide/from_gml_to_native.md) — Tutorial tracing
  game code through the full stack
- [aria-gml](aria-gml.md) — GML-style API if you prefer that feel
- [aria-raylib](aria-raylib.md) — The underlying raylib bindings
