# aria-input

**Version**: 0.12.2
**Category**: terminal, input
**License**: MIT

Key mapping and button state manager for Aria. Pure Aria implementation —
tracks button press/release via bitmask operations (division/modulo pattern),
key name bindings, and frame-based input updates. Rebuilt from FFI in v0.12.2.

## API Reference

| Function | Description |
|----------|-------------|
| `inp_create() → int64` | Create input manager handle |
| `inp_set_buttons(int64, int64) → int32` | Set raw button bitmask |
| `inp_buttons(int64) → string` | Get current bitmask as string |
| `inp_has_button(int64, int64) → int64` | Test if button is set (1/0) |
| `inp_update(int64) → int32` | Advance to next frame |
| `inp_btnp(int64, int64) → int64` | Button just pressed this frame |
| `inp_btnr(int64, int64) → int64` | Button just released this frame |
| `inp_clear(int64) → int32` | Clear all button state |
| `inp_bind_key(int64, string, int64) → int32` | Bind key name to button code |
| `inp_key_to_btn(int64, string) → string` | Lookup button code for key name |
| `inp_btn_name(int64) → string` | Get button name from code |

**Type:Input** wrapper available.

## Quick Start

```aria
use "aria_input.aria".*;

func:main = int32() {
    int64:inp = raw inp_create();
    int32:_b = raw inp_bind_key(inp, "space", 1i64);
    int32:_s = raw inp_set_buttons(inp, 3i64);
    int64:has = raw inp_has_button(inp, 1i64);
    int32:_u = raw inp_update(inp);
    string:name = raw inp_btn_name(1i64);
    println(name);
    exit(0i32);
};
```

## See Also

- [aria-display](aria-display.md) — Terminal display state
