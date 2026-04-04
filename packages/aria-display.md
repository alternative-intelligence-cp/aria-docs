# aria-display

**Version**: 0.12.2
**Category**: terminal, ui
**License**: MIT

Terminal display state manager for Aria. Pure Aria implementation — tracks
cursor position, colors, text attributes, and terminal dimensions.
Rebuilt from FFI to pure Aria in v0.12.2.

## API Reference

| Function | Description |
|----------|-------------|
| `disp_create(int64, int64) → int64` | Create display (cols, rows) |
| `disp_reset(int64) → int32` | Reset all state to defaults |
| `disp_move(int64, int64, int64) → int32` | Move cursor to (row, col) |
| `disp_home(int64) → int32` | Move cursor to (0, 0) |
| `disp_hide_cursor(int64) → int32` | Hide cursor |
| `disp_show_cursor(int64) → int32` | Show cursor |
| `disp_cursor_row(int64) → string` | Get cursor row |
| `disp_cursor_col(int64) → string` | Get cursor column |
| `disp_cursor_visible(int64) → string` | Get cursor visibility ("1"/"0") |
| `disp_set_fg(int64, int64) → int32` | Set foreground color |
| `disp_set_bg(int64, int64) → int32` | Set background color |
| `disp_fg(int64) → string` | Get foreground color |
| `disp_bg(int64) → string` | Get background color |
| `disp_set_bold(int64, int64) → int32` | Set bold attribute |
| `disp_set_reverse(int64, int64) → int32` | Set reverse video attribute |
| `disp_resize(int64, int64, int64) → int32` | Resize terminal dimensions |
| `disp_cols(int64) → string` | Get column count |
| `disp_rows(int64) → string` | Get row count |
| `disp_color_name(int64) → string` | Get color name from code |

**Type:Display** wrapper available.

## Quick Start

```aria
use "aria_display.aria".*;

func:main = int32() {
    int64:d = raw disp_create(80i64, 24i64);
    int32:_m = raw disp_move(d, 10i64, 20i64);
    int32:_f = raw disp_set_fg(d, 2i64);
    int32:_b = raw disp_set_bold(d, 1i64);
    string:row = raw disp_cursor_row(d);
    string:col = raw disp_cursor_col(d);
    string:clr = raw disp_color_name(2i64);
    println(clr);
    exit(0i32);
};
```

## See Also

- [aria-input](aria-input.md) — Key and button input
