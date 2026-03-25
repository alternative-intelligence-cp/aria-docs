# aria-opengl

**Version**: 0.2.8
**Category**: graphics, gamedev, gpu
**License**: Apache-2.0 WITH Runtime-Library-Exception

OpenGL 3.3 Core Profile bindings for Aria. Provides direct access to the GPU
shader pipeline — vertex buffers, shader compilation, uniform uploads, and draw
calls — for custom 3D rendering and post-processing effects beyond what the
raylib-based packages support.

Uses SDL2 for windowing/OpenGL context creation and GLAD for GL function loading.

## Features

- Window creation with OpenGL 3.3 Core context via SDL2
- VSync control
- Viewport and clear operations
- Shader pipeline: compile vertex/fragment shaders, link programs, get uniforms
- Vertex buffer objects (VBO), vertex array objects (VAO), element buffers (EBO)
- Texture loading (via stb_image), bind to texture units
- Uniform upload: int, float, vec2/3/4, mat4
- Frame timing: `aria_gl_delta_time()`
- Input: keyboard + mouse (SDL2-based)
- Constants: depth/blend enable bits, texture parameters, buffer usage hints

## Quick Start

```aria
use "../src/aria_opengl.aria".*;

func:main = int32() {
    int32:ok = gl_init(800i32, 600i32, "OpenGL Triangle");
    if (ok != 1i32) { pass(1i32); }

    // Vertex data: 3 positions interleaved with 3 colors
    flt32[18]:verts = [
         0.0f32,  0.5f32, 0.0f32,  1.0f32, 0.0f32, 0.0f32,
        -0.5f32, -0.5f32, 0.0f32,  0.0f32, 1.0f32, 0.0f32,
         0.5f32, -0.5f32, 0.0f32,  0.0f32, 0.0f32, 1.0f32
    ];

    int32:vao = gl_gen_vertex_array();
    int32:vbo = gl_gen_buffer();
    gl_bind_vertex_array(vao);
    gl_bind_buffer(GL_ARRAY_BUFFER, vbo);
    gl_buffer_data_f32(GL_ARRAY_BUFFER, verts, 18i32, GL_STATIC_DRAW);
    gl_vertex_attrib_pointer(0i32, 3i32, 24i32, 0i32);   // position
    gl_vertex_attrib_pointer(1i32, 3i32, 24i32, 12i32);  // color
    gl_enable_vertex_attrib(0i32);
    gl_enable_vertex_attrib(1i32);

    string:vert_src = "#version 330 core\n\
layout(location=0) in vec3 aPos;\n\
layout(location=1) in vec3 aColor;\n\
out vec3 vColor;\n\
void main() { gl_Position = vec4(aPos,1.0); vColor = aColor; }\n";

    string:frag_src = "#version 330 core\n\
in vec3 vColor;\n\
out vec4 FragColor;\n\
void main() { FragColor = vec4(vColor, 1.0); }\n";

    int32:prog = gl_create_program_from_source(vert_src, frag_src);

    int32:done = 0i32;
    while (done == 0i32) {
        int32:ev = gl_poll_events();
        if (ev == -1i32) { done = 1i32; }

        gl_clear_color(0.08f32, 0.08f32, 0.12f32, 1.0f32);
        gl_clear(GL_COLOR_BUFFER_BIT);

        gl_use_program(prog);
        gl_bind_vertex_array(vao);
        gl_draw_arrays(GL_TRIANGLES, 0i32, 3i32);

        gl_swap();
    }

    gl_delete_buffer(vbo);
    gl_delete_vertex_array(vao);
    gl_delete_program(prog);
    gl_quit();
    pass(0i32);
};
```

## API Reference

### Window / Context

```aria
func:gl_init        = int32(int32:w, int32:h, string:title);  // returns 1 on success
func:gl_quit        = void();
func:gl_swap        = void();
func:gl_set_vsync   = int32(int32:enable);
func:gl_poll_events = int32();  // returns -1 on quit, 0 otherwise
func:gl_delta_time  = flt32();  // seconds since last frame
```

### Viewport & Clear

```aria
func:gl_viewport    = void(int32:x, int32:y, int32:w, int32:h);
func:gl_clear_color = void(flt32:r, flt32:g, flt32:b, flt32:a);
func:gl_clear       = void(int32:mask);
func:gl_enable      = void(int32:cap);
func:gl_disable     = void(int32:cap);

// Clear mask bits:
int32:GL_COLOR_BUFFER_BIT   = 16384i32;
int32:GL_DEPTH_BUFFER_BIT   = 256i32;
int32:GL_STENCIL_BUFFER_BIT = 1024i32;

// Enable/disable caps:
int32:GL_DEPTH_TEST = 2929i32;
int32:GL_BLEND      = 3042i32;
```

### Shaders

```aria
func:gl_create_shader          = int32(int32:type);
func:gl_shader_source          = void(int32:shader, string:src);
func:gl_compile_shader         = void(int32:shader);
func:gl_get_shader_status      = int32(int32:shader);  // 1=ok, 0=error
func:gl_print_shader_log       = void(int32:shader);
func:gl_delete_shader          = void(int32:shader);
func:gl_create_program         = int32();
func:gl_attach_shader          = void(int32:prog, int32:shader);
func:gl_link_program           = void(int32:prog);
func:gl_get_program_status     = int32(int32:prog);
func:gl_use_program            = void(int32:prog);
func:gl_delete_program         = void(int32:prog);

// Convenience: compile+link in one call
func:gl_create_program_from_source = int32(string:vert_src, string:frag_src);

// Shader types:
int32:GL_VERTEX_SHADER   = 35633i32;
int32:GL_FRAGMENT_SHADER = 35632i32;
```

### Uniforms

```aria
func:gl_get_uniform_location = int32(int32:prog, string:name);
func:gl_uniform_1i           = void(int32:loc, int32:v);
func:gl_uniform_1f           = void(int32:loc, flt32:v);
func:gl_uniform_2f           = void(int32:loc, flt32:x, flt32:y);
func:gl_uniform_3f           = void(int32:loc, flt32:x, flt32:y, flt32:z);
func:gl_uniform_4f           = void(int32:loc, flt32:x, flt32:y, flt32:z, flt32:w);
func:gl_uniform_mat4         = void(int32:loc, flt32[16]:mat);
```

### Buffers & Arrays

```aria
func:gl_gen_vertex_array  = int32();
func:gl_bind_vertex_array = void(int32:vao);
func:gl_delete_vertex_array = void(int32:vao);

func:gl_gen_buffer    = int32();
func:gl_bind_buffer   = void(int32:target, int32:buf);
func:gl_delete_buffer = void(int32:buf);

func:gl_buffer_data_f32 = void(int32:target, flt32[]:data,
                                int32:count, int32:usage);
func:gl_buffer_data_u32 = void(int32:target, uint32[]:data,
                                int32:count, int32:usage);

func:gl_vertex_attrib_pointer = void(int32:index, int32:size,
                                      int32:stride, int32:offset);
func:gl_enable_vertex_attrib  = void(int32:index);

// Buffer targets:
int32:GL_ARRAY_BUFFER         = 34962i32;
int32:GL_ELEMENT_ARRAY_BUFFER = 34963i32;

// Usage hints:
int32:GL_STATIC_DRAW  = 35044i32;
int32:GL_DYNAMIC_DRAW = 35048i32;
```

### Drawing

```aria
func:gl_draw_arrays   = void(int32:mode, int32:first, int32:count);
func:gl_draw_elements = void(int32:mode, int32:count);

// Primitive modes:
int32:GL_POINTS    = 0i32;
int32:GL_LINES     = 1i32;
int32:GL_TRIANGLES = 4i32;
int32:GL_TRIANGLE_STRIP = 5i32;
```

### Textures

```aria
func:gl_load_texture   = int32(string:path);  // loads PNG/JPG via stb_image
func:gl_bind_texture   = void(int32:unit, int32:tex);
func:gl_delete_texture = void(int32:tex);

// Texture parameters:
int32:GL_TEXTURE_2D       = 3553i32;
int32:GL_NEAREST          = 9728i32;
int32:GL_LINEAR           = 9729i32;
int32:GL_REPEAT           = 10497i32;
int32:GL_CLAMP_TO_EDGE    = 33071i32;
```

### Input

```aria
func:gl_key_down     = int32(int32:scancode);  // held
func:gl_key_pressed  = int32(int32:scancode);  // just pressed
func:gl_mouse_x      = int32();
func:gl_mouse_y      = int32();
func:gl_mouse_button = int32(int32:btn);
```

## Build

```bash
cd packages/aria-opengl/shim
make    # produces libaria_opengl_shim.so
```

Requires: `libSDL2-dev`, GLAD headers (included).

## Compile Your Program

```bash
ariac my_app.aria -o my_app \
  -L packages/aria-opengl/shim \
  -laria_opengl_shim $(sdl2-config --libs) -lGL -lm
```

## Source Structure

```
aria-opengl/
├── aria-package.toml
├── shim/
│   ├── aria_opengl_shim.c  # C bridge (~500 lines) — SDL2 + GLAD + GL
│   ├── glad.c              # GLAD OpenGL loader
│   ├── glad.h
│   ├── khrplatform.h
│   └── Makefile
└── src/
    └── aria_opengl.aria    # Aria bindings + constants (pub wrappers)
```

## See Also

- [aria-raylib](aria-raylib.md) — Higher-level 2D rendering (simpler to use)
- [aria-tetris](aria-tetris.md) — Example of aria-raylib for a complete game
