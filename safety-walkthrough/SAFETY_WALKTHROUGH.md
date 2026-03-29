# Safety-Critical Development in Aria: A Walkthrough

**Version:** 0.3.4  
**Date:** 2026-04-10  
**Status:** Living document  

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [The Application: Medical Infusion Pump](#2-the-application-medical-infusion-pump)
3. [Feature 1: failsafe — The Mandatory Safety Net](#3-feature-1-failsafe--the-mandatory-safety-net)
4. [Feature 2: Result\<T\> — No Silent Failures](#4-feature-2-resultt--no-silent-failures)
5. [Feature 3: TBB — Sticky Error Propagation](#5-feature-3-tbb--sticky-error-propagation)
6. [Feature 4: limit\<Rules\> — Value Constraints with Z3 Verification](#6-feature-4-limitrules--value-constraints-with-z3-verification)
7. [Feature 5: Borrow Semantics — Compile-Time Memory Safety](#7-feature-5-borrow-semantics--compile-time-memory-safety)
8. [Feature 6: Emergency Operators — ?! and !!!](#8-feature-6-emergency-operators---and-)
9. [Feature 7: wild — Controlled Unsafe Access](#9-feature-7-wild--controlled-unsafe-access)
10. [Putting It Together: The Complete Pump Controller](#10-putting-it-together-the-complete-pump-controller)
11. [Z3 Verification in Practice](#11-z3-verification-in-practice)
12. [Comparison with Other Approaches](#12-comparison-with-other-approaches)
13. [Summary](#13-summary)

---

## 1. Introduction

Safety-critical software — code that controls medical devices, aircraft, nuclear
reactors, and autonomous vehicles — has a fundamental problem: **the consequences
of a bug are not a stack trace, they are injury or death.**

Traditional approaches to this problem fall into two camps:

1. **Restricted subsets** (MISRA C, SPARK/Ada): take an existing language, ban
   the dangerous parts, and add external analysis tools.
2. **Formal methods bolted on** (Frama-C, CBMC): write normal C, then attempt
   to prove properties after the fact with separate tools.

Both approaches share a weakness: **safety is an afterthought.** The language
itself does not enforce safety — external tools and coding standards do. A
developer who forgets to run the checker, or who disables a warning, is back to
writing unsafe code.

Aria takes a different approach: **safety is structural.** The compiler itself
refuses to produce a binary that lacks mandatory error handling, that uses
unchecked results, or that violates declared constraints. You cannot forget to
handle errors because the compiler will not let you. You cannot silently overflow
a safety-critical integer because the type system traps it.

This walkthrough demonstrates Aria's safety features through a realistic
application — a medical drug infusion pump controller — building up from
individual features to a complete, compilable program.

### Who This Document Is For

- Safety engineers evaluating Aria for safety-critical domains
- Language researchers comparing error-handling and verification models
- Developers curious about Aria's safety guarantees and how they differ from
  Rust, Ada/SPARK, or traditional C/C++ approaches

### Prerequisites

- The `ariac` compiler (v0.3.4+)
- Basic familiarity with imperative programming
- No prior Aria experience required

### Companion Files

Each section references a standalone, compilable example in the `examples/`
directory:

| File | Feature |
|------|---------|
| `01_failsafe_basics.aria` | Mandatory failsafe handler |
| `02_result_handling.aria` | Result\<T\> with pass/fail |
| `03_tbb_propagation.aria` | TBB sticky error types |
| `04_limit_rules.aria` | limit\<Rules\> constraints |
| `05_borrow_safety.aria` | Borrow semantics ($$i/$$m) |
| `06_emergency_operators.aria` | ?! and !!! operators |
| `07_infusion_pump.aria` | Complete pump controller |

---

## 2. The Application: Medical Infusion Pump

A drug infusion pump is a device that delivers fluids — typically medication —
into a patient's body at a controlled rate. Infusion pump software failures have
caused real patient deaths and are a frequent subject of FDA recalls.

Common failure modes include:

- **Overdose:** software calculates too high a flow rate
- **Silent sensor failure:** a disconnected sensor returns stale data
- **Arithmetic overflow:** a rate calculation wraps around to a plausible but
  wrong value
- **Memory corruption:** a pointer bug overwrites the dosage register
- **Missing error handling:** an error return is ignored, execution continues
  with garbage data

Our example pump controller will demonstrate how each of Aria's safety features
addresses one or more of these failure modes.

The controller performs a single infusion cycle:

```
1. Load and validate patient data
2. Read sensors (temperature, pressure, flow rate)
3. Calculate weight-based dosage using TBB arithmetic
4. Apply limit<Rules> to constrain the final dose
5. Deliver or enter safe mode based on error count
```

---

## 3. Feature 1: failsafe — The Mandatory Safety Net

### The Problem

In C, error-handling functions like `atexit()` or signal handlers are optional.
A developer can write a program with no error handling at all, and the compiler
will happily produce a binary. In safety-critical systems, this is unacceptable.

### Aria's Solution

Every Aria program **must** define a `failsafe` function. The compiler checks
for its existence and refuses to compile if it is missing. This is enforced at
the analysis phase (BUG-005 fix, v0.3.4), not by convention or linting.

### Syntax

```aria
func:failsafe = int32(tbb32:err) {
    // err: the TBB error code that triggered this handler
    // Body MUST call exit() — failsafe cannot "resume"
    exit(1);
};
```

### Key Properties

| Property | Detail |
|----------|--------|
| **Mandatory** | Compiler error if missing |
| **Signature** | `int32(tbb32:err)` — fixed, not configurable |
| **Error code** | `tbb32` — uses TBB type so it cannot be accidentally corrupted |
| **Return** | Must call `exit(code)` — this is a terminal handler |
| **Invocation** | Called automatically on unhandled errors, or explicitly via `!!!` |

### Example

*(See `examples/01_failsafe_basics.aria`)*

```aria
func:failsafe = int32(tbb32:err) {
    print("EMERGENCY: failsafe triggered\n");
    exit(1);
};

func:main = int32() {
    print("System online — failsafe registered.\n");
    int32:status = 0;
    if (status == 0) {
        print("All subsystems nominal.\n");
    }
    exit(0);
};
```

### Why This Matters

In traditional safety-critical C (IEC 62304, DO-178C), external coding
standards mandate error handling — but the compiler itself does not enforce them.
Aria makes the guarantee structural: **no failsafe, no binary.**

---

## 4. Feature 2: Result\<T\> — No Silent Failures

### The Problem

In C, functions signal errors through return codes, `errno`, or null pointers.
All of these can be silently ignored:

```c
FILE *f = fopen("data.bin", "r");  // f might be NULL
fread(buf, 1, 100, f);             // UB if f is NULL — no compiler warning
```

In C++, exceptions can be unhandled. In Go, errors are values that can be
discarded with `_`.

### Aria's Solution

Functions that can fail return `Result<T>` implicitly. The caller **must** check
`.is_error` before accessing `.value` — the compiler enforces this rule (called
"no-checky-no-val"). Accessing `.value` without a prior `.is_error` check is a
compile error.

### Syntax

```aria
// Declaring a function that can fail:
func:read_sensor = int32() {
    pass(42i32);    // success — wraps 42 in Result<int32>
    fail(1i32);     // error   — wraps 1 as error code
};

// Calling and checking:
Result<int32>:r = read_sensor();
if (r.is_error) {
    int32:code = r.error;    // safe — checked first
} else {
    int32:val = r.value;     // safe — checked first
}
```

### Error Codes

Aria defines standardized error code ranges:

| Range | Meaning | Examples |
|-------|---------|----------|
| `ERR` | Unknown/unclassified | TBB min sentinel |
| `0` | No error / success | — |
| `-1` to `-16` | System errors | -1 SYS_GENERAL, -3 SYS_IO, -7 SYS_OVERFLOW |
| `1` to `10` | User-defined errors | 1 USR_GENERAL, application-specific |

### Example

*(See `examples/02_result_handling.aria`)*

```aria
func:read_temperature = int32() {
    pass(365i32);    // 36.5 °C in tenths
};

func:read_pressure = int32() {
    fail(1i32);     // sensor offline
};

func:main = int32() {
    Result<int32>:temp = read_temperature();
    if (temp.is_error) {
        print("Temperature sensor failed\n");
    } else {
        int32:t = temp.value;
        print("Temperature OK\n");
    }

    Result<int32>:pres = read_pressure();
    if (pres.is_error) {
        int32:code = pres.error;
        print("Pressure sensor offline\n");
    } else {
        int32:p = pres.value;
        print("Pressure nominal\n");
    }
    exit(0);
};
```

### Why This Matters

The compiler guarantees at the type level that **every error path is explicitly
handled**. There is no mechanism to silently discard an error — no `_`, no
unchecked return codes, no unhandled exceptions.

---

## 5. Feature 3: TBB — Sticky Error Propagation

### The Problem

In IEEE 754 floating-point, `NaN` propagates through arithmetic: `NaN + 1 = NaN`.
This is valuable — it prevents corrupted data from producing plausible results.
But integers have no equivalent. In C, `INT_MAX + 1` is undefined behavior. In
most languages, integer overflow silently wraps to a negative number.

This is catastrophic in safety-critical systems. Imagine a dosage calculation:

```
dose = patient_weight * rate_per_kg
```

If `patient_weight` overflows to a small positive number due to a sensor bug,
the dose might appear valid but be dramatically wrong.

### Aria's Solution

TBB (Trusted Balanced Bitfield) types are integers with a **sentinel error
value** at the type minimum. Once a TBB variable enters the ERR state — through
overflow, underflow, or division by zero — it **stays** in ERR through all
subsequent arithmetic.

| Type | ERR Sentinel |
|------|-------------|
| `tbb8` | -128 |
| `tbb16` | -32768 |
| `tbb32` | -2147483648 |
| `tbb64` | -9223372036854775808 |

### Sticky Behavior

The critical property is that **errors never heal:**

```
ERR + 1     = ERR      (not a valid number)
ERR * 0     = ERR      (not zero — errors don't annihilate!)
ERR - ERR   = ERR      (not zero — errors don't cancel!)
1 + ERR     = ERR      (commutativity preserved)
```

This is fundamental. In C, `INT_MIN * 0 = 0`, which means an overflow error can
silently disappear. TBB guarantees this cannot happen.

### Example

*(See `examples/03_tbb_propagation.aria`)*

```aria
func:main = int32() {
    // Normal arithmetic works as expected
    tbb32:sensor_a = 100;
    tbb32:sensor_b = 200;
    tbb32:sum = sensor_a + sensor_b;    // 300

    // Overflow produces ERR
    tbb32:big = 2147483647;
    tbb32:one = 1;
    tbb32:overflow = big + one;          // ERR (-2147483648)

    // ERR is sticky — it NEVER heals
    tbb32:still_bad = overflow + sensor_a;  // ERR
    tbb32:zero = 0;
    tbb32:mul_zero = overflow * zero;       // ERR (not zero!)
    tbb32:self_sub = overflow - overflow;   // ERR (not zero!)

    exit(0);
};
```

### Application: Dosage Calculation

In the infusion pump controller, the dose calculation uses TBB arithmetic:

```aria
tbb32:w = weight_kg;
tbb32:base = rate_per_kg;
tbb32:dose = w * base;

tbb32:err_sentinel = -2147483648;
if (dose == err_sentinel) {
    fail(-7);   // SYS_OVERFLOW — propagate error up
}
```

If **any** input is corrupted, the output is guaranteed to be ERR. The pump
cannot deliver a corrupted dose because the ERR sentinel will be caught by the
check, or by the `limit<Rules>` constraint (which rejects ERR values since
they are negative and outside the valid dose range).

---

## 6. Feature 4: limit\<Rules\> — Value Constraints with Z3 Verification

### The Problem

Safety-critical systems have **value constraints** that must never be violated:
a drug dose must not exceed the maximum, a motor speed must stay below the
redline, a reactor temperature must remain within safe bounds.

In traditional languages, these constraints live in comments, documentation,
or runtime assertions that can be forgotten, disabled, or bypassed.

### Aria's Solution

Aria's `Rules` declarations define named constraints. The `limit<RuleName>`
annotation on a variable means the compiler and runtime enforce those constraints
at every assignment. With Z3 verification (`--verify`), the compiler can
**prove** constraints at compile time — before the program ever runs.

### Syntax

```aria
// Scalar constraint: dose must be 1–5000 mg
Rules:r_safe_dose = {
    $ > 0,
    $ <= 5000
};

// Struct field constraint: patient must be adult
Rules<Patient>:r_adult = {
    $.age_years >= 18,
    $.weight_kg >= 30
};

// Cascading: pediatric dose inherits safe dose + adds upper bound
Rules:r_pediatric = {
    limit<r_safe_dose>,
    $ <= 500
};

// Apply to a variable:
limit<r_safe_dose> int32:dose = 250;   // OK — 250 is in [1, 5000]
limit<r_safe_dose> int32:bad  = 9999;  // Runtime error → failsafe
```

### Z3 Verification

When compiled with `--verify`, the compiler translates Rules constraints into Z3
SMT assertions and attempts to prove them statically:

```bash
ariac --verify 04_limit_rules.aria
```

If the compiler can prove a constraint is always satisfied, no runtime check is
needed. If it finds a counter-example, it reports the violating values. If it
can neither prove nor disprove, a runtime check is emitted as a safety net.

### Example

*(See `examples/04_limit_rules.aria`)*

```aria
Rules:r_safe_dose_mg = {
    $ > 0,
    $ <= 5000
};

Rules:r_pediatric_dose = {
    limit<r_safe_dose_mg>,
    $ <= 500
};

struct:Patient = {
    int32:weight_kg;
    int32:age_years;
};

Rules<Patient>:r_adult_patient = {
    $.weight_kg >= 30,
    $.age_years >= 18
};

func:main = int32() {
    limit<r_safe_dose_mg> int32:dose = 250;              // OK
    limit<r_pediatric_dose> int32:child_dose = 100;       // OK (cascade)

    stack Patient:pat = Patient{weight_kg: 80, age_years: 35};
    limit<r_adult_patient> Patient:adult = pat;           // OK
    exit(0);
};
```

### Application: Multi-Layer Dosage Protection

In the infusion pump, constraints form a multi-layer defense:

1. **Weight constraint** (`r_valid_weight`): patient weight 1–300 kg
2. **Dose constraint** (`r_safe_dose`): final dose 1–5000 mg
3. **Patient eligibility** (`r_eligible_patient`): age ≥ 18, weight ≥ 30 kg
4. **Flow constraint** (`r_safe_flow`): delivery rate 1–1000 mL/hr

A dosage that somehow passes the calculation check, passes the
patient-max-dose clamp, but falls outside 1–5000 mg would be caught by
`limit<r_safe_dose>` and trigger the failsafe handler.

### Why This Matters

Unlike runtime assertions in C (`assert(dose <= 5000)`) which can be compiled
out with `NDEBUG`, Aria's constraints are part of the type system. They cannot
be disabled. With Z3, they can be verified at compile time — providing the
strongest possible guarantee without runtime cost.

---

## 7. Feature 5: Borrow Semantics — Compile-Time Memory Safety

### The Problem

Memory corruption is the leading cause of security vulnerabilities (per CISA,
NSA, and Microsoft). In safety-critical systems, a corrupted pointer can
overwrite a dosage register, crash a flight controller, or cause a reactor SCRAM.

C and C++ provide no structural protection against:
- Use-after-free
- Data races from concurrent aliased mutation
- Dangling pointers

### Aria's Solution

Aria uses explicit borrow annotations with two rules enforced at compile time:

| Annotation | Meaning | Rule |
|-----------|---------|------|
| `$$i Type:ref = var;` | Immutable borrow | Any number may coexist |
| `$$m Type:ref = var;` | Mutable borrow | At most one; no $$i borrows on same variable |

This prevents the fundamental aliasing problem: you can never have a mutable
reference coexisting with any other reference to the same data.

### Example

*(See `examples/05_borrow_safety.aria`)*

```aria
func:main = int32() {
    int32:sensor_reading = 42;

    // Multiple immutable borrows — ALLOWED
    $$i int32:view_a = sensor_reading;
    $$i int32:view_b = sensor_reading;
    int32:sum = view_a + view_b;   // 84

    // Single mutable borrow — write-through
    int32:actuator_position = 0;
    $$m int32:control = actuator_position;
    control = 100;
    // actuator_position is now 100

    exit(0);
};
```

### Compile-Time Rejection

The compiler rejects conflicting borrows:

```aria
int32:x = 10;
$$i int32:reader = x;
$$m int32:writer = x;   // COMPILE ERROR: $$i borrow active on x
```

This is checked **at compile time** — no runtime overhead, no possibility of
the check being disabled.

### Application: Safe Patient Data Access

In the infusion pump, patient data is accessed through immutable borrows:

```aria
$$i int32:pat_weight = safe_weight;
$$i int32:pat_max = checked_patient.max_dose_mg;
```

This guarantees that the dosage calculation cannot accidentally modify patient
data. If a future developer tries to write through these references, the
compiler will reject the code.

---

## 8. Feature 6: Emergency Operators — ?! and !!!

### The Problem

Error handling in safety-critical systems must be both **convenient** (so
developers actually use it) and **auditable** (so reviewers can trace every
error path). Exception-based languages make error handling convenient but
difficult to audit. Return-code languages make error handling auditable but
tedious.

### Aria's Solution

Aria provides two operators that form a **two-level error escalation system**:

| Operator | Name | Level | Behavior |
|----------|------|-------|----------|
| `?!` | Emphatic unwrap | Local | Unwrap Result; use default on error |
| `!!!` | Direct failsafe | Global | Immediately invoke failsafe handler |

**Level 1 — `?!` (local recovery):** "I can handle this with a safe default."

```aria
Result<int32>:sensor = read_sensor();
int32:value = sensor ?! safe_default;
// On success: value = sensor.value
// On error:   value = safe_default
```

**Level 2 — `!!!` (global escalation):** "This is catastrophic. Emergency stop."

```aria
!!! -2;   // Immediately calls failsafe(-2), program terminates
```

### Example

*(See `examples/06_emergency_operators.aria`)*

```aria
// Level 1: Local recovery with ?!
Result<int32>:ok_reading = read_sensor_ok();
int32:val_ok = ok_reading ?! 50i32;    // val_ok == 98 (unwrapped)

Result<int32>:bad_reading = read_sensor_fail();
int32:val_bad = bad_reading ?! 50i32;  // val_bad == 50 (default)

// Level 2: Escalate after exhausting local recovery
if (consecutive_failures >= 3) {
    !!! -2;   // trigger failsafe — emergency shutdown
}
```

### Application: Sensor Fallback Cascade

In the infusion pump, ?! provides graceful degradation:

```aria
// Temperature: use body-temp default if sensor fails
int32:temperature = temp_result ?! 370i32;

// Pressure: use 100 mmHg default if sensor fails  
int32:pressure = pres_result ?! 100i32;

// Flow rate: use 0 (safe stop) if sensor fails
int32:flow = flow_result ?! 0i32;
```

Each sensor has a domain-specific safe default. If too many sensors fail, the
controller enters safe mode rather than delivering an uncertain dose.

### Auditability

Both operators are syntactically distinctive — `?!` and `!!!` stand out in code
review. A security auditor can grep for these tokens to find every error recovery
and escalation point in the codebase.

---

## 9. Feature 7: wild — Controlled Unsafe Access

### The Problem

Safety-critical systems sometimes need to interact with hardware registers,
DMA buffers, or foreign-function interfaces that require raw pointer access.
In Rust, this is `unsafe {}`. In Ada, it is `pragma Import`. The challenge is
making unsafe code **visible and auditable** while still allowing it when needed.

### Aria's Approach

The `wild` keyword declares raw, unmanaged pointers that bypass the borrow
checker:

```aria
wild void->:hw_reg = @extern("get_hardware_register", 0x40001000);
```

`wild` pointers are:
- **Explicitly marked** — they cannot be confused with safe references
- **Grep-able** — every use of `wild` in a codebase is an audit point
- **Unrestricted** — they support pointer arithmetic, casts, and FFI

The philosophy is the same as Rust's `unsafe`: make the dangerous parts visible
so reviewers know exactly where to focus.

### When To Use wild

| Use Case | Example |
|----------|---------|
| Hardware register access | `wild uint32->:reg = ...;` |
| FFI with C libraries | `extern func:malloc = wild void->(int64:size);` |
| DMA buffer management | `wild uint8->:dma_buf = ...;` |
| Performance-critical paths | Manual memory layouts |

In the infusion pump controller, `wild` would be used for the actual hardware
interface layer — reading ADC registers, controlling valve GPIOs, and
communicating over CAN bus. The example code uses simulated functions instead
to keep the demonstrations portable and compilable without hardware.

---

## 10. Putting It Together: The Complete Pump Controller

*(See `examples/07_infusion_pump.aria`)*

The complete infusion pump controller combines all features into a coherent
program:

### Architecture

```
┌─────────────────────────────────────────────────┐
│                   failsafe()                     │
│            Last line of defense                  │
│     Closes valves, sounds alarm, logs error      │
└────────────────────┬────────────────────────────┘
                     │ invoked by !!! or unhandled error
┌────────────────────┴────────────────────────────┐
│                    main()                        │
│                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────┐│
│  │ Load Patient │→│ Read Sensors │→│ Calculate ││
│  │   Data       │  │  (Result<T>) │  │  Dosage  ││
│  │              │  │              │  │  (TBB)   ││
│  │ limit<Rules> │  │ ?! defaults  │  │          ││
│  │ $$i borrows  │  │              │  │          ││
│  └─────────────┘  └─────────────┘  └────┬─────┘│
│                                          │      │
│                                   ┌──────┴─────┐│
│                                   │  Constrain  ││
│                                   │ limit<Rules>││
│                                   │             ││
│                                   │ Deliver or  ││
│                                   │ Safe Mode   ││
│                                   └─────────────┘│
└──────────────────────────────────────────────────┘
```

### Safety Layers in the Code

| Layer | Feature | What It Prevents |
|-------|---------|-----------------|
| 1 | `failsafe()` | Unhandled errors escaping the program |
| 2 | `Result<T>` | Silent sensor failures |
| 3 | `?!` operator | Sensor failure without safe default |
| 4 | TBB arithmetic | Overflow producing plausible wrong dosage |
| 5 | `limit<Rules>` | Out-of-range dosage reaching the pump |
| 6 | `$$i` borrows | Accidental mutation of patient data |
| 7 | Error counting | Delivering dose with too many unknowns |

### Walkthrough of Key Sections

**Patient data validation:**
```aria
stack Patient:patient = Patient{
    id: 1042, weight_kg: 72, age_years: 45, max_dose_mg: 2000
};
limit<r_eligible_patient> Patient:checked_patient = patient;
limit<r_valid_weight> int32:safe_weight = checked_patient.weight_kg;

$$i int32:pat_weight = safe_weight;
int32:max_dose_val = checked_patient.max_dose_mg;
$$i int32:pat_max = max_dose_val;
```

Three constraints are applied before any calculation begins: the patient must
be eligible (age/weight), the weight must be in human range, and subsequent
access is read-only through immutable borrows.

**Sensor reading with local recovery:**
```aria
Result<int32>:temp_result = read_temperature();
int32:temperature = temp_result ?! 370i32;   // default: 37.0 °C

Result<int32>:flow_result = read_flow_rate();
int32:flow = flow_result ?! 0i32;            // default: stop delivery
```

Each sensor has a domain-specific default. The flow sensor defaults to 0 (no
delivery) — the safest possible fallback for a failed flow measurement.

**TBB-safe dosage calculation:**
```aria
tbb32:w = weight_kg;
tbb32:base = base_dose_per_kg;
tbb32:dose = w * base;

tbb32:err_sentinel = -2147483648;
if (dose == err_sentinel) {
    fail(-7);   // SYS_OVERFLOW
}
```

If either input is corrupted, the TBB multiplication produces ERR, which is
caught by the sentinel check. The error cannot propagate silently.

**Final constraint enforcement:**
```aria
limit<r_safe_dose> int32:final_dose = clamped;
```

Even after clamping to the patient's maximum, the dose must still pass the
global safe-dose rule (1–5000 mg). This is the last automated check before
delivery.

**Delivery decision:**
```aria
if (errors >= 2) {
    // Multiple sensor failures — do NOT deliver
    exit(0);   // safe mode
}
```

The controller counts errors throughout the cycle. Two or more failures trigger
safe mode — no drug is delivered.

---

## 11. Z3 Verification in Practice

Aria v0.3.4 includes a built-in Z3 SMT solver integration that can verify
limit\<Rules\> constraints and function contracts at compile time.

### Usage

```bash
# Verify all constraints
ariac --verify 07_infusion_pump.aria

# Generate a detailed verification report
ariac --verify --verify-report pump_report.txt 07_infusion_pump.aria

# Verify specific categories
ariac --verify-contracts 07_infusion_pump.aria     # requires/ensures
ariac --verify-overflow  07_infusion_pump.aria      # integer overflow
```

### What Z3 Proves

The Z3 verifier operates in three phases:

1. **Phase 1 — Constraint Verification:** Checks that every `limit<Rules>`
   assignment satisfies its constraints. For constant values, this is a direct
   proof. For computed values, Z3 searches for counter-examples.

2. **Phase 2 — Contract Verification:** Checks `requires` (preconditions)
   and `ensures` (postconditions) on function signatures.

3. **Phase 3 — Overflow Analysis:** Checks integer arithmetic for potential
   overflow, especially on safety-critical types.

### Verification Report Example

```
=== Z3 Verification Report ===
File: 07_infusion_pump.aria

[PASS] limit<r_safe_dose> at line 156: dose ∈ [1, 5000] — PROVED
[PASS] limit<r_valid_weight> at line 134: weight ∈ [1, 300] — PROVED
[PASS] limit<r_eligible_patient> at line 131: age >= 18, weight >= 30 — PROVED
[PASS] limit<r_safe_flow> — no violations found

Constraints verified: 4/4
Counter-examples: 0
Verification time: 0.023s
```

### Comparison with External Tools

| Approach | When | Integrated | Incremental |
|----------|------|-----------|-------------|
| SPARK/Ada | Compile time | ✓ | ✓ |
| Frama-C/WP | Post-hoc | ✗ | ✗ |
| CBMC | Post-hoc | ✗ | ✗ |
| **Aria + Z3** | **Compile time** | **✓** | **✓** |

Aria's Z3 integration runs as part of the normal compilation pipeline. No
separate tool invocation, no configuration files, no annotation language
different from the source language.

---

## 12. Comparison with Other Approaches

### How Aria Compares to Safety-Critical Alternatives

| Feature | C (MISRA) | Ada/SPARK | Rust | **Aria** |
|---------|-----------|-----------|------|----------|
| Mandatory error handler | ✗ Convention | ✗ Optional | ✗ Optional | **✓ Compiler-enforced** |
| Checked results | ✗ Return codes | ✓ Exceptions | ✓ Result\<T\> | **✓ Result\<T\> + no-checky-no-val** |
| Integer overflow safety | ✗ UB | ✓ Constraint_Error | ✓ Panic (or wrap) | **✓ TBB sticky sentinel** |
| Value constraints | ✗ assert() | ✓ Subtypes | ✗ (crate-level) | **✓ limit\<Rules\> + Z3** |
| Memory safety | ✗ Manual | ✓ (no pointers) | ✓ Borrow checker | **✓ Borrow ($$i/$$m) + wild** |
| Formal verification | External (Frama-C) | SPARK prover | External (Kani) | **Built-in Z3** |
| Unsafe escape hatch | Entire language | `pragma Import` | `unsafe {}` | **`wild` keyword** |

### Key Differentiators

1. **failsafe is mandatory, not optional.** No other mainstream language refuses
   to compile a program that lacks a top-level error handler.

2. **TBB has no equivalent.** Rust panics on overflow (in debug) or wraps (in
   release). Ada raises Constraint_Error (which can be caught and ignored). TBB
   errors are **sticky and inescapable** — they propagate through all arithmetic
   forever.

3. **limit\<Rules\> with Z3 is first-class.** Ada subtypes are similar in spirit
   but lack SMT verification. Rust has no equivalent — constraint checking
   requires external crates and runtime assertions.

4. **Two-level error escalation (?! / !!!) is distinct from try/catch.** The
   operators are syntactically loud, making error paths visible in code review.

---

## 13. Summary

Aria's safety model is built on a simple insight: **safety features that can be
forgotten will be forgotten.** Every feature described in this walkthrough is
either mandatory (failsafe), compiler-enforced (Result checking, borrow rules),
or structurally integrated (TBB, limit\<Rules\>, Z3).

### The Safety Stack

```
Layer 7:  limit<Rules> + Z3 ─── Provable value constraints
Layer 6:  $$i / $$m borrows ─── Compile-time memory safety
Layer 5:  TBB types ─────────── Sticky error propagation
Layer 4:  ?! operator ──────── Local recovery with safe defaults
Layer 3:  Result<T> ─────────── Mandatory explicit error handling
Layer 2:  !!! operator ──────── Global emergency escalation
Layer 1:  failsafe() ────────── Mandatory last line of defense
```

Each layer catches failures that escape the layer above it. A corrupted sensor
reading might be caught by `?!` (Layer 4); if not, by TBB propagation (Layer 5);
if not, by `limit<Rules>` (Layer 7); if not, by `failsafe` (Layer 1).

### Compiling the Examples

```bash
# Build the compiler
cd aria/build && cmake .. && make -j$(nproc)

# Compile and run individual examples
./ariac ../../aria-docs/safety-walkthrough/examples/01_failsafe_basics.aria
./01_failsafe_basics

# Compile with Z3 verification
./ariac --verify ../../aria-docs/safety-walkthrough/examples/04_limit_rules.aria

# Compile the complete pump controller
./ariac ../../aria-docs/safety-walkthrough/examples/07_infusion_pump.aria
./07_infusion_pump
```

### Next Steps

- **For safety engineers:** Review the pump controller code and compare it to
  your current development practices. Consider which failure modes Aria prevents
  that your current toolchain does not.
- **For language researchers:** The formal comparison in Section 12 is intended
  as a starting point. We welcome analysis from the verification and safety
  communities.
- **For developers:** Clone the [Aria repository](https://github.com/alternative-intelligence-cp/aria),
  build the compiler, and try compiling and modifying the example files. Break
  things. See what the compiler catches.

---

*This document is part of the Aria documentation suite. For language specification,
see `specs/aria_specs.txt`. For getting started, see `GETTING_STARTED.md`.*
