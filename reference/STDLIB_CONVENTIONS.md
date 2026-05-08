# Nitpick Standard Library Conventions

> Created v0.21.5 (May 2026). Codifies naming, error-handling, and borrow
> conventions that all stdlib packages should follow. Existing packages
> deviating from these are tracked in `KNOWN_ISSUES.md` and migrated as the
> opportunity arises (deprecate-then-rename, never break consumers silently).

This document is **prescriptive for new code** and **descriptive for existing
code** — when fixing a package, prefer to align with these rules. Renaming a
public symbol is a breaking change and requires a deprecation cycle.

---

## 1. Naming

### Free functions

```
<package>_<verb>_<noun>
```

The package prefix uses a short identifier, lowercase, no `aria_` prefix
(because every package already lives under `aria-*`):

| Package        | Free-function prefix | Example                   |
|----------------|----------------------|---------------------------|
| `aria-libc`    | `libc_`              | `libc_open`, `libc_read`  |
| `aria-fs`      | `fs_`                | `fs_read_to_string`       |
| `aria-str`     | `str_`               | `str_to_upper`            |
| `aria-json`    | `json_`              | `json_parse`              |
| `aria-channel` | `channel_`           | `channel_send`            |
| `aria-time`    | `time_`              | `time_monotonic_ns`       |

C-shim externs that talk directly to libc keep the `aria_libc_*` prefix
to disambiguate from the Nitpick-side `libc_*` wrappers (e.g.
`aria_libc_time_monotonic_ns`).

### Methods

```
Type.verb_noun
```

Methods are dispatched via UFCS (`recv.method(args)` lowers to
`Module.method(recv, args)`).

| Receiver         | Method                          |
|------------------|---------------------------------|
| `String`         | `s.length()`, `s.split(sep)`    |
| `Vec<T>`         | `v.push(x)`, `v.pop()`          |
| `HashMap<K,V>`   | `m.get(k)`, `m.set(k, v)`       |
| `Result<T>`      | `r.is_error()`, `r.map(f)`      |

### Constants

`UPPER_SNAKE_CASE`, declared `const` for compile-time integers or `fixed`
for runtime-immutable bindings:

```
const int32:MAX_PATH_LEN = 4096i32;
fixed int64:DEFAULT_TIMEOUT_MS = 30000i64;
```

### Type names

`UpperCamelCase` for structs, enums, and traits:

```
struct:HttpRequest = { ... };
enum:LogLevel = { ... };
trait:Display = { ... };
```

---

## 2. Error Handling

### When to return `Result<T>`

A function MUST return `Result<T>` when **any** of the following hold:

- It performs I/O (file, network, syscall)
- It parses untrusted input
- It can fail for reasons outside the caller's static control
- It calls a function that returns `Result<T>` and cannot meaningfully
  recover

A function MUST NOT return `Result<T>` when **all** of the following hold:

- The input domain fully constrains the output (pure transformation)
- All edge cases (empty input, max-int, etc.) have a sensible total result
- It performs no I/O and no allocation that can fail at runtime

### Result vs. failsafe

- `Result<T>` is for **expected, recoverable** failures (file not found,
  parse error, network timeout).
- `failsafe` is for **unrecoverable, abort-the-program** errors (out of
  memory, type-tag mismatch in a hash table, broken invariant).
- Never use `failsafe` to signal a recoverable failure — the caller has
  no way to handle it.

### Sentinel values

Avoid magic sentinel returns (`-1`, `NULL`, empty string for "not found").
Use `Result<T>` or `optional<T>` instead. Existing packages that return
sentinels are documented in `KNOWN_ISSUES.md`; new code MUST NOT add them.

---

## 3. Borrow Signatures

### Read-only borrow: `$$i`

Use `$$i` for parameters the function only reads:

```
func:str_length = int64($$i String:s) {
    pass s.length();
};
```

### Mutable borrow: `$$m`

Use `$$m` for parameters the function may write to:

```
func:vec_push = NIL($$m Vec<int32>:v, int32:x) {
    v.push(x);
};
```

### Owned parameters

Pass owned values (no `$$i` / `$$m`) only when the function consumes the
value (transfers it into a collection, drops it, or returns ownership of
something derived from it). Owned parameters are rare in stdlib code —
prefer borrows.

### Returns

- Returning a borrow requires lifetime annotation (closure-lifetime rules
  apply, see v0.20.x).
- Returning an owned value is the default and requires no annotation.

---

## 4. Iteration

Iterators follow the `Iterator<T>` trait pattern:

```
trait:Iterator = {
    type Item;
    func:next = optional<Item>($$m Self:self);
};
```

Methods that return iterators end in `_iter` (`Vec.iter()`, `String.chars()`
is grandfathered). Iterators borrow from their source; the source must
out-live the iterator (lifetime checked by the borrow checker).

---

## 5. Module Layout

Every package lives in `REPOS/aria-packages/packages/<name>/` with:

```
<name>/
├── npkbld.md          # Build manifest (required)
├── README.md          # One-paragraph summary + quick example (required)
├── src/
│   └── <name>.aria    # Public surface (required)
├── tests/
│   └── *.aria         # Unit tests (≥1 file required)
└── docs/              # Optional — if present, linked from guide chapter
    └── *.md
```

Re-exports are explicit: a package that re-exports another's symbols MUST
declare them with `pub use` rather than relying on transitive imports.

---

## 6. Version Compatibility

- Packages follow SemVer.
- A breaking API change (rename, signature change, removal) requires a
  major version bump (`0.x.0` → `0.(x+1).0`) **and** a deprecation cycle
  of at least one minor release where both old and new symbols exist.
- Bug fixes that change observable behavior in a way callers may rely on
  (e.g. error code change) are minor bumps and noted in the package
  CHANGELOG.

---

## 7. Documentation

Every public symbol (function, type, constant, trait) MUST have a doc
comment with at minimum:

- One-sentence summary
- Param descriptions (type-only is insufficient — describe meaning)
- Return value description, including error variants for `Result<T>`
- One example showing typical use

Doc comments use the `//!` prefix:

```
//! Read an entire file into a string.
//!
//! @param path  Filesystem path to the file. Must be a regular file.
//! @returns     `Ok(contents)` on success, or `Err(io_error)` on failure.
//!
//! Example:
//!     let result = fs_read_to_string("/etc/hostname");
//!     match result {
//!         Ok(s)  => println(`hostname = &{s}`),
//!         Err(_) => println("read failed"),
//!     }
func:fs_read_to_string = Result<String>($$i String:path) { ... };
```

---

## See Also

- `REPOS/aria-docs/reference/abi.md` — extern ABI rules
- `REPOS/aria-docs/reference/RESERVED_WORDS.md` — reserved identifiers
- `REPOS/aria-docs/reference/TRAITS_AND_BORROW_SEMANTICS_RFC.md` — borrow
  rules in depth
- `REPOS/aria/KNOWN_ISSUES.md` — packages currently deviating from these
  conventions
