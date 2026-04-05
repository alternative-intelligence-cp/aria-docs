# Why Verification?

## What SMT Verification Does

Aria's compiler integrates the Z3 SMT (Satisfiability Modulo Theories) solver to
**prove program properties at compile time**. Instead of discovering bugs at runtime
through testing, the solver mathematically proves that certain classes of bugs
*cannot exist* in your program.

When a property is:
- **Proven** — the compiler eliminates the runtime check entirely (free performance)
- **Disproven** — a compile-time diagnostic with a counterexample is emitted
- **Unknown** — the runtime check is preserved (conservative safety)

## What It Can Prove

| Domain | What It Checks | Example |
|--------|---------------|---------|
| Rules & limits | Values satisfy constraints | `limit<Positive> int32:x = 42` — proves 42 > 0 |
| Function contracts | Preconditions/postconditions hold | `requires b != 0` — proves callers never pass 0 |
| Integer overflow | Arithmetic stays in bounds | `a + b` won't exceed INT32_MAX |
| Division by zero | Divisor is non-zero | `a / b` where b is Rules-constrained |
| Data races | Shared variables are properly guarded | mutex lock/unlock pairs around writes |
| Null safety | Optionals are populated | `??` operator on provably non-nil values |
| Memory safety | No use-after-free | `wild` pointers used only while live |

## Verification vs Testing

| | Testing | Verification |
|---|---------|-------------|
| **Coverage** | Checks specific inputs | Proves for ALL possible inputs |
| **False negatives** | Misses edge cases | Mathematically complete (when Proven) |
| **Cost** | Runtime | Compile time |
| **Failure mode** | Runtime crash | Compile-time error |
| **Limitation** | Can't prove absence of bugs | Scope limited to what solver can reason about |

Testing confirms your program works for the cases you thought of.
Verification proves your program works for cases you *didn't* think of.

**Use both.** Testing validates behavior; verification eliminates entire classes of bugs.

## When to Use Verification

- **Public APIs** — Add `requires`/`ensures` contracts to functions others will call
- **System boundaries** — Apply `limit<Rules>` where untrusted data enters
- **Critical invariants** — Use `prove()` for properties that must always hold
- **Performance-sensitive paths** — Use `--smt-opt` to eliminate runtime checks
- **Concurrent code** — Use `--verify-concurrency` to catch data races at compile time

## Enabling Verification

```bash
# Enable all verification
ariac program.aria -o program --verify

# Individual domains
ariac program.aria --verify-contracts    # Just contracts
ariac program.aria --verify-overflow     # Just overflow
ariac program.aria --verify-concurrency  # Just data races

# Verification + optimization (recommended for release builds)
ariac program.aria --smt-opt             # Implies --verify, eliminates proven checks

# Control verification depth (v0.14.3+)
ariac program.aria --verify --verify-level=1   # Fast (Rules only)
ariac program.aria --verify --verify-level=2   # Standard (+ contracts, overflow)
ariac program.aria --verify --verify-level=3   # Thorough (all phases, full proofs)
```

## How It Works (Overview)

All Z3 work happens between type checking and IR generation. The compiler:

1. Walks the AST looking for verifiable properties
2. Translates each property to a Z3 formula
3. Asserts the *negation* of the property
4. If Z3 proves the negation is unsatisfiable → the property holds for all inputs
5. Results flow to codegen, which emits faster code for proven-safe operations

This is a standard **negation-based proof** technique: if there's no way the property
can fail, then it always holds.

## Next

- [Rules and Limits](02_rules_and_limits.md) — constraining values
- [Contracts](03_contracts.md) — design by contract
- [Optimizations](04_optimizations.md) — performance from proofs
