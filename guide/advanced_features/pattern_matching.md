# Pattern Matching

**Category**: Advanced Features → Patterns  
**Purpose**: Match values against patterns and destructure data

---

## Overview

Pattern matching provides powerful **structural comparison** and **data extraction**.

---

## Basic Match

```aria
enum Color {
    Red,
    Green,
    Blue,
}

func:describe = string(Color:color) {
    match color {
        Color.Red => return "red",
        Color.Green => return "green",
        Color.Blue => return "blue",
    }
}
```

---

## Match with Values

```aria
func:describe_number = string(int32:n) {
    match n {
        0 => return "zero",
        1 => return "one",
        2 => return "two",
        _ => return "many",  // Default case
    }
}
```

---

## Range Patterns

```aria
func:categorize_age = string(int32:age) {
    match age {
        0..=12 => return "child",
        13..=19 => return "teenager",
        20..=64 => return "adult",
        65.. => return "senior",
        _ => return "invalid",
    }
}
```

---

## Destructuring Tuples

```aria
func:process_point = NIL((i32, i32:point)) -> string {
    match point {
        (0, 0) => return "origin",
        (x, 0) => return `on x-axis at &{x}`,
        (0, y) => return `on y-axis at &{y}`,
        (x, y) if x == y => return "on diagonal",
        (x, y) => return `at (&{x}, &{y})`,
    }
}
```

---

## Destructuring Structs

```aria
struct Point {
    x: i32,
    y: i32,
}

func:describe_point = string(Point:p) {
    match p {
        Point { x: 0, y: 0 } => return "origin",
        Point { x, y: 0 } => return "on x-axis",
        Point { x: 0, y } => return "on y-axis",
        Point { x, y } => return `point at (&{x}, &{y})`,
    }
}
```

---

## Nested Patterns

```aria
enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
    Point(i32, i32),
}

func:area = flt64(Shape:shape) {
    match shape {
        Shape.Circle(r) => return 3.14159 * r * r,
        Shape.Rectangle(w, h) => return w * h,
        Shape.Point(_, _) => return 0.0,
    }
}
```

---

## Guards

```aria
func:classify_number = string(int32:n) {
    match n {
        x if x < 0 => return "negative",
        x if x == 0 => return "zero",
        x if x % 2 == 0 => return "positive even",
        x if x % 2 == 1 => return "positive odd",
        _ => return "unknown",
    }
}
```

---

## Option Matching

```aria
func:get_default = int32(?i32:opt, int32:default) {
    match opt {
        Some(value) => return value,
        None => return default,
    }
}

func:process_optional = NIL(?string:opt) {
    match opt {
        Some(s) if s.len() > 0 => print(s),
        Some(_) => print("empty string"),
        None => print("no value"),
    }
}
```

---

## Result Matching

```aria
func:handle_result = NIL(Result<int32>:Result) {
    match result {
        Ok(value) => print(`Success: &{value}`),
        Err(error) => stderr_write(`Error: &{error}`),
    }
}

func:process_with_recovery = Data(Result<Data>:Result) {
    match result {
        Ok(data) => return data,
        Err(NetworkError) => return fetch_from_cache(),
        Err(ParseError) => return Data.default(),
        Err(e) => panic(`Unexpected error: &{e}`),
    }
}
```

---

## Array/Slice Patterns

```aria
func:process_list = NIL([]i32:items) {
    match items {
        [] => print("empty"),
        [x] => print(`one item: &{x}`),
        [x, y] => print(`two items: &{x}, &{y}`),
        [first, ...rest] => {
            print(`first: &{first}`);
            print(`rest: &{rest}`);
        }
        _ => print("many items"),
    }
}
```

---

## Wildcard Patterns

```aria
func:process_tuple = NIL((i32, string, bool:t)) {
    match t {
        (0, _, _) => print("first is zero"),
        (_, "hello", _) => print("second is hello"),
        (_, _, true) => print("third is true"),
        _ => print("no match"),
    }
}
```

---

## Common Patterns

### Error Handling

```aria
func:safe_divide = Result<flt64>(flt64:a, flt64:b) {
    match b {
        0.0 => fail("Division by zero"),
        b if b.is_nan() => fail("Invalid divisor"),
        _ => pass(a / b),
    }
}
```

---

### State Machine

```aria
enum State {
    Idle,
    Running(i32),
    Paused(i32),
    Done,
}

func:transition = State(State:current, Event:event) {
    match (current, event) {
        (State.Idle, Event.Start) => return State.Running(0),
        (State.Running(n), Event.Pause) => return State.Paused(n),
        (State.Paused(n), Event.Resume) => return State.Running(n),
        (State.Running(n), Event.Complete) if n >= 100 => return State.Done,
        (State.Running(n), Event.Update) => return State.Running(n + 1),
        (state, _) => return state,  // No transition
    }
}
```

---

### Command Processing

```aria
enum Command {
    Quit,
    Help,
    Set(string, string),
    Get(string),
    Delete(string),
}

func:execute = NIL(Command:cmd, *State:state) {
    match cmd {
        Command.Quit => exit(0),
        Command.Help => print_help(),
        Command.Set(key, value) => state.insert(key, value),
        Command.Get(key) => {
            match state.get(key) {
                Some(value) => print(value),
                None => stderr_write("Key not found"),
            }
        }
        Command.Delete(key) => state.remove(key),
    }
}
```

---

### JSON Parsing

```aria
enum Json {
    Null,
    Bool(bool),
    Number(f64),
    String(string),
    Array([]Json),
    Object(HashMap<string, Json>),
}

func:stringify = string(Json:json) {
    match json {
        Json.Null => return "null",
        Json.Bool(true) => return "true",
        Json.Bool(false) => return "false",
        Json.Number(n) => return `&{n}`,
        Json.String(s) => return `\"&{s}\"`,
        Json.Array(items) => {
            // Handle array
        }
        Json.Object(map) => {
            // Handle object
        }
    }
}
```

---

## Multi-Pattern Match

```aria
func:is_vowel = bool(char:c) {
    match c {
        'a' | 'e' | 'i' | 'o' | 'u' => return true,
        'A' | 'E' | 'I' | 'O' | 'U' => return true,
        _ => return false,
    }
}

func:classify_char = string(char:c) {
    match c {
        'a'..'z' | 'A'..'Z' => return "letter",
        '0'..'9' => return "digit",
        ' ' | '\t' | '\n' => return "whitespace",
        _ => return "other",
    }
}
```

---

## Best Practices

### ✅ DO: Handle All Cases

```aria
func:safe_match = int32(?i32:opt) {
    match opt {
        Some(value) => return value,
        None => return 0,  // ✅ All cases handled
    }
}
```

### ✅ DO: Use Guards for Complex Conditions

```aria
func:validate_age = Result<int32>(int32:age) {
    match age {
        n if n < 0 => fail("Age cannot be negative"),
        n if n > 150 => fail("Age too high"),
        n => pass(n),
    }
}
```

### ✅ DO: Destructure to Extract Data

```aria
struct User {
    name: string,
    age: i32,
    email: string,
}

func:greet = NIL(User:user) {
    match user {
        User { name, age, .. } if age < 18 => {
            print(`Hi &{name}! You're under 18.`);
        }
        User { name, .. } => {
            print(`Hello &{name}!`);
        }
    }
}
```

### ⚠️ WARNING: Exhaustive Matching Required

```aria
enum Status {
    Active,
    Inactive,
    Pending,
}

func:process = NIL(Status:status) {
    match status {
        Status.Active => handle_active(),
        Status.Inactive => handle_inactive(),
        // ⚠️ Compiler error: missing Status.Pending
    }
}

// ✅ Fix: handle all cases or use _
func:process_fixed = NIL(Status:status) {
    match status {
        Status.Active => handle_active(),
        Status.Inactive => handle_inactive(),
        Status.Pending => handle_pending(),  // ✅ All cases
    }
}
```

### ❌ DON'T: Ignore Unused Bindings Warnings

```aria
// ❌ Bad - unused binding
match value {
    Some(x) => return default,  // x not used
    None => return default,
}

// ✅ Good - use _ for unused
match value {
    Some(_) => return default,
    None => return default,
}
```

---

## Related

- [destructuring](destructuring.md) - Destructuring syntax
- [error_propagation](error_propagation.md) - Error handling

---

**Remember**: Pattern matching makes code **safer** and more **expressive** by ensuring all cases are handled!
