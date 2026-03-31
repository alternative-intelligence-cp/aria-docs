# Standard Input

## Reading from stdin

```aria
string:line = raw stdin_read_line();    // read one line
string:all = raw stdin_read_all();      // read until EOF
```

Both return `Result<string>`.

## Related

- [print.md](print.md) — output
- [file_io.md](file_io.md) — file operations
