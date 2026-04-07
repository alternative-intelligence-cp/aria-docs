# JIT Compilation in Aria

## Overview

Aria includes a built-in JIT assembler for runtime code generation. The JIT subsystem builds on two foundations:

- **WildX** (`wildx_alloc.cpp`) — W⊕X memory management with ASLR, guard pages, code signing, and quota enforcement
- **Assembler** (`assembler.cpp`) — x86-64 instruction encoder with label backpatching, register allocation, peephole optimization, and instruction selection

The JIT is accessible from Aria code through the `jit` stdlib package, which provides FFI bindings to the C++ assembler API.

## Architecture

```
┌──────────────────────────────────────────────┐
│               Aria Source Code               │
│         use jit;  use wildx;                 │
├──────────────────────────────────────────────┤
│           jit.aria FFI Bindings              │
│    81 bindings + 17 helpers + constants      │
├──────────────────────────────────────────────┤
│           Assembler Pipeline                 │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ IR Queue │→ │ Peephole │→ │ Liveness  │  │
│  │ (lazy)   │  │ Optimizer│  │ Analysis  │  │
│  └──────────┘  └──────────┘  └───────────┘  │
│        ↓              ↓             ↓        │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ Linear   │→ │ Insn     │→ │ Machine   │  │
│  │ Scan RA  │  │Selection │  │ Code Emit │  │
│  └──────────┘  └──────────┘  └───────────┘  │
├──────────────────────────────────────────────┤
│        WildX — W⊕X Memory Manager            │
│  ASLR | Guard Pages | Code Signing | Quota   │
├──────────────────────────────────────────────┤
│              x86-64 Hardware                 │
└──────────────────────────────────────────────┘
```

## Instruction Set (v0.7.2+)

The assembler supports 45+ x86-64 instructions across multiple categories:

### Integer
- **Data movement:** `MOV r64, imm64` / `MOV r64, r64`
- **Arithmetic:** `ADD`, `SUB`, `IMUL` (r64,r64 and r64,imm32)
- **Bitwise:** `XOR`, `AND`, `OR`, `NOT`, `NEG`
- **Shifts:** `SHL`, `SHR`, `SAR` with imm8
- **Compare:** `CMP r64, r64` / `CMP r64, imm32`
- **Stack:** `PUSH`, `POP`
- **Flow:** `JMP`, `JE/JNE/JL/JLE/JG/JGE/JB/JBE/JA/JAE`, `RET`
- **Call:** `CALL r64`, `CALL label`, `CALL abs`

### Floating-Point (SSE2)
- `MOVSD` (reg-reg, load, store), `ADDSD`, `SUBSD`, `MULSD`, `DIVSD`, `UCOMISD`
- XMM0–XMM15 registers

### SIMD (SSE)
- `MOVAPS` (reg-reg, aligned load/store), `ADDPS`, `MULPS`
- Packed float32 (4x f32) operations

### Memory
- `MOV r64, [base+offset]` (load), `MOV [base+offset], r64` (store)
- `LEA r64, [base+offset]` (address computation)
- `store_local` / `load_local` (RBP-relative stack frame)

## Register Allocator (v0.7.3+)

The JIT includes a linear scan register allocator for automatic register assignment:

```aria
extern func:asm_create = int64();
extern func:asm_vreg_new_gpr = int64(int64:ctx);
extern func:asm_mov_r64_imm64 = NIL(int64:ctx, int64:reg, int64:val);
extern func:asm_add_r64_r64 = NIL(int64:ctx, int64:dst, int64:src);
extern func:asm_mov_r64_r64 = NIL(int64:ctx, int64:dst, int64:src);
extern func:asm_ret = NIL(int64:ctx);
extern func:asm_finalize = int64(int64:ctx);
extern func:asm_execute = int64(int64:guard);

func:main = int32() {
    int64:a = asm_create();
    int64:v0 = asm_vreg_new_gpr(a);
    int64:v1 = asm_vreg_new_gpr(a);

    drop asm_mov_r64_imm64(a, v0, 10i64);
    drop asm_mov_r64_imm64(a, v1, 32i64);
    drop asm_add_r64_r64(a, v0, v1);
    drop asm_mov_r64_r64(a, 0i64, v0);   // REG_RAX = 0
    drop asm_ret(a);

    int64:guard = asm_finalize(a);
    int64:result = asm_execute(guard);
    // result == 42
    exit 0;
};

func:failsafe = int32(tbb32:err) { exit 1; };
```

**Features:**
- 12 allocatable GPRs (RAX, RCX, RDX, RSI, RDI, R8, R9, RBX, R12-R15)
- 14 allocatable XMMs (XMM0-XMM13)
- Automatic spill/reload when registers are exhausted
- Auto prologue/epilogue when callee-saved registers are needed
- Mixed physical + virtual register support

## Peephole Optimizer (v0.7.4)

The JIT runs a peephole optimization pass on the IR before register allocation:

| Pattern | Optimization | Bytes Saved |
|---|---|---|
| `MOV r, 0` | `XOR r, r` | 6–7 |
| `MOV r, r` | eliminated | 3–4 |
| `ADD r, 0` / `SUB r, 0` | eliminated | 7 |
| `SHL r, 0` / `SHR r, 0` | eliminated | 4 |
| `MOV r, X; MOV r, Y` | dead store eliminated | 10 |
| `MOV r, 2^n; IMUL d, r` | `SHL d, n` | ~7 |
| `XOR r, r; ADD r, s` | `MOV r, s` | 3–4 |

Statistics available via `aria_asm_peephole_stats()`.

## Instruction Selection (v0.7.4)

During code emission, the allocator selects optimal machine encodings:

| IR Instruction | Selected Encoding | Bytes Saved |
|---|---|---|
| `MOV_IMM64` (value ≤ 0xFFFFFFFF) | `MOV r32, imm32` | 4–5 |
| `CMP r, 0` | `TEST r, r` | 4 |
| `ADD r, 1` | `INC r` | 4 |
| `SUB r, 1` | `DEC r` | 4 |
| `ADD/SUB r, imm8` | imm8 form | 3 |

Statistics available via `aria_asm_insn_sel_stats()`.

## Profiling Integration (v0.7.4)

JIT code can be registered with Linux `perf` for profiling:

```aria
jit.asm_perf_map_register(code_ptr, code_size, "my_jit_function");
// Now visible in: perf record -p <pid> && perf report
```

This writes to `/tmp/perf-<pid>.map` in the format expected by `perf`.

## WildX Security (v0.7.1)

All JIT code runs through WildX's security pipeline:
- **ASLR:** Random mmap hints for JIT pages
- **Guard pages:** `PROT_NONE` sentinels around executable regions
- **Code signing:** FNV-1a hash verified before every execution
- **W⊕X:** Strict WRITABLE → EXECUTABLE state machine (never both)
- **Quota:** Default 64MB, configurable via `aria_wildx_set_quota()`
- **Audit logging:** `--wildx-audit` flag for ALLOC/SEAL/EXEC/FREE events

## Multi-Architecture (v0.7.4)

Architecture detection and abstraction:

```aria
let arch = jit.asm_get_arch();       // ASM_ARCH_X86_64 or ASM_ARCH_AARCH64
let ok = jit.asm_arch_supported(arch); // true on x86-64
```

AArch64 backend is stubbed for future implementation. The architecture abstraction layer supports querying the current target and checking support before code generation.

## Safety

JIT compilation is a **Layer 3 (raw)** operation — it bypasses all safety guarantees. Executable memory is:
- Not bounds-checked
- Not type-checked
- A security risk if used with untrusted input

Always use WildX guards and code signing for JIT code.

## Related

- [memory_model/wild.md](../memory_model/wild.md) — unmanaged memory
- [types/pointer.md](../types/pointer.md) — pointer operations
- [safety_layers.md](safety_layers.md) — safety layer definitions
