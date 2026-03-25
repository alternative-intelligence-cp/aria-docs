# aria-raylib

**Version**: 0.3.0
**Category**: graphics, gamedev, audio, input
**License**: Apache-2.0 WITH Runtime-Library-Exception

raylib bindings for Aria — the primary game development package. Provides
windowing, 2D drawing, texture loading, audio playback, keyboard/mouse/gamepad
input, and procedural sound generation through a thin C shim.

This is the layer that `aria-gml` and `aria-tetris` build on.

## Features

- Window lifecycle and frame management
- 2D drawing: rectangles, circles, lines, triangles, text, textures
- Color structs decomposed to RGBA scalars (safe across FFI)
- Keyboard, mouse, and gamepad input (xinput-compatible)
- Audio: init/play/stop/unload sounds and music streams
- Procedural tone generation: `rl_gen_beep()` synthesizes square/triangle/sawtooth/sine
- Texture atlas support: `DrawTextureRec`-style drawing
- Frame timing: `rl_get_delta_time()`, `rl_get_fps()`

## Quick Start

```aria
use "../src/aria_raylib.aria".*;

func:main = int32() {
    rl_init_window(640i32, 480i32, "Hello Aria");
    rl_set_target_fps(60i32);

    rl_init_audio_device();
    int32:beep = rl_gen_beep(440i32, 200i32, 3i32, 0.5f32);  // 440Hz sine, 200ms

    int32:done = 0i32;
    while (done == 0i32) {
        if (rl_is_key_pressed(32i32) == 1i32) {  // Space
            rl_play_sound(beep);
        }

        rl_begin_drawing();
        rl_clear_background(30i32, 30i32, 30i32, 255i32);
        rl_draw_text("Press SPACE to beep!", 120i32, 200i32, 28i32,
                     255i32, 255i32, 255i32, 255i32);
        rl_end_drawing();

        done = rl_window_should_close();
    }

    rl_close_audio_device();
    rl_close_window();
    pass(0i32);
};
```

## API Reference

### Window

```aria
func:rl_init_window        = void(int32:w, int32:h, string:title);
func:rl_close_window       = void();
func:rl_window_should_close = int32();  // 1 = ESC / close button
func:rl_set_target_fps     = void(int32:fps);
func:rl_get_delta_time     = flt32();
func:rl_get_fps            = int32();
```

### Drawing

```aria
func:rl_begin_drawing      = void();
func:rl_end_drawing        = void();
func:rl_clear_background   = void(int32:r, int32:g, int32:b, int32:a);

func:rl_draw_rectangle       = void(int32:x, int32:y, int32:w, int32:h,
                                    int32:r, int32:g, int32:b, int32:a);
func:rl_draw_rectangle_lines = void(int32:x, int32:y, int32:w, int32:h,
                                    int32:r, int32:g, int32:b, int32:a);
func:rl_draw_circle          = void(int32:cx, int32:cy, flt32:radius,
                                    int32:r, int32:g, int32:b, int32:a);
func:rl_draw_line            = void(int32:x1, int32:y1, int32:x2, int32:y2,
                                    int32:r, int32:g, int32:b, int32:a);
func:rl_draw_text            = void(string:text, int32:x, int32:y,
                                    int32:font_size,
                                    int32:r, int32:g, int32:b, int32:a);
func:rl_measure_text         = int32(string:text, int32:font_size);
```

### Textures

```aria
func:rl_load_texture        = int32(string:path);
func:rl_unload_texture      = void(int32:handle);
func:rl_draw_texture        = void(int32:handle, int32:x, int32:y,
                                   int32:r, int32:g, int32:b, int32:a);
func:rl_draw_texture_rec    = void(int32:handle,
                                   int32:src_x, int32:src_y,
                                   int32:src_w, int32:src_h,
                                   int32:dst_x, int32:dst_y,
                                   int32:r, int32:g, int32:b, int32:a);
func:rl_get_texture_width   = int32(int32:handle);
func:rl_get_texture_height  = int32(int32:handle);
```

### Keyboard

```aria
func:rl_is_key_pressed  = int32(int32:key);  // just pressed
func:rl_is_key_down     = int32(int32:key);  // held
func:rl_is_key_released = int32(int32:key);  // just released

// Common key codes (raylib KEY_* enum values):
// KEY_SPACE=32, KEY_ENTER=257, KEY_ESC=256
// KEY_RIGHT=262, KEY_LEFT=263, KEY_DOWN=264, KEY_UP=265
// KEY_A..KEY_Z = 65..90
```

### Mouse

```aria
func:rl_is_mouse_button_pressed  = int32(int32:btn);
func:rl_is_mouse_button_down     = int32(int32:btn);
func:rl_is_mouse_button_released = int32(int32:btn);
func:rl_get_mouse_x              = int32();
func:rl_get_mouse_y              = int32();
func:rl_get_mouse_wheel          = flt32();
```

### Gamepad

```aria
func:rl_is_gamepad_available       = int32(int32:gamepad);
func:rl_is_gamepad_button_pressed  = int32(int32:gamepad, int32:button);
func:rl_is_gamepad_button_down     = int32(int32:gamepad, int32:button);
func:rl_is_gamepad_button_released = int32(int32:gamepad, int32:button);
func:rl_get_gamepad_axis_movement  = flt32(int32:gamepad, int32:axis);

// Button constants (raylib GamepadButton enum):
int32:GP_DPAD_UP    = 1i32;    int32:GP_DPAD_RIGHT = 2i32;
int32:GP_DPAD_DOWN  = 3i32;    int32:GP_DPAD_LEFT  = 4i32;
int32:GP_FACE_Y     = 5i32;    int32:GP_FACE_B     = 6i32;
int32:GP_FACE_A     = 7i32;    int32:GP_FACE_X     = 8i32;
int32:GP_L1         = 9i32;    int32:GP_L2         = 10i32;
int32:GP_R1         = 11i32;   int32:GP_R2         = 12i32;
int32:GP_SELECT     = 13i32;   int32:GP_HOME       = 14i32;
int32:GP_START      = 15i32;   int32:GP_L3         = 16i32;
int32:GP_R3         = 17i32;

// Axis constants:
int32:GP_AXIS_LEFT_X  = 0i32;  int32:GP_AXIS_LEFT_Y  = 1i32;
int32:GP_AXIS_RIGHT_X = 2i32;  int32:GP_AXIS_RIGHT_Y = 3i32;
```

### Audio

```aria
func:rl_init_audio_device  = void();
func:rl_close_audio_device = void();

// Load from file:
func:rl_load_sound   = int32(string:path);  // returns slot index
func:rl_unload_sound = void(int32:handle);
func:rl_load_music   = int32(string:path);
func:rl_unload_music = void(int32:handle);

// Playback:
func:rl_play_sound   = void(int32:handle);
func:rl_stop_sound   = void(int32:handle);
func:rl_play_music   = void(int32:handle);
func:rl_stop_music   = void(int32:handle);
func:rl_update_music = void(int32:handle);  // call every frame

// Procedural synthesis — no audio file needed:
func:rl_gen_beep = int32(int32:freq_hz, int32:dur_ms,
                          int32:wave_type, flt32:volume);
// wave_type: 0=square  1=triangle  2=sawtooth  3=sine
// Returns slot index. Use with rl_play_sound() / rl_unload_sound().
```

## Procedural Sound: rl_gen_beep

`rl_gen_beep` synthesizes a tone in C memory and loads it into raylib's sound
system. No `.wav` or audio file is needed — great for games that want sound
without bundling audio assets.

```aria
aria_rl_init_audio_device();

// Move sound: 220Hz square, 40ms, moderate volume
int32:snd_move = rl_gen_beep(220i32, 40i32, 0i32, 0.3f32);

// Tetris clear: 880Hz triangle, 200ms, loud
int32:snd_tetris = rl_gen_beep(880i32, 200i32, 1i32, 0.6f32);

// Play on event:
rl_play_sound(snd_move);
```

The function applies a 10% fade-out envelope to avoid clicks.

## Build

```bash
cd packages/aria-raylib/shim
cc -O2 -shared -fPIC -Wall -o libaria_raylib_shim.so \
   aria_raylib_shim.c $(pkg-config --cflags --libs raylib) -lm
```

## Source Structure

```
aria-raylib/
├── aria-package.toml
├── shim/
│   ├── aria_raylib_shim.c   # C bridge (~540 lines)
│   └── libaria_raylib_shim.so
└── src/
    └── aria_raylib.aria     # Aria bindings + pub wrappers + constants
```

## See Also

- [aria-tetris](aria-tetris.md) — Full game using this package
- [aria-gml](aria-gml.md) — GML-api layer built on aria-raylib
- [aria-opengl](aria-opengl.md) — Direct OpenGL 3.3 for custom shaders
