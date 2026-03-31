# fail Keyword

**Category**: Functions → Result Handling  
**Purpose**: Extract and handle error states from TBB results  
**Philosophy**: Make error handling explicit and first-class

---

## What is `fail`?

`fail` is Aria's keyword for **extracting error information** when a TBB result is ERR. It's the opposite of `pass`:

- **pass**: Extracts success value, propagates ERR
- **fail**: Handles ERR case, allows success to continue

---

## Basic Syntax

```aria
when fail operation() then
    // This block executes if operation() returned ERR
    // Handle the error
end
```

---

## How It Works

### Simple Error Handling

```aria
func:divide_safe = tbb32(tbb32:a, tbb32:b) {
    Result: tbb32 = divide(a, b);
    
    when fail result then
        stderr_write("ERROR: Division failed (divide by zero?)\n");
        pass(0);  // Fallback value
    end
    
    pass(result);
}
```

**If `divide` returns ERR**:
- `fail result` is true
- Error handling block executes
- Function returns fallback value

**If `divide` succeeds**:
- `fail result` is false
- Error block is skipped
- Function returns actual result

---

## Inline Error Checking

```aria
func:process_file = bool(string:filename) {
    when fail open_file(filename) then
        stderr_write("ERROR: Cannot open file: " + filename + "\n");
        pass(false);
    end
    
    // File opened successfully, continue processing
    pass(true);
}
```

---

## Common Patterns

### With Fallback Values

```aria
func:parse_number_safe = tbb32(string:s) {
    num: tbb32 = parse(s);
    
    when fail num then
        stddbg_write("Parse failed, returning default");
        pass(0);  // Default value
    end
    
    pass(num);
}
```

### With Error Messages

```aria
func:load_config = Config?(string:path) {
    file: File? = open_file(path);
    
    when fail file then
        stderr_write("ERROR: Cannot load config from " + path + "\n");
        stderr_write("  Using default configuration\n");
        pass(default_config());
    end
    
    content: string = pass file.read();
    config: Config = pass parse_json(content);
    
    pass(config);
}
```

### Chaining Error Checks

```aria
func:process_pipeline = Data?(string:input) {
    step1: Data? = parse(input);
    when fail step1 then
        stderr_write("ERROR: Parse failed\n");
        pass(ERR);
    end
    
    step2: Data? = validate(step1);
    when fail step2 then
        stderr_write("ERROR: Validation failed\n");
        pass(ERR);
    end
    
    step3: Data? = transform(step2);
    when fail step3 then
        stderr_write("ERROR: Transform failed\n");
        pass(ERR);
    end
    
    pass(step3);
}
```

---

## `fail` with `else`

```aria
func:get_value = tbb32(string:key) {
    value: tbb32 = lookup(key);
    
    when fail value then
        stderr_write("Key not found: " + key + "\n");
        pass(0);
    else
        stddbg_write("Found value: " + value);
        pass(value);
    end
}
```

---

## Combining `fail` and `pass`

```aria
func:robust_process = Result(string:data) {
    // Try primary method
    Result: Result? = try_primary(data);
    
    when fail result then
        stddbg_write("Primary method failed, trying fallback");
        
        // Try fallback method
        result = try_fallback(data);
        
        when fail result then
            stderr_write("ERROR: Both methods failed\n");
            pass(ERR);
        end
    end
    
    // At this point, result is guaranteed to be valid
    pass(pass finalize(result));
}
```

---

## Error Recovery Patterns

### Retry Logic

```aria
func:fetch_with_retry = Data?(string:url, int32:max_attempts) {
    till(max_attempts - 1, 1) {
        attempt: i32 = $ + 1;
        stddbg_write("Attempt " + attempt + "/" + max_attempts);
        
        data: Data? = fetch(url);
        
        when fail data then
            when attempt < max_attempts then
                stddbg_write("Retrying...");
                continue;
            else
                stderr_write("ERROR: All retry attempts failed\n");
                pass(ERR);
            end
        end
        
        // Success!
        pass(data);
    }
    
    pass(ERR);  // Shouldn't reach here
}
```

### Graceful Degradation

```aria
func:get_user_preference = string(int32:user_id, string:key) {
    // Try user-specific preference
    pref: string? = db.get_user_pref(user_id, key);
    
    when fail pref then
        // Fallback to default preference
        pref = db.get_default_pref(key);
        
        when fail pref then
            // Ultimate fallback: hardcoded default
            stddbg_write("Using hardcoded default for " + key);
            pass("default_value");
        end
    end
    
    pass(pref);
}
```

---

## Error Logging

```aria
func:process_with_logging = Result?(Item:item) {
    Result: Result? = process(item);
    
    when fail result then
        // Log error details
        stderr_write("ERROR: Processing failed\n");
        stderr_write("  Item ID: " + item.id + "\n");
        stderr_write("  Item type: " + item.type + "\n");
        stderr_write("  Timestamp: " + Time::now() + "\n");
        
        // Also log to debug stream
        stddbg_write("Full item data: " + item.serialize());
        
        pass(ERR);
    end
    
    pass(result);
}
```

---

## Best Practices

### ✅ DO: Provide Context in Error Messages

```aria
when fail parse_json(data) then
    stderr_write("ERROR: Failed to parse JSON\n");
    stderr_write("  Data length: " + data.len() + " bytes\n");
    stderr_write("  First 100 chars: " + data.substr(0, 100) + "\n");
    pass(ERR);
end
```

### ✅ DO: Use Fallback Values When Appropriate

```aria
// Configuration with sensible defaults
timeout: tbb32 = load_timeout_config();

when fail timeout then
    stddbg_write("Using default timeout");
    timeout = 30;  // 30 seconds default
end
```

### ✅ DO: Log Before Propagating Errors

```aria
when fail critical_operation() then
    stderr_write("CRITICAL: Operation failed, aborting\n");
    stddbg_write("Stack trace: " + get_stack_trace());
    pass(ERR);  // Propagate error upward
end
```

### ❌ DON'T: Silently Ignore Errors

```aria
// Wrong: Error is ignored
Result: tbb32 = dangerous_operation();
when fail result then
    // Empty block - error is lost!
end

// Right: At least log it
when fail dangerous_operation() then
    stderr_write("ERROR: Operation failed\n");
    pass(ERR);
end
```

### ❌ DON'T: Use `fail` with Non-TBB Types

```aria
// Wrong: i32 doesn't have ERR
value: i32 = get_number();
when fail value then  // Compile error!
    // ...
end

// Right: Use TBB types
value: tbb32 = get_number();
when fail value then
    // ...
end
```

---

## `fail` vs `pass` Decision Guide

**Use `pass` when**:
- You want errors to propagate automatically
- No special error handling needed
- Building a pipeline of operations

**Use `fail` when**:
- You want to handle the error case explicitly
- You need to log error details
- You have a fallback/default value
- You want to retry or recover

### Example Comparison

```aria
// Using pass: Simple propagation
func:simple_process = Data?(string:file) {
    content: string = pass read_file(file);
    data: Data = pass parse(content);
    pass(data);
    // Errors propagate automatically
}

// Using fail: Explicit handling
func:robust_process = Data?(string:file) {
    content: string? = read_file(file);
    
    when fail content then
        stderr_write("ERROR: Cannot read " + file + "\n");
        content = try_backup_file(file + ".bak");
        
        when fail content then
            pass(ERR);  // Both attempts failed
        end
    end
    
    data: Data = pass parse(content);
    pass(data);
}
```

---

## Error State Introspection

```aria
func:diagnose_error = NIL(tbb32:value) {
    when fail value then
        print("Value is in error state (ERR)\n");
        stddbg_write("ERR value: " + value);  // Shows -128 for tbb8, etc.
    else
        print("Value is valid: " + value + "\n");
    end
}
```

---

## Real-World Example: Database Query

```aria
func:fetch_user_safely = User?(int32:user_id) {
    // Try to fetch from cache first
    user: User? = cache.get_user(user_id);
    
    when fail user then
        stddbg_write("Cache miss for user " + user_id);
        
        // Fetch from database
        user = db.query_user(user_id);
        
        when fail user then
            stderr_write("ERROR: User " + user_id + " not found in database\n");
            stddbg_write("Database connection status: " + db.status());
            pass(ERR);
        end
        
        // Cache the result for next time
        cache.set_user(user_id, user);
        stddbg_write("Cached user " + user_id);
    end
    
    pass(user);
}
```

---

## Error Context with Multiple Failures

```aria
func:complex_operation = Result?(Data:data) {
    // Try operation A
    result_a: Result? = try_method_a(data);
    
    when fail result_a then
        stddbg_write("Method A failed, trying B");
        
        // Try operation B
        result_b: Result? = try_method_b(data);
        
        when fail result_b then
            stddbg_write("Method B failed, trying C");
            
            // Try operation C
            result_c: Result? = try_method_c(data);
            
            when fail result_c then
                stderr_write("ERROR: All methods (A, B, C) failed\n");
                pass(ERR);
            end
            
            pass(result_c);
        end
        
        pass(result_b);
    end
    
    pass(result_a);
}
```

---

## Related Topics

- [pass Keyword](pass_keyword.md) - Opposite of fail (extracts success)
- [TBB Types](../types/tbb8.md) - Types that support ERR
- [when/then/end](../control_flow/when_then_end.md) - Conditional control flow
- [Sticky Errors](../types/sticky_errors.md) - Error propagation

---

**Remember**: `fail` makes error handling **explicit and first-class**. Use it when you need to do something specific with errors - log them, recover from them, or provide fallbacks.
