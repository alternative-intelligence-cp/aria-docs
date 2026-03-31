# Bitwise Operators

**Bitwise operations require unsigned types** (`uint8`, `uint16`, `uint32`, `uint64`).

| Operator | Operation | Example |
|----------|-----------|---------|
| `&` | Bitwise AND | `(a & b)` |
| `\|` | Bitwise OR | `(a \| b)` |
| `^` | Bitwise XOR | `(a ^ b)` |
| `~` | Bitwise NOT | `(~a)` |
| `<<` | Left shift | `(a << n)` |
| `>>` | Right shift | `(a >> n)` |

## Compound Assignment

| Operator | Example |
|----------|---------|
| `<<=` | `flags <<= 1;` |
| `>>=` | `flags >>= 1;` |

## Example

```aria
uint32:flags = 0xFF00u32;
uint32:mask  = 0x0FF0u32;

uint32:result = (flags & mask);   // 0x0F00
uint32:all    = (flags | mask);   // 0xFFF0
uint32:diff   = (flags ^ mask);   // 0xF0F0
uint32:inv    = (~flags);         // 0xFFFF00FF
uint32:shl    = (flags << 4u32);  // 0xF0000
uint32:shr    = (flags >> 4u32);  // 0x0FF0
```

## Common Patterns

```aria
// Set bit
flags = (flags | (1u32 << bit));

// Clear bit
flags = (flags & (~(1u32 << bit)));

// Toggle bit
flags = (flags ^ (1u32 << bit));

// Check bit
bool:is_set = ((flags & (1u32 << bit)) != 0u32);
```
