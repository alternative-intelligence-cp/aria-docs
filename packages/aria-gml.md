# aria-gml

**Version**: 0.2.8
**Category**: gamedev, compatibility, ffi
**License**: Apache-2.0 WITH Runtime-Library-Exception

GameMaker Language (GML) compatibility layer for Aria. Provides GML-style
function names and event-driven game loop semantics over `aria-raylib`, so
developers familiar with GameMaker can write Aria games immediately using
API names they already know.

This package is also the foundation of the educational track — see
[From GML to Native Code](../guide/from_gml_to_native.md) for a deep-dive
comparing GML source to the compiled native stack.

## Features

- 40+ GML-named drawing functions (`draw_rectangle`, `draw_circle`, `draw_text`, …)
- GML-style draw state: `draw_set_color()` and `draw_set_alpha()` persist across calls
- GML keyboard constants (`vk_left`, `vk_right`, `vk_space`, …)
- GML mouse functions (`mouse_x()`, `mouse_y()`, `mouse_check_button()`, …)
- GML color constants (`c_red`, `c_blue`, `c_white`, …) and `make_color_rgb()`
- GML random utilities (`irandom()`, `irandom_range()`, `random_val()`, `randomize()`)
- GML-style game loop model via `gml_game_run(w, h, title, step, draw)`
- Sprite management: `sprite_load()`, `draw_sprite()`, `sprite_get_width/height()`
- Sound management: `sound_load()`, `audio_play_sound()`, `audio_stop_sound()`
- Music streaming: `music_load()`, `music_play()`, `music_stop()`, `music_update()`
- Color packing: GML format `r | (g<<8) | (b<<16)`
- Corner-style rectangles: `draw_rectangle(x1, y1, x2, y2)` (same as GML)

## Quick Start

```aria
use "../src/aria_gml.aria".*;

flt64:bx  = 320.0f64;
flt64:by  = 240.0f64;
flt64:bvx = 3.0f64;
flt64:bvy = 2.0f64;

func:step = void(flt64:dt) {
    bx = bx + bvx;
    by = by + bvy;
    if (bx < 16.0f64)  { bvx = 0.0f64 - bvx; }
    if (bx > 624.0f64) { bvx = 0.0f64 - bvx; }
    if (by < 16.0f64)  { bvy = 0.0f64 - bvy; }
    if (by > 464.0f64) { bvy = 0.0f64 - bvy; }
};

func:draw = void() {
    draw_set_color(c_yellow);
    draw_circle(bx, by, 16.0f64);
    draw_set_color(c_white);
    draw_text(8.0f64, 8.0f64, "Bouncing Ball — written in Aria");
};

func:main = int32() {
    gml_game_run(640i32, 480i32, "My Game", step, draw);
    pass(0i32);
};
```

## API Reference

### Game Loop

```aria
// Entry point — runs the game loop at 60fps
func:gml_game_run = void(int32:w, int32:h, string:title,
                          void_func:step_fn, void_func:draw_fn);

// Discrete loop control (alternative to gml_game_run)
func:gml_begin     = void(int32:w, int32:h, string:title);
func:gml_end       = void();
func:gml_loop_once = int32();   // returns 0 when window closed
func:gml_step      = void(flt64:dt);
func:gml_draw_begin = void();
func:gml_draw_end   = void();
```

### Drawing

```aria
// Draw state
func:draw_set_color = void(int32:color);
func:draw_set_alpha = void(flt64:alpha);
func:draw_get_color = int32();
func:draw_get_alpha = flt64();
func:draw_set_font_size = void(int32:size);
func:draw_clear = void();

// Primitives
func:draw_rectangle = void(flt64:x1, flt64:y1, flt64:x2, flt64:y2);
func:draw_circle    = void(flt64:x, flt64:y, flt64:r);
func:draw_triangle  = void(flt64:x1, flt64:y1, flt64:x2, flt64:y2,
                            flt64:x3, flt64:y3);
func:draw_line      = void(flt64:x1, flt64:y1, flt64:x2, flt64:y2);
func:draw_point     = void(flt64:x, flt64:y);

// Text
func:draw_text      = void(flt64:x, flt64:y, string:text);
func:draw_text_ext  = void(flt64:x, flt64:y, string:text, int32:font_size);

// Sprites
func:draw_sprite     = void(int32:sprite, flt64:subimage, flt64:x, flt64:y);
func:draw_sprite_ext = void(int32:sprite, flt64:subimage,
                             flt64:x, flt64:y,
                             flt64:xscale, flt64:yscale,
                             flt64:rot, int32:color, flt64:alpha);
```

### Colors

```aria
// Color constants (same packed format as GML)
int32:c_black   = 0i32;
int32:c_white   = 16777215i32;
int32:c_red     = 255i32;
int32:c_green   = 65280i32;
int32:c_blue    = 16711680i32;
int32:c_yellow  = 65535i32;
int32:c_orange  = 4235519i32;
int32:c_purple  = 16711935i32;
int32:c_aqua    = 16776960i32;
int32:c_teal    = 8421376i32;
int32:c_gray    = 8421504i32;
int32:c_silver  = 12632256i32;

// Color utilities
func:make_color_rgb  = int32(int32:r, int32:g, int32:b);
func:color_get_red   = int32(int32:color);
func:color_get_green = int32(int32:color);
func:color_get_blue  = int32(int32:color);
```

### Keyboard

```aria
func:keyboard_check          = int32(int32:key);  // held
func:keyboard_check_pressed  = int32(int32:key);  // just pressed this frame
func:keyboard_check_released = int32(int32:key);  // just released this frame
func:keyboard_lastkey        = int32();

// Key constants (GML-compatible)
int32:vk_left  = 263i32;    int32:vk_right = 262i32;
int32:vk_up    = 265i32;    int32:vk_down  = 264i32;
int32:vk_space = 32i32;     int32:vk_enter = 257i32;
int32:vk_escape= 256i32;    int32:vk_shift = 340i32;
int32:vk_ctrl  = 341i32;    int32:vk_alt   = 342i32;
// ... plus vk_a through vk_z, vk_0 through vk_9, vk_f1 through vk_f12
```

### Mouse

```aria
func:mouse_x                      = flt64();
func:mouse_y                      = flt64();
func:mouse_check_button           = int32(int32:btn);
func:mouse_check_button_pressed   = int32(int32:btn);
func:mouse_check_button_released  = int32(int32:btn);
func:mouse_wheel                  = int32();

int32:mb_left   = 0i32;
int32:mb_right  = 1i32;
int32:mb_middle = 2i32;
```

### Audio

```aria
func:sound_load       = int32(string:path);
func:sound_delete     = void(int32:handle);
func:audio_play_sound = void(int32:handle);
func:audio_stop_sound = void(int32:handle);
func:audio_sound_is_playing = int32(int32:handle);
func:audio_set_sound_volume = void(int32:handle, flt64:vol);

func:music_load   = int32(string:path);
func:music_play   = void(int32:handle);
func:music_stop   = void(int32:handle);
func:music_update = void(int32:handle);
```

### Randomness

```aria
func:randomize      = void();                          // seed from time
func:irandom        = int32(int32:n);                  // 0..n-1
func:irandom_range  = int32(int32:lo, int32:hi);       // lo..hi-1
func:random_val     = flt64();                         // 0.0..1.0
func:random_range   = flt64(flt64:lo, flt64:hi);       // lo..hi
```

## Build

```bash
cd packages/aria-gml/shim
make    # produces libaria_gml_shim.so
```

Or manually:
```bash
cc -O2 -shared -fPIC -Wall -o libaria_gml_shim.so \
   aria_gml_shim.c $(pkg-config --cflags --libs raylib) -lm
```

## Compile Your Program

```bash
ariac my_game.aria -o my_game \
  -L packages/aria-gml/shim \
  -laria_gml_shim -lraylib -lm
```

## Source Structure

```
aria-gml/
├── aria-package.toml
├── shim/
│   ├── aria_gml_shim.c     # C bridge (~370 lines)
│   └── Makefile
├── src/
│   └── aria_gml.aria       # Aria bindings + pub wrappers + constants
└── examples/
    └── gml_bounce/         # Bouncing ball demo
```

## See Also

- [From GML to Native Code](../guide/from_gml_to_native.md) — Tutorial
- [aria-raylib](aria-raylib.md) — Lower-level raylib bindings
- [aria-tetris](aria-tetris.md) — Full game example using aria-raylib
