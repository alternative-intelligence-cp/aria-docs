# aria-dns

**Version**: 0.12.0
**Category**: networking, dns
**License**: MIT

DNS resolution for Aria. Forward and reverse lookups, IPv4 validation,
hostname detection, and safe fallback resolution.

## Features

- Forward DNS resolution (hostname → IP)
- Reverse DNS lookup (IP → hostname)
- Multi-result resolution with `dns_resolve_all`
- IPv4 address and hostname validation
- Fallback resolution with `dns_resolve_or`

## API Reference

| Function | Description |
|----------|-------------|
| `dns_resolve(string) → string` | Resolve hostname to first IP address |
| `dns_resolve_all(string) → string` | Resolve hostname to all IP addresses |
| `dns_reverse(string) → string` | Reverse lookup IP → hostname |
| `dns_is_ipv4(string) → int64` | Check if string is valid IPv4 address |
| `dns_is_hostname(string) → int64` | Check if string is a hostname |
| `dns_resolve_or(string, string) → string` | Resolve with fallback on failure |
| `dns_can_resolve(string) → int64` | Check if hostname is resolvable |

**Type:Dns** wrapper with methods: `resolve`, `resolve_all`, `reverse`, `is_ipv4`, `is_hostname`, `resolve_or`, `can_resolve`.

## Quick Start

```aria
use "aria_dns.aria".*;

func:main = int32() {
    string:ip = raw dns_resolve("example.com");
    println(ip);
    int64:valid = raw dns_is_ipv4(ip);
    exit(0i32);
};
```

## See Also

- [aria-http](aria-http.md) — HTTP client
- [aria-socket](aria-socket.md) — Raw TCP/UDP sockets
