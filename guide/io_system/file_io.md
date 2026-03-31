# File I/O

## Reading Files

```aria
string:content = raw readFile("data.txt");
```

## Writing Files

```aria
drop writeFile("output.txt", "Hello, file!");
```

## File Operations

| Function | Description |
|----------|-------------|
| `readFile(path)` | Read entire file as string |
| `writeFile(path, data)` | Write string to file |
| `fileExists(path)` | Check if file exists |
| `fileSize(path)` | Get file size in bytes |
| `deleteFile(path)` | Delete a file |

All file operations return `Result<T>`.

## Streams

| Stream | FD | Purpose |
|--------|----|---------|
| `stdin` | 0 | Standard input |
| `stdout` | 1 | Standard output |
| `stderr` | 2 | Standard error |
| `stddbg` | 3 | Debug stream (Hexstream) |
| `stddati` | 4 | Data input stream (Hexstream) |
| `stddato` | 5 | Data output stream (Hexstream) |

## Related

- [print.md](print.md) — print/println
- [stdin.md](stdin.md) — reading from stdin
- [sys.md](sys.md) — direct syscalls
