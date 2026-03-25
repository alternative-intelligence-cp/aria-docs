# Core Design Principle: No Symbol Ambiguity

## The Problem with Context-Dependent Symbols

**Bad Example: C's `*` operator**
```c
int x = 8 * 8;        // Multiplication
int* ptr;             // Pointer type
int y = *ptr;         // Dereference
int* a, b;            // a is pointer, b is int (WTF?)
```

One symbol, **FOUR different meanings** depending on context. You have to mentally parse the entire expression to know what `*` means. This is cognitive overhead that serves **zero** purpose.

## Aria's Principle: One Symbol, One Meaning

**If two different things should happen, use different symbols.**

### Aria's Clear Syntax

```aria
// Arithmetic
int32:product = 8i32 * 8i32;  // * ONLY for multiplication

// Pointer types - different symbol!
wild int32->:ptr = @x;         // -> for pointer type (in declarations)
int32:value = ptr->deref;      // -> for pointer member access (C-style)

// Address-of - different symbol!
wild int32->:addr = @x;        // @ for taking address

// No ambiguity, ever
```

### Type Belongs With Type, Not Name

**C's Confusion:**
```c
int *a, b, c;  // a is int*, b and c are int (SURPRISE!)
int* a, b, c;  // Same thing, but looks like all are pointers
```

**Aria's Clarity:**
```aria
int32:a = 10i32;              // Type on left, name on right
wild int32->:ptr = @a;        // Pointer is part of TYPE
wild int32->:p1 = @x;
wild int32->:p2 = @y;         // Both clearly pointers
```

The `->` is part of the **type specification**, not some weird syntax trick on the name.

## Examples of Ambiguity We Avoid

### 1. Const Madness (C++)

**C++ has multiple meanings for const:**
```cpp
const int x = 10;           // Can't change x
const int* ptr1;            // Can't change *ptr1, CAN change ptr1
int* const ptr2;            // CAN change *ptr2, can't change ptr2  
const int* const ptr3;      // Can't change either
int const x = 10;           // Same as const int (position matters!)
```

**Aria's Solution:**
```aria
fixed int32:x = 10i32;      // Can't change x. Period. Done.
```

One keyword. One meaning. Always.

### 2. Declaration/Definition Confusion (C)

**C's weird declaration syntax:**
```c
int (*func_ptr)(int, int);  // Pointer to function
int *arr[10];               // Array of pointers
int (*arr)[10];             // Pointer to array
```

You read it "inside-out" and "clockwise" and do mental gymnastics.

**Aria's Solution:**
```aria
func:callback = int32(int32, int32);  // Function type, clear
wild int32->:arr[10];                  // Array of pointers, clear
wild [10]int32->:arr_ptr;              // Pointer to array, clear
```

Read left to right. Type first, name last.

### 3. Overloaded Operators Doing Different Things

**C's `&` operator:**
```c
int x = 5 & 3;     // Bitwise AND
int* ptr = &x;     // Address-of
int y = x & y;     // Wait, which one?
```

**Aria's Solution:**
```aria
int32:x = 5i32 & 3i32;        // & ONLY for bitwise AND
wild int32->:ptr = @x;        // @ ONLY for address-of
```

Different operations = different symbols.

## The `fixed` Keyword Design

Following this principle, `fixed` means **exactly one thing**:

```aria
fixed T:name = value;
```

**What it means:**
- The variable `name` cannot be changed
- Its value is LOCKED
- Forever
- In all contexts
- No exceptions
- No "well actually..."

**What it does NOT mean:**
- ❌ "Sometimes you can change it"
- ❌ "You can change parts of it"  
- ❌ "Different behavior depending on type"
- ❌ "Only the pointer/reference is fixed"

**Just: FIXED. IMMUTABLE. DONE.**

### Implementation Validation (2026-02-07)

We just implemented `fixed` and it delivers on this promise:

```aria
// Test 1: Primitive reassignment - BLOCKED ✅
fixed int32:x = 10i32;
x = 20i32;
// ERROR: Cannot reassign fixed variable 'x' - fixed variables are immutable

// Test 2: Struct field modification - BLOCKED ✅  
fixed Point:p = Point{x: 10i32, y: 20i32};
p.x = 42i32;
// ERROR: Cannot modify field of fixed variable 'p' - fixed variables are immutable

// Test 3: Reading fixed values - WORKS ✅
fixed int32:a = 10i32;
fixed int32:b = 20i32;
int32:sum = a + b;  // Returns 30 - no problem!
```

**Implementation enforces:**
- Direct variable reassignment → Prevented
- Struct field modification → Prevented  
- Reading/using values → Allowed

**Error messages are clear:**
- No cryptic "cannot assign to const lvalue"
- Just: "Cannot reassign fixed variable 'x' - fixed variables are immutable"
- English. Actionable. Unambiguous.

One keyword. One meaning. Delivered.

## Design Philosophy: Informed Consent

**Aria is opinionated, but the final choice is yours.**

The language doesn't remove power - it makes you **acknowledge when you use it**.

### The Safe Path is the Easy Path

```aria
// Default: Safe, managed, type-checked
int32:x = 42i32;                    // GC by default
Point:p = Point{x: 10i32, y: 20i32}; // Bounds-checked
int32:result = divide(10i32, 0i32) ? 0i32;  // Errors handled
```

**You get:**
- Memory safety (GC)
- Type safety (strict checking)
- Error handling (result types)
- Thread safety (where applicable)

**For free. No keywords needed.**

### The Unsafe Path Requires Explicit Consent

```aria
// Want manual memory? Use 'wild'
wild int32->:ptr = malloc(sizeof(int32));
ptr->deref = 42i32;
free(ptr);  // YOU are responsible now

// Want to mutate? Already allowed (mutable by default)
int32:x = 10i32;
x = 20i32;  // No problem

// Want to prevent mutation? Use 'fixed'
fixed int32:x = 10i32;
x = 20i32;  // ERROR - you asked for immutability, you got it

// Want executable memory (JIT/codegen)? Use 'wildx'
wildx u8->:code = allocate_executable(...);
// Write code to memory, then mark executable
// W^X enforced: writable OR executable, never both
```

**The language makes you own your choices:**
- Used `wild`? Memory safety is YOUR problem now
- Used `wildx`? Executable memory management is YOUR problem now
- Used `fixed`? You can't mutate it. Period. You asked for this.

### The One Non-Negotiable Invariant

**Even `wildx` cannot bypass W^X (Write XOR Execute):**

```aria
wildx u8->:code = allocate_executable(1024);

// You can write to it OR execute it, never both
write_code(code, instructions);  // Memory is WRITABLE
mark_executable(code);            // Now it's EXECUTABLE (but NOT writable)
execute(code);                    // Run the code

// Cannot write AND execute simultaneously
// This is enforced by the runtime, not optional
```

**Why this matters:**
- W^X prevents code injection attacks
- Even JIT compilers need temporal separation
- This is a **security invariant**, not a performance trade-off
- You can use `wildx` for runtime codegen, but you cannot bypass fundamental security

**The tooling makes it easy:**
```aria
// Allocate executable memory region
wildx u8->:code = allocate_executable(size);

// Write your code (memory is writable, NOT executable)
write_bytecode(code, instructions);

// Mark as executable (memory becomes executable, NOT writable)
mark_executable(code);

// Now execute it
result = execute_code(code);
```

The APIs are there. The process is straightforward. You just have to do it.

**The principle:**
> "We trust you with executable memory. We do NOT trust you to violate basic security invariants."

**If the extra step of marking memory executable is too much work, you shouldn't be playing in those weeds.**

Some safety rails exist for your protection. Some exist for everyone's protection. W^X is the latter.

### You Can't Accidentally Opt Out

**What you CAN'T do:**
```aria
// ❌ Can't accidentally go unsafe
int32->:ptr = malloc(...);  // ERROR: Need 'wild' qualifier

// ❌ Can't accidentally skip bounds checks
int32:arr[10];
arr[50] = 42i32;  // Bounds checked unless you use 'wildx'

// ❌ Can't accidentally ignore errors
int32:x = might_fail();  // ERROR: Must unwrap result<int32>

// ❌ Can't "just cast" things
flt64:x = (flt64)42i32;  // ERROR: Explicit conversion required
```

**The principle:**
> "Make the right way the easy way, make the dangerous way require deliberate action."

If something goes wrong and you used `wild` or `wildx`, you **signed the TOS**. You acknowledged what you were doing. No "oops, I didn't know" defense.

### This Respects the Programmer

**What it's NOT:**
- ❌ "You're too stupid to use pointers"
- ❌ "The language knows better than you"
- ❌ "We removed features for your own good"

**What it IS:**
- ✅ "Here are the power tools. They're clearly labeled."
- ✅ "The safe tools are easier to reach"
- ✅ "If you grab the dangerous one, acknowledge it"
- ✅ "We trust you to make informed decisions"

**Python/Rust approach:** "We'll make everything safe by removing choices"  
**C approach:** "Everything is dangerous by default, good luck"  
**Aria approach:** "Safe by default, unsafe by explicit choice"

### Surgical Precision: Unsafe Only Where Needed

**The critical advantage: You can transition back and forth.**

```aria
// 99% of your code - safe, managed, type-checked
func:process_user_input = result<Data>(string:input) {
    Data:d = parse(input) ? fallback;
    Data:validated = validate(d) ? fallback;
    pass(validated);  // Memory safe, bounds checked
}

// 0.1% of your code - performance-critical, needs wild
func:optimize_9d_manifold = result<Matrix>(wild Matrix->:data, int32:threads) {
    // Nikola's high-performance math: 1000s of tiny threads in 9D space
    // Needs raw memory access for speed
    wild flt64->:raw = data->raw_buffer;
    
    // Ultra-fast calculations with manual memory management
    compute_manifold_interactions(raw, threads);
    
    // Transform back to safe
    Matrix:result = safe_copy(raw);
    pass(result);  // Returns to safety
}

// Back to safe code - unsafe section is contained
func:display_results = void(Matrix:m) {
    print(m.values);  // Memory safe again
    pass(NIL);
}
```

**What this enables:**
- Use `wild`/`wildx` **only** for the critical 0.1% that needs it
- Stay safe for the 99.9% that doesn't
- **Transition back** when the performance-critical section is done
- Unsafe code is **localized and explicit**

**Contrast:**

**Safe-only languages (Python/JavaScript):**
- Can't solve the performance-critical problems at all
- 9D manifold with 1000s of threads? Too slow or impossible
- You hit a wall and can't proceed

**Unsafe-everywhere languages (C):**
- Can solve the problem, but entire codebase is now vulnerable
- UI code has raw pointers, I/O has manual memory
- One mistake anywhere = security hole
- Can't "go back safe" because there is no safe

**Aria:**
- Solve the hard problems (raw performance when needed)
- Keep the easy problems safe (most of your code)
- **Unsafe doesn't contaminate** - it's scoped
- **Explicit boundaries** - you know exactly where unsafe starts/stops

**The principle:**
> "Use the minimum unsafe surface area required to solve the problem."

**This solves problems that safe-only languages literally cannot solve,** while maintaining safety everywhere possible.

You're not stuck with `wild` - you use it surgically, then return to safety.

### Real-World Analogy

**Industrial Equipment:**
- Safety guards are ON by default
- Removing them requires a physical key and a button press
- The emergency stop is big and red and obvious
- But you CAN remove the guards if you need to
- And when you do, everyone knows you did it deliberately

**Aria does the same for code.**

## Implementation Rules

1. **One symbol, one meaning** - Never reuse symbols for different operations
2. **Clear before clever** - Verbose and obvious beats terse and cryptic
3. **No context switches** - Symbol meaning doesn't change based on surrounding code
4. **Type belongs with type** - Not scattered across the declaration
5. **Read left to right** - Type, then name, then value (natural reading order)

## Why This Matters

**Cognitive Load:**
- Every context-dependent symbol is a mental branch
- Mental branches compound exponentially
- Time spent remembering rules = time NOT spent solving problems

**Blueprint Analogy:**
- In construction, symbols mean the same thing on every page
- You don't need to "check the context" to know what a beam symbol means
- Code should work the same way

**Error Prevention:**
- Ambiguous syntax → easy to misread
- Misreading → bugs
- Bugs → production failures
- Production failures → system crashes, money lost, lives at risk

## Counter-Examples from Real Languages

### JavaScript's `const` Lie
```javascript
const obj = {x: 10};
obj.x = 20;  // WORKS! "const" doesn't mean constant
```

`const` means "can't reassign the variable itself" but you CAN mutate its contents. That's not constant, that's "fixed reference to mutable data". Use different words for different concepts!

### Python's Mutable Default Arguments
```python
def append_to(element, target=[]):
    target.append(element)
    return target

append_to(1)  # [1]
append_to(2)  # [1, 2]  ← SURPRISE!
```

The default argument is evaluated ONCE at function definition, not each call. This is a context-dependent behavior that violates expectations.

### C++'s `auto` Decay
```cpp
int arr[5] = {1,2,3,4,5};
auto x = arr;  // x is int*, not int[5]
```

`auto` sometimes decays arrays to pointers. Context-dependent type deduction.

## Aria's Promise

**In Aria:**
- A keyword means what it says
- It means the same thing everywhere
- You read it once, understand it forever
- No surprises
- No gotchas
- No context switches

**The compiler serves the programmer, not the other way around.**

---

*"Complexity is the enemy of reliability."* — Design principle for mission-critical systems
