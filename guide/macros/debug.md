# Debugging Macros with `--expand-macros`

The `--expand-macros` flag dumps the compiler's macro expansion trace to
stdout and exits. Use it to inspect what each macro call expands to before
type-checking and code generation.

## Usage

```
npkc myfile.aria --expand-macros
```

The flag parses and type-checks the file (triggering macro expansion), then
prints the expansion log and exits with code 0. No binary is produced.

## Output Format

For each macro invocation in the file, the trace shows:

```
// macro expansion at <file>:<line>:<column>
// call:     <macro-name>!(<argument-AST>)
// expanded: <expanded-AST>
```

Followed by a total count:

```
// total: N expansion(s)
```

If the file contains no macro invocations:

```
// no macro invocations found in <file>
```

## Example

Given `example.aria`:
```aria
macro:double = (x) {
    x + x;
};

func:main = int32() {
    int32:val = double!(5i32);
    assert!(true);
    exit(0);
};
func:failsafe = int32(tbb32:e) { exit(1); };
```

Running:
```
$ npkc example.aria --expand-macros
// macro expansion at example.aria:6:17
// call:     double!(Literal(5:i32))
// expanded: Block([ExprStmt(Binary(Literal(5:i32) PLUS Literal(5:i32)))])

// macro expansion at example.aria:7:5
// call:     assert!(Literal(true))
// expanded: AssertStatic(Literal(true))

// total: 2 expansion(s)
```

## Reading the Output

- **`call:`** — shows the macro name and the raw AST of each argument
- **`expanded:`** — shows the substituted body AST after hygiene (gensym)
- **Line/column** — the call site in the source file
- Built-in macros (`assert!`, `todo!`, `unreachable!`, `cfg!`) appear in the
  trace with their canonical expanded forms

## Combining with Other Flags

`--expand-macros` cannot be combined with `--tokens` or `--ast-dump`. Use
each flag independently.

## Exit Code

`--expand-macros` always exits 0 on a well-formed file. If the file has
a syntax or type error, it exits non-zero with the diagnostic on stderr.
