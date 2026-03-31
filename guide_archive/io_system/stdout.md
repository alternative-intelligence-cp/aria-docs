# stdout (Standard Output)

**Category**: I/O System → Control Plane  
**File Descriptor**: 1  
**Direction**: Output only  
**Philosophy**: Human-readable output, separate from structured data

---

## What is stdout?

`stdout` (standard output) is the traditional **output stream** for human-readable messages. It's file descriptor 1, the Unix standard for program output.

**In Aria's six-stream model**: `stdout` is part of the **control plane** - it's for humans, not machines.

---

## Basic Usage

```aria
// Simple output
print("Hello, world!\n");

// Multiple values
print("Result: " + result + "\n");

// Expressions
print("Array has " + items.len() + " elements\n");

// Formatted output
print(format("Balance: &{:.2f}", balance));
```

---

## stdout vs stddato

**The Critical Distinction**: `stdout` is for **humans**, `stddato` is for **machines**.

### stdout (Human Output)

```aria
// User-facing messages
print("Processing complete\n");
print("Files processed: " + count + "\n");
```

**Purpose**: Messages that humans read in terminals

### stddato (Machine Output)

```aria
// Structured data
stddato.write_json({"count": count, "status": "ok"});
```

**Purpose**: Data that other programs consume

### The Power: Both at Once!

```aria
// Humans see progress
print("Processing batch...\n");

// Machines get structured results
stddato.write_json({"batch": batch_id, "status": "done"});
```

**No conflicts!** stdout and stddato are completely separate.

---

## Common Patterns

### Progress Messages

```aria
func:process_batch = NIL([]Item:items) {
    total: usize = items.len();
    
    till(total - 1, 1) {
        // Human progress on stdout
        print("Processing " + ($ + 1) + "/" + total + "...\r");
        stdout.flush();  // Update immediately
        
        process_item(items[$]);
    }
    
    print("\n");  // Newline after progress
}
```

### Results Display

```aria
func:display_results = NIL([]Result:results) {
    print("\nResults:\n");
    print("========\n");
    
    till(results.length - 1, 1) {
        print("  " + results[$].name + ": " + results[$].value + "\n");
    }
    
    print("\nTotal: " + results.len() + " items\n");
}
```

---

## Best Practices

### ✅ DO: Use for User Messages

```aria
print("Welcome to MyApp!\n");
print("Operation completed successfully.\n");
```

### ✅ DO: Provide Both Human and Machine Output

```aria
// Humans
print("Processed " + count + " records\n");

// Machines
stddato.write_json({"records_processed": count});
```

### ❌ DON'T: Use for Debug Output

```aria
// WRONG: Debug output pollutes stdout
print("DEBUG: Entering function\n");

// RIGHT: Use stddbg
stddbg_write("Entering function");
```

### ❌ DON'T: Use for Structured Data

```aria
// WRONG: JSON on stdout breaks human output
print(json_dumps(data));

// RIGHT: Use stddato
stddato.write_json(data);
```

---

## Related Topics

- [stddato](stddato.md) - Structured output (vs stdout)
- [stderr](stderr.md) - Error messages
- [stddbg](stddbg.md) - Debug output
- [Six-Stream Topology](six_stream_topology.md)

---

**Remember**: `stdout` is for humans. If you're emitting JSON or binary data, use `stddato`. If you're debugging, use `stddbg`. Keep them separate!
