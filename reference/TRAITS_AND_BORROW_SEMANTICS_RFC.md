# RFC: Traits & Borrow Semantics for Aria

**Status:** Design Proposal (v0.2.4)  
**Implementation Target:** v0.2.5 or v0.2.6  
**Author:** AI-assisted design  

---

## Motivation

Two gaps in Aria's type system limit what real programs can express:

1. **No polymorphic dispatch.** You can't write a function that works on "anything with an `.encode()` method." Database backends, serialization formats, and generic algorithms all need this.

2. **No way to share handles.** Move-by-default consumes every variable on first use. Passing a DB handle to two functions requires the `last_db()` re-acquisition hack. Borrows exist in the checker but aren't user-facing.

Both are needed for Nikola: DB abstraction across SQLite/PostgreSQL/Redis, GPU handle passing, iterator patterns, and generic collections.

---

## Part 1: Traits

### Design Goals

- **Monomorphization by default** — zero runtime cost, the compiler stamps out a copy per concrete type
- **Vtable dispatch opt-in** — `dyn Trait` for cases where you truly need runtime polymorphism
- **No trait soup** — keep the common case simple, don't require 5 bounds on every function
- **Compose with UFCS** — `obj.method()` already desugars to `Type_method(obj)`, traits extend this naturally
- **Compose with Type declarations** — `Type:Person` already groups struct + methods, trait conformance plugs in

### Syntax

#### Defining a trait

```aria
trait:Encodable = {
    func:encode = string(Self:self);
    func:size_hint = int32(Self:self);
};
```

- `Self` refers to the implementing type (resolved at compile time for monomorphization)
- Trait body contains function signatures — no default implementations in v1 (KISS)
- Traits can require other traits: `trait:Ordered = Equatable & { func:compare = int32(Self:a, Self:b); };`

#### Implementing a trait

```aria
impl Encodable for Vec3 {
    func:encode = string(Vec3:self) {
        pass("(" + int32_toString(self.x) + "," + int32_toString(self.y) + ")");
    };
    func:size_hint = int32(Vec3:self) {
        pass(20i32);
    };
};
```

- `impl Trait for Type { ... }` — explicit, clear, hard to confuse
- Each function must match the trait signature exactly (with `Self` replaced by the concrete type)
- An impl block can live in any file (not just where the type or trait is defined)

#### Integrating with Type declarations

```aria
Type:Person = {
    struct:internal = { int32:id; };
    struct:interface = { string:name; int32:age; };

    // Type methods (UFCS — already works)
    func:create = Person(string:name, int32:age) { ... };
    func:greet = NIL(Person:self) { print(self.name); pass(NIL); };

    // Trait conformance declared inline
    conforms Encodable;
    conforms Equatable;
};

// Implementations can be separate
impl Encodable for Person { ... };
```

- `conforms Trait;` inside a Type declaration is a compile-time assertion — the compiler checks that an `impl` block exists
- This keeps the Type declaration as a readable summary of what a type can do

#### Using trait bounds

```aria
func<T: Encodable>:write_all = NIL(T[]:items) {
    int32:i = 0i32;
    while (i < items.length) {
        print(items[i].encode());
        i = i + 1i32;
    }
    pass(NIL);
};
```

- Generic param constraints: `<T: Encodable>`, `<T: Encodable & Ordered>` (parser already handles this)
- The compiler monomorphizes: `write_all::<Vec3>` generates a concrete function with `Vec3_encode` calls
- UFCS glue: `items[i].encode()` → resolved to `Vec3_encode(items[i])` after monomorphization

#### Dynamic dispatch (opt-in)

```aria
func:write_any = NIL(dyn Encodable:item) {
    print(item.encode());
    pass(NIL);
};
```

- `dyn Trait` is a fat pointer (data ptr + vtable ptr)
- Runtime cost — only use when you need heterogeneous collections or plugin-style extensibility
- The compiler builds a vtable struct: `{ ptr encode_fn; ptr size_hint_fn; }`

### Standard Traits (Proposed)

| Trait | Methods | Notes |
|-------|---------|-------|
| `Equatable` | `equals(Self, Self) -> bool` | `==` operator desugars to this |
| `Ordered` | `compare(Self, Self) -> int32` | `<`, `>`, `<=`, `>=` desugar |
| `Hashable` | `hash(Self) -> int64` | Required for HashMap keys |
| `Copyable` | `copy(Self) -> Self` | Opt-in value semantics (most types are move-only) |
| `Displayable` | `to_string(Self) -> string` | `print()` and string interpolation |
| `Addable` | `add(Self, Self) -> Self` | `+` operator, plus `Subtractable`, `Multipliable`, `Dividable` |
| `Defaultable` | `default() -> Self` | Zero-value construction |
| `Droppable` | `finalize(Self) -> NIL` | Custom cleanup (called by `drop()` or scope exit) |

### Implementation Strategy

1. **Parser** — `trait:Name = { ... };` and `impl Trait for Type { ... }` node types (NT_TRAIT_DECL already reserved)
2. **Type checker** — resolve trait bounds on generic params, verify impl blocks match trait signatures
3. **IR codegen** — monomorphization: stamp out concrete function for each `<T>` instantiation with known type
4. **Vtable codegen** (later) — for `dyn Trait`: generate vtable struct, emit indirect calls

---

## Part 2: Borrow Semantics

### Design Goals

- **Make the existing borrow checker user-facing** — the infrastructure is already there (`$` operator, AccessPath, loan tracking)
- **Rust's safety without Rust's learning curve** — Aria should guide, not punish
- **Explicit borrows** — no implicit reference creation, you always see `$`
- **Compatible with move-by-default** — borrows are a controlled exception to the consume-on-use rule

### Syntax

#### Immutable borrow

```aria
int32:x = 42i32;
int32$:ref = $x;          // borrow x (immutable)
print(ref);                // use the borrow — x is still alive
print(x);                  // x is still usable — it wasn't moved
```

- `$x` creates an immutable borrow of `x`
- `int32$` is the borrow type (already in the type system as `SafeRefType`)
- Multiple immutable borrows allowed simultaneously

#### Mutable borrow

```aria
int32:x = 42i32;
int32$:ref = $mut x;       // mutable borrow
ref = 99i32;                // mutate through the borrow
// x is now 99
// cannot use x while ref is alive (enforced by borrow checker)
```

- `$mut x` creates a mutable borrow
- Only one mutable borrow at a time (borrow checker already enforces this)
- Original variable is frozen while mutably borrowed

#### Passing borrows to functions

```aria
func:print_value = NIL(int32$:val) {
    print(val);
    pass(NIL);
};

func:increment = NIL(int32$ mut:val) {
    val = val + 1i32;
    pass(NIL);
};

func:main = int32() {
    int32:x = 10i32;
    print_value($x);       // pass immutable borrow — x survives
    increment($mut x);     // pass mutable borrow — x is modified
    print(x);              // prints 11
    pass(0i32);
};
```

- `int32$` in a parameter = accepts an immutable borrow
- `int32$ mut` in a parameter = accepts a mutable borrow
- The caller uses `$x` or `$mut x` at the call site — always explicit

#### Structs and handles

```aria
// The Nikola use case: pass a DB handle to multiple functions
int32:db = aria_sqlite_open(":memory:");
setup_tables($mut db);      // borrows db, doesn't consume it
insert_data($mut db);       // borrows db again — previous borrow ended
query_data($db);            // immutable borrow — read-only access
aria_sqlite_close(db);      // final use — db is moved/consumed here
```

- No more `last_db()` hack
- The borrow checker ensures `db` isn't used after `close()`
- Mutable borrows are sequential (one at a time), immutable borrows can overlap

#### Split borrows (already implemented)

```aria
struct:Pair = { int32:a; int32:b; };
Pair:p = Pair{a: 1i32, b: 2i32};
int32$:ref_a = $p.a;       // borrow field a
int32$:ref_b = $p.b;       // borrow field b — disjoint, allowed
// Both refs alive simultaneously — the borrow checker tracks access paths
```

### Lifetime Rules

Aria uses **scope-based lifetimes** (no explicit lifetime annotations like Rust's `'a`):

1. A borrow cannot outlive the scope of the borrowed value
2. The borrow checker uses Appendage Theory: `Depth(Borrower) ≤ Depth(Owner)`
3. Returned borrows are forbidden in v1 — return owned values instead
4. Borrows in structs are forbidden in v1 — use owned values or pointers

These restrictions eliminate the need for lifetime parameters while covering 90% of use cases. The remaining 10% (self-referential structs, borrowed iterators) can use `wild` pointers with explicit unsafe opt-in.

### Two-Phase Borrows (already implemented)

```aria
Vector<int32>:vec = Vector{};
vec.push(vec.len());    // Works! push($mut vec, vec.len($vec))
                        // Phase 1: reserve mutable borrow for push
                        // Phase 2: activate after len() completes
```

### Pin Semantics (`#` operator — existing)

```aria
wild int8[1024]:buffer = malloc(1024);
int8#:pinned = #buffer;    // pin: GC won't move this memory
ffi_read(pinned, 1024);    // safe to pass to C — address is stable
// pinned released when scope exits
```

- `#` pins a value in memory (prevents GC relocation)
- Critical for FFI: C functions expect stable pointers
- Borrow checker tracks pin lifetime

---

## Part 3: Interaction Between Traits and Borrows

### Trait methods and self borrowing

```aria
trait:Printable = {
    func:display = NIL(Self$:self);      // takes immutable borrow of self
};

trait:Resettable = {
    func:reset = NIL(Self$ mut:self);    // takes mutable borrow of self
};
```

- Trait methods declare whether they borrow self immutably (`Self$`) or mutably (`Self$ mut`)
- This lets the compiler enforce at the call site: `obj.display()` auto-borrows immutably, `obj.reset()` auto-borrows mutably
- If a method needs to consume self (like `drop`), it takes `Self:self` (owned)

### Generic bounds with borrow-friendliness

```aria
func<T: Printable>:print_twice = NIL(T$:item) {
    item.display();     // first call borrows the borrow (re-borrow)
    item.display();     // second call — fine, immutable borrow
    pass(NIL);
};
```

- Passing `T$` (borrowed) means the function doesn't consume the value
- The caller retains ownership: `print_twice($my_value);`

---

## Summary of Syntax Additions

| Feature | Syntax | Example |
|---------|--------|---------|
| Trait definition | `trait:Name = { ... };` | `trait:Encodable = { func:encode = string(Self:self); };` |
| Trait implementation | `impl Trait for Type { ... };` | `impl Encodable for Vec3 { ... };` |
| Trait conformance | `conforms Trait;` | Inside `Type:` declaration |
| Trait bound | `<T: Trait>` | `func<T: Ordered>:sort = ...` |
| Multiple bounds | `<T: A & B>` | `func<T: Hashable & Equatable>:lookup = ...` |
| Dynamic dispatch | `dyn Trait` | `func:f = NIL(dyn Encodable:x)` |
| Immutable borrow | `$x` / `Type$` | `int32$:ref = $x;` |
| Mutable borrow | `$mut x` / `Type$ mut` | `int32$ mut:ref = $mut x;` |
| Pin | `#x` / `Type#` | `int8#:pinned = #buffer;` |

---

## Open Questions

1. **Default trait methods?** Allowing `func:encode = string(Self:self) { ... };` with a body inside a trait definition. Useful but adds complexity. Propose: defer to v2.
2. **Associated types?** `trait:Iterator = { type Item; func:next = Item$(...); };` — needed for iterators but complex. Propose: defer to v2.
3. **Trait objects in collections?** `dyn Encodable[]:items` for heterogeneous arrays. Needs vtable + fat pointer arrays. Propose: defer to v2.
4. **Operator overloading via traits?** `+` → `Addable.add()`. Propose: yes, include in v1 (natural, expected).
5. **Auto-borrow at call sites?** Should `obj.method()` auto-create `$obj` if method takes `Self$`? Propose: yes, for ergonomics (Rust does this).

---

## Implementation Phases

### Phase 1 (v0.2.5): Core Traits + User-Facing Borrows
- `trait:` declaration parsing and AST
- `impl Trait for Type` parsing and AST
- Trait bound resolution in type checker
- Monomorphization in IR codegen
- User-facing `$x` / `$mut x` borrow syntax
- Standard traits: `Equatable`, `Displayable`, `Copyable`

### Phase 2 (v0.2.6): Operator Traits + Dynamic Dispatch
- `Addable`, `Ordered`, `Hashable` with operator desugaring
- `dyn Trait` fat pointer support
- `conforms` assertion in Type declarations
- Generic collections (`Vector<T>`, `HashMap<K: Hashable, V>`)

### Phase 3 (v0.2.7+): Advanced
- Default trait methods
- Associated types
- Trait objects in collections
- Derived traits (`#[derive(Equatable, Hashable)]`)
