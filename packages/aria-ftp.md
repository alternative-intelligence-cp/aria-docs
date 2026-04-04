# aria-ftp

**Version**: 0.12.2
**Category**: networking, protocol
**License**: MIT

FTP command builder and reply parser for Aria. Pure Aria implementation —
builds FTP protocol strings and parses server responses. Pair with
`aria-socket` for actual network I/O.

## API Reference

| Function | Description |
|----------|-------------|
| `ftp_create() → int64` | Create FTP session handle |
| `ftp_cmd_user(int64, string) → string` | Build USER command |
| `ftp_cmd_pass(int64, string) → string` | Build PASS command |
| `ftp_cmd_cwd(int64, string) → string` | Build CWD (change directory) command |
| `ftp_cmd_pwd(int64) → string` | Build PWD command |
| `ftp_cmd_list(int64) → string` | Build LIST command |
| `ftp_cmd_retr(int64, string) → string` | Build RETR (download) command |
| `ftp_cmd_stor(int64, string) → string` | Build STOR (upload) command |
| `ftp_cmd_dele(int64, string) → string` | Build DELE (delete) command |
| `ftp_cmd_mkd(int64, string) → string` | Build MKD (mkdir) command |
| `ftp_cmd_type(int64, string) → string` | Build TYPE command (A/I) |
| `ftp_cmd_pasv(int64) → string` | Build PASV command |
| `ftp_cmd_quit(int64) → string` | Build QUIT command |
| `ftp_parse_reply(int64, string) → int32` | Parse FTP reply line |
| `ftp_reply_code(int64) → string` | Get reply code (e.g., "220") |
| `ftp_reply_class(int64) → string` | Get reply class (1-5) |
| `ftp_reply_msg(int64) → string` | Get reply message text |
| `ftp_last_cmd(int64) → string` | Get last command sent |

**Type:FTP** wrapper available.

## Quick Start

```aria
use "aria_ftp.aria".*;

func:main = int32() {
    int64:ftp = raw ftp_create();
    string:cmd1 = raw ftp_cmd_user(ftp, "anonymous");
    string:cmd2 = raw ftp_cmd_pass(ftp, "user@example.com");
    string:cmd3 = raw ftp_cmd_list(ftp);
    println(cmd1);
    int32:_p = raw ftp_parse_reply(ftp, "220 Welcome to FTP");
    string:code = raw ftp_reply_code(ftp);
    println(code);
    exit(0i32);
};
```

## See Also

- [aria-socket](aria-socket.md) — Raw sockets for transport
- [aria-smtp](aria-smtp.md) — SMTP protocol builder
