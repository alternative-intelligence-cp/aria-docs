# From GML to Native Code: A GameMaker Developer's Guide to Aria

**Target Audience**: Developers who learned programming through GameMaker Language (GML)
and want to understand how high-level game code connects to compiled native performance.

---

## Introduction

If you've ever written `draw_sprite(spr_player, 0, x, y)` in GameMaker and wondered
what actually happens inside the engine — this guide is for you.

We'll trace a single draw call all the way from GML source code down to the GPU driver,
and show how Aria lets you write that same game logic yourself, compiling directly to
native machine code with no runtime overhead.

---

## The Stack: GML vs Aria

```
                GML                               Aria
                ───                               ────
           your GML script                   your Aria source
                │                                   │
         GameMaker runner                     ariac compiler
         (closed-source)                       (open source)
                │                                   │
         hidden C++ code                     your C shim (.c)
                │                                   │
              OpenGL / raylib           ──────► raylib / OpenGL
                │                                   │
              GPU driver                       GPU driver
                │                                   │
              screen                            screen
```

In GameMaker, every step of the middle stack is hidden from you. In Aria, you own
every layer. The C shim is your code. The library calls are your calls. Nothing is magic.

---

## Comparison: A Simple Draw Call

### GML

```gml
// In a Draw event:
draw_set_color(c_cyan);
draw_rectangle(x, y, x + 32, y + 32, false);
draw_set_color(c_white);
draw_text(x, y - 20, "Hello!");
```

### Aria (using aria-gml for a familiar feel)

```aria
use "../src/aria_gml.aria".*;

func:step = void(flt64:dt) { };

func:draw = void() {
    draw_set_color(make_color_rgb(0i32, 240i32, 240i32));
    draw_rectangle(100.0f64, 200.0f64, 132.0f64, 232.0f64);
    draw_set_color(make_color_rgb(255i32, 255i32, 255i32));
    draw_text(100.0f64, 180.0f64, "Hello!");
}

func:main = int32() {
    gml_game_run(640i32, 480i32, "My Game", step, draw);
    pass(0i32);
};
```

### Aria (using raw raylib, no GML layer)

```aria
extern "aria_raylib_shim" {
    func:aria_rl_draw_rectangle = void(int32:x, int32:y, int32:w, int32:h,
                                       int32:r, int32:g, int32:b, int32:a);
    func:aria_rl_draw_text      = void(string:text, int32:x, int32:y,
                                       int32:font_size,
                                       int32:r, int32:g, int32:b, int32:a);
    // ... other externs
}

// In your draw section:
aria_rl_draw_rectangle(100i32, 200i32, 32i32, 32i32, 0i32, 240i32, 240i32, 255i32);
aria_rl_draw_text("Hello!", 100i32, 180i32, 20i32, 255i32, 255i32, 255i32, 255i32);
```

The GML layer just wraps the raylib layer, which wraps the GPU driver. You can work at
any level you choose.

---

## How `draw_sprite()` Works — All the Way Down

Let's trace `draw_sprite(spr_player, 0, x, y)` in GML, and the equivalent in Aria.

### Step 1: GML source → GameMaker bytecode

GameMaker compiles your GML into its own bytecode format (YYC compiles to native C++
instead, but that's still hidden). You can't inspect it.

### Step 2: Aria source → machine code via ariac

When you write:
```aria
draw_sprite(player_sprite, 0.0f64, 128.0f64, 64.0f64);
```

The `ariac` compiler:
1. Parses `draw_sprite` as an `extern` function declared in `aria_gml.aria`
2. Emits an LLVM IR `call` instruction to the external symbol `aria_gml_draw_sprite`
3. LLVM backend generates an x86-64 `call` instruction in the output binary

You can see the IR that ariac generates — it's just function calls, no magic.

### Step 3: The C shim

The Aria binary calls `aria_gml_draw_sprite()` in `libaria_gml_shim.so`. Here's what
that function looks like in `aria_gml_shim.c`:

```c
// Simplified — actual source is in aria-packages/packages/aria-gml/shim/
void aria_gml_draw_sprite(int32_t handle, double subimage,
                           double x, double y) {
    if (handle < 0 || handle >= MAX_SPRITES) return;
    Sprite *spr = &sprites[handle];
    int frame = (int)subimage % spr->frame_count;
    // Draw the sprite's texture region at (x, y)
    DrawTextureRec(spr->texture,
                   (Rectangle){frame * spr->w, 0, spr->w, spr->h},
                   (Vector2){(float)x, (float)y},
                   WHITE);
}
```

This calls `DrawTextureRec()` from raylib.

### Step 4: raylib → OpenGL

`DrawTextureRec()` in raylib does something like:

```c
// Inside raylib source (rlgl.h):
rlSetTexture(texture.id);
rlBegin(RL_QUADS);
    rlTexCoord2f(srcRec.x/texture.width, srcRec.y/texture.height);
    rlVertex2f(dest.x, dest.y);
    // ... 4 vertices ...
rlEnd();
rlSetTexture(0);
```

### Step 5: OpenGL → GPU driver → screen

The OpenGL calls get translated into GPU commands by your system's driver (Mesa/NVIDIA/AMD).
The driver uploads the vertex data and texture coordinates to the GPU, which runs a
fragment shader and writes each pixel to the framebuffer.

**The full chain:**
```
your Aria source
     ↓  ariac compiles to x86-64 + links to .so
aria_gml_draw_sprite() in your C shim
     ↓  C function call
DrawTextureRec() in raylib
     ↓  raylib calls OpenGL
glBindTexture() + glDrawArrays()
     ↓  OpenGL driver translates to GPU commands
GPU renders pixels to framebuffer
     ↓
screen
```

**In Aria, you own layers 1–3. In GML, GameMaker owns layers 2–3 for you.**

---

## How Shared Libraries Work

When GameMaker loads a GML extension, it calls `external_define()`:

```gml
var ext = external_define("mylib.dll", "my_function",
                           dll_cdecl, ty_real, 2, ty_real, ty_real);
var result = external_call(ext, x, y);
```

This is exactly how Aria's `extern` blocks work — except Aria generates the equivalent
calls at compile time, not runtime.

```aria
extern "mylib" {
    func:my_function = flt64(flt64:x, flt64:y);
}

// This compiles to the same thing as external_call() at runtime
flt64:result = my_function(x, y);
```

### .so vs .dll

| Platform | Extension | Load command |
|----------|-----------|--------------|
| Linux    | `.so`     | `dlopen()`   |
| Windows  | `.dll`    | `LoadLibrary()` |
| macOS    | `.dylib`  | `dlopen()`   |

At runtime, your Aria binary uses the system linker to resolve `aria_gml_draw_sprite`
to the address of that function in `libaria_gml_shim.so`. Same mechanism as
`external_call()` in GML.

---

## Side-by-Side: Bouncing Ball

Here's the same bouncing ball game in GML and Aria.

### GML (in GameMaker project)

```gml
// Create event (obj_ball):
x = 320;
y = 240;
hspeed = 4;
vspeed = 3;

// Step event (obj_ball):
if (x < 16 || x > room_width - 16) hspeed = -hspeed;
if (y < 16 || y > room_height - 16) vspeed = -vspeed;

// Draw event (obj_ball):
draw_set_color(c_yellow);
draw_circle(x, y, 16, false);
draw_set_color(c_white);
draw_text(8, 8, "Bouncing Ball");
```

### Aria (using aria-gml)

```aria
use "../src/aria_gml.aria".*;

flt64:bx  = 320.0f64;
flt64:by  = 240.0f64;
flt64:bvx = 4.0f64;
flt64:bvy = 3.0f64;

func:step = void(flt64:dt) {
    bx = bx + bvx;
    by = by + bvy;
    if (bx < 16.0f64)  { bvx = 0.0f64 - bvx; }
    if (bx > 624.0f64) { bvx = 0.0f64 - bvx; }
    if (by < 16.0f64)  { bvy = 0.0f64 - bvy; }
    if (by > 464.0f64) { bvy = 0.0f64 - bvy; }
};

func:draw = void() {
    draw_set_color(make_color_rgb(255i32, 220i32, 0i32));
    draw_circle(bx, by, 16.0f64);
    draw_set_color(make_color_rgb(255i32, 255i32, 255i32));
    draw_text(8.0f64, 8.0f64, "Bouncing Ball");
};

func:main = int32() {
    gml_game_run(640i32, 480i32, "Bouncing Ball", step, draw);
    pass(0i32);
};
```

The structure is identical: create (init) → step → draw, just with Aria syntax.

### What's different?

| Feature | GML | Aria |
|---------|-----|------|
| Compilation | JIT / YYC | Ahead-of-time to native |
| Startup time | ~2s (runner) | <10ms |
| Binary size | ~50MB (runner) | ~500KB |
| Memory | GC + runner overhead | Explicit, no GC |
| Distribution | GameMaker runner required | Single binary |
| Modify behavior | Extensions + GML only | Change any layer |

---

## How the Compiler Turns Aria into Machine Code

### 1. Parse

```
aria_rl_draw_rectangle(100i32, 200i32, 32i32, 32i32, 0i32, 240i32, 240i32, 255i32);
```

The parser builds an AST node: `FunctionCall("aria_rl_draw_rectangle", [100, 200, ...])`

### 2. Type check

The type checker confirms `aria_rl_draw_rectangle` was declared in an `extern` block
with 8 `int32` parameters, verifies all arguments match.

### 3. IR generation

The IR generator emits LLVM IR:
```llvm
call void @aria_rl_draw_rectangle(i32 100, i32 200, i32 32, i32 32,
                                   i32 0, i32 240, i32 240, i32 255)
```

### 4. LLVM optimization + codegen

LLVM optimizes (constant folding, inlining, etc.) and emits x86-64:
```asm
; Push arguments right-to-left (System V AMD64 ABI: first 6 in registers)
mov  edi, 100       ; x
mov  esi, 200       ; y
mov  edx, 32        ; w
mov  ecx, 32        ; h
mov  r8d, 0         ; r
mov  r9d, 240       ; g
; remaining args on stack...
call aria_rl_draw_rectangle@PLT
```

### 5. Linker

The linker resolves `aria_rl_draw_rectangle` to the address in `libaria_raylib_shim.so`,
writing the PLT entry. At runtime, the dynamic linker fills in the actual address.

This is exactly what happens when GameMaker's YYC mode calls a native function.

---

## The ABI Layer: Why the Shim Exists

GameMaker extensions use `double` and `char*` for all parameters because that's GML's
type system. Aria has a full type system (int32, flt32, flt64, string, etc.) but its
FFI ABI has some quirks:

- `flt32` (32-bit float) is passed as 64-bit double at the ABI level
- Arrays are passed as pointers to their first element
- Strings are `char*` null-terminated

The C shim handles these conversions:

```c
// In aria_raylib_shim.c:
// Aria declares: func:aria_rl_draw_rectangle = void(int32:x, ...)
// C shim:
void aria_rl_draw_rectangle(int32_t x, int32_t y, int32_t w, int32_t h,
                              int32_t r, int32_t g, int32_t b, int32_t a) {
    DrawRectangle(x, y, w, h, (Color){r, g, b, a});
    // raylib's Color struct is decomposed to scalars here ↑
}
```

raylib's `DrawRectangle` takes a `Color` struct. You can't pass structs across the
Aria FFI directly (no struct types in Aria yet), so the shim decomposes them to scalars.
This is the same reason GML extensions use only `double` — the FFI boundary is always
the simplest common type.

---

## Building an Aria Game from Scratch

### 1. Install

```bash
git clone https://github.com/aria-lang/aria
cd aria && make release
export PATH="$PATH:$(pwd)/build"
```

### 2. Get aria-packages

```bash
git clone https://github.com/aria-lang/aria-packages
cd aria-packages/packages/aria-gml/shim
make           # builds libaria_gml_shim.so
```

### 3. Write your game

```aria
use "../aria-gml/src/aria_gml.aria".*;

flt64:px = 300.0f64;
flt64:py = 200.0f64;
flt64:spd = 3.0f64;

func:step = void(flt64:dt) {
    if (keyboard_check(vk_left)  == 1i32) { px = px - spd; }
    if (keyboard_check(vk_right) == 1i32) { px = px + spd; }
    if (keyboard_check(vk_up)    == 1i32) { py = py - spd; }
    if (keyboard_check(vk_down)  == 1i32) { py = py + spd; }
};

func:draw = void() {
    draw_set_color(make_color_rgb(0i32, 200i32, 255i32));
    draw_rectangle(px - 16.0f64, py - 16.0f64, px + 16.0f64, py + 16.0f64);
    draw_set_color(make_color_rgb(255i32, 255i32, 255i32));
    draw_text(8.0f64, 8.0f64, "Arrow keys to move");
};

func:main = int32() {
    gml_game_run(640i32, 480i32, "Player Movement", step, draw);
    pass(0i32);
};
```

### 4. Compile

```bash
ariac my_game.aria -o my_game \
  -L path/to/aria-gml/shim \
  -laria_gml_shim -lraylib -lm

export LD_LIBRARY_PATH=path/to/aria-gml/shim
./my_game
```

---

## Key Differences from GML

| Concept | GML | Aria |
|---------|-----|------|
| Variables | `var x = 5;` | `int32:x = 5i32;` |
| Functions | `function my_func(a, b) { return a + b; }` | `func:my_func = int32(int32:a, int32:b) { pass(a + b); };` |
| Loops | `for (var i = 0; i < 10; i++) { }` | `for(int32:i = 0i32; i < 10i32; i += 1i32) { }` |
| Conditionals | `if (x > 0) { } else { }` | `if (x > 0i32) { }` (no else yet — use flag pattern) |
| Types | Dynamic (all values are `double` internally) | Static: `int32`, `flt64`, `string`, `bool`, etc. |
| Return | `return value;` | `pass(value);` |
| Strings | `"hello" + " world"` | `string_concat("hello", " world")` |
| Arrays | `array[i] = x;` | Fixed-size: `int32[10]:arr = ...; arr[i] = x;` |
| Instance vars | `self.x`, implicit in events | Explicit variables in enclosing scope |
| Objects | Instances with Create/Step/Draw events | Functions + state via closures or globals |
| Structs | `{ x: 1, y: 2 }` | Not yet (planned for v0.3.x) |

---

## What's Next

- **aria-tetris**: A complete Tetris implementation in Aria — gamepad, sound, high
  score, line-clear animation. Read the source to see these patterns at scale.
- **aria-gml**: The full GML compatibility layer — 40+ GML functions, all documented.  
- **aria-raylib**: The lower-level raylib bindings — use these when you need full control
  over rendering.
- **aria-opengl**: Direct OpenGL 3.3 bindings — for custom shaders and advanced rendering.

See [aria-docs/reference/](../reference/) for the full language reference, and
[aria-docs/guide/](.) for in-depth topic guides.
