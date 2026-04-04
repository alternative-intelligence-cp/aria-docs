# aria-smtp

**Version**: 0.12.2
**Category**: networking, protocol, email
**License**: MIT

SMTP command builder and reply parser for Aria. Pure Aria implementation —
builds SMTP protocol strings, composes email messages, and parses server
responses. Pair with `aria-socket` for transport.

## API Reference

| Function | Description |
|----------|-------------|
| `smtp_create() → int64` | Create SMTP session handle |
| `smtp_cmd_ehlo(int64, string) → string` | Build EHLO command |
| `smtp_cmd_helo(int64, string) → string` | Build HELO command |
| `smtp_cmd_mail_from(int64, string) → string` | Build MAIL FROM command |
| `smtp_cmd_rcpt_to(int64, string) → string` | Build RCPT TO command |
| `smtp_cmd_data(int64) → string` | Build DATA command |
| `smtp_cmd_quit(int64) → string` | Build QUIT command |
| `smtp_cmd_rset(int64) → string` | Build RSET command |
| `smtp_cmd_noop(int64) → string` | Build NOOP command |
| `smtp_build_message(int64, string, string, string, string) → string` | Compose full email message |
| `smtp_parse_reply(int64, string) → int32` | Parse SMTP reply line |
| `smtp_reply_code(int64) → string` | Get reply code (e.g., "250") |
| `smtp_reply_class(int64) → string` | Get reply class (2-5) |
| `smtp_reply_msg(int64) → string` | Get reply message text |
| `smtp_last_cmd(int64) → string` | Get last command sent |

**Type:SMTP** wrapper available.

## Quick Start

```aria
use "aria_smtp.aria".*;

func:main = int32() {
    int64:sm = raw smtp_create();
    string:ehlo = raw smtp_cmd_ehlo(sm, "mail.example.com");
    string:from = raw smtp_cmd_mail_from(sm, "alice@example.com");
    string:to = raw smtp_cmd_rcpt_to(sm, "bob@example.com");
    string:msg = raw smtp_build_message(sm, "alice@example.com", "bob@example.com", "Hello", "Hi Bob!");
    println(ehlo);
    exit(0i32);
};
```

## See Also

- [aria-socket](aria-socket.md) — Raw sockets for transport
- [aria-ftp](aria-ftp.md) — FTP protocol builder
