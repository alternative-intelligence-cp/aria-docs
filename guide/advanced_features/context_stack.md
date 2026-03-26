# Context Stack

**Category**: Advanced Features → Syntax  
**Purpose**: Track execution context and error information

---

## Overview

The context stack tracks **function calls**, **error locations**, and **debugging information** at runtime.

---

## Stack Traces

```aria
func:level3 = NIL() {
    panic("Something went wrong!");
}

func:level2 = NIL() {
    level3();
}

func:level1 = NIL() {
    level2();
}

func:main = NIL() {
    level1();
}

/*
Output:
panic: Something went wrong!
    at level3 (example.aria:2)
    at level2 (example.aria:6)
    at level1 (example.aria:10)
    at main (example.aria:14)
*/
```

---

## Error Context

```aria
func:process_file = Result<Data>(string:path) {
    content: string = readFile(path)
        .context(`Failed to read file: &{path}`)?;
    
    data: Data = parse(content)
        .context(`Failed to parse content from &{path}`)?;
    
    pass(Ok(data));
}

/*
If parsing fails:
Error: Failed to parse content from config.txt
Caused by: Invalid JSON at line 5
*/
```

---

## Call Stack Information

```aria
func:debug_stack = NIL() {
    // Get current stack trace
    stack: StackTrace = StackTrace.capture();
    frames = stack.frames();
    
    till(frames.length - 1, 1) {
        print("Function: &{frames[$].function}");
        print("File: &{frames[$].file}:&{frames[$].line}");
    }
}
```

---

## Unwinding

```aria
func:level3 = Result<NIL>() {
    pass(Err("Error at level3"));
}

func:level2 = Result<NIL>() {
    level3()?;  // Unwinds on error
    pass(Ok());
}

func:level1 = Result<NIL>() {
    level2()?;  // Unwinds on error
    pass(Ok());
}

func:main = NIL() {
    match level1() {
        Ok(_) => print("Success"),
        Err(e) => {
            stderr_write(`Error: &{e}`);
            // Stack automatically unwound
        }
    }
}
```

---

## Panic Unwinding

```aria
struct Resource {
    name: string,
}

impl Resource {
    func:destroy = NIL() {
        print(`Cleaning up &{self.name}`);
    }
}

impl Drop for Resource {
    func:drop = NIL() {
        self.destroy();
    }
}

func:use_resources = NIL() {
    r1: Resource = Resource { name: "A" };
    r2: Resource = Resource { name: "B" };
    r3: Resource = Resource { name: "C" };
    
    panic("Something failed!");
    
    // Stack unwinds, drop() called for r3, r2, r1 in reverse order
}

/*
Output:
Cleaning up C
Cleaning up B
Cleaning up A
panic: Something failed!
*/
```

---

## Common Patterns

### Error Propagation with Context

```aria
func:load_config = Result<Config>() {
    path: string = get_config_path()
        .context("Failed to determine config path")?;
    
    content: string = readFile(path)
        .context(`Failed to read config from &{path}`)?;
    
    config: Config = parse_config(content)
        .context("Failed to parse config")?;
    
    validate_config(config)
        .context("Config validation failed")?;
    
    pass(Ok(config));
}

/*
Error chain:
Error: Config validation failed
Caused by: Failed to parse config
Caused by: Failed to read config from /etc/app/config.toml
Caused by: Permission denied
*/
```

---

### Backtrace Capture

```aria
struct Error {
    message: string,
    backtrace: StackTrace,
}

impl Error {
    func:new = Error(string:message) {
        return Error {
            message: message,
            backtrace: StackTrace.capture(),
        };
    }
    
    func:print = NIL() {
        stderr_write(`Error: &{self.message}\n`);
        stderr_write("Backtrace:\n");
        frames = self.backtrace.frames();
        till(frames.length - 1, 1) {
            stderr_write("  &{frames[$].file}:&{frames[$].line} - &{frames[$].function}\n");
        }
    }
}
```

---

### Scope Guards

```aria
struct Guard {
    name: string,
}

impl Guard {
    func:new = Guard(string:name) {
        print(`Entering &{name}`);
        pass(Guard { name: name });
    }
}

impl Drop for Guard {
    func:drop = NIL() {
        print(`Leaving &{self.name}`);
    }
}

func:example = NIL() {
    _g1: Guard = Guard.new("outer");
    
    {
        _g2: Guard = Guard.new("inner");
        do_work();
    }  // _g2 dropped
    
    more_work();
}  // _g1 dropped

/*
Output:
Entering outer
Entering inner
Leaving inner
Leaving outer
*/
```

---

### Defer Cleanup

```aria
func:process = Result<NIL>() {
    file: File = open("data.txt")?;
    defer file.close();
    
    lock: Mutex = acquire_lock()?;
    defer lock.unlock();
    
    // Work with file and lock
    data: string = file.read_all()?;
    process_data(data)?;
    
    pass(Ok());
    // Deferred cleanup runs in reverse order:
    // 1. lock.unlock()
    // 2. file.close()
}
```

---

## Debug Information

```aria
func:trace_execution = NIL() {
    print("__FUNCTION__: &{__FUNCTION__}");  // Current function
    print("__FILE__: &{__FILE__}");          // Current file
    print("__LINE__: &{__LINE__}");          // Current line
    print("__MODULE__: &{__MODULE__}");      // Current module
}
```

---

## Best Practices

### ✅ DO: Add Context to Errors

```aria
func:load_user = Result<User>(int32:id) {
    user: User = database.find(id)
        .context(`Failed to load user &{id}`)?;
    
    pass(Ok(user));
}
```

### ✅ DO: Use RAII for Cleanup

```aria
func:safe_resource_use = Result<NIL>() {
    resource: Resource = acquire()?;
    defer resource.release();
    
    // Use resource
    resource.do_work()?;
    
    pass(Ok());
    // resource.release() called automatically
}
```

### ✅ DO: Capture Backtraces for Debugging

```aria
func:create_error = Error(string:msg) {
    return Error {
        message: msg,
        backtrace: StackTrace.capture(),
        timestamp: now(),
    };
}
```

### ⚠️ WARNING: Stack Overflow

```aria
func:infinite_recursion = NIL(int32:n) {
    print(n);
    infinite_recursion(n + 1);  // ⚠️ Will overflow stack
}

// ✅ Better - use iteration or tail recursion
func:safe_iteration = NIL(int32:max) {
    till(max - 1, 1) {
        print($);
    }
}
```

### ❌ DON'T: Ignore Unwinding

```aria
// ❌ Bad - resource leak on panic
func:bad = NIL() {
    resource: *Resource = allocate();
    risky_operation();  // Might panic
    free(resource);     // Never reached if panic
}

// ✅ Good - RAII ensures cleanup
func:good = NIL() {
    resource: Resource = Resource.new();
    defer resource.cleanup();
    risky_operation();  // Cleanup happens even on panic
}
```

---

## Stack Depth Control

```aria
// Get current stack depth
func:check_depth = int32() {
    stack: StackTrace = StackTrace.capture();
    pass(stack.depth());
}

func:recursive = Result<NIL>(int32:n, int32:max_depth) {
    if check_depth() > max_depth {
        pass(Err("Stack too deep"));
    }
    
    if n > 0 {
        pass(recursive(n - 1, max_depth));
    }
    
    pass(Ok());
}
```

---

## Related

- [error_propagation](error_propagation.md) - Error handling
- [pattern_matching](pattern_matching.md) - Match expressions

---

**Remember**: The context stack provides **debugging information** and ensures **proper cleanup** during unwinding!
