# `await` Keyword

**Category**: Advanced Features → Concurrency  
**Syntax**: `await <async_expression>`  
**Purpose**: Wait for async operation to complete

---

## Overview

`await` **suspends** execution until an async operation completes.

---

## Basic Await

```aria
async func:fetch = Result<Data>() {
    response: Response = await http.get("https://api.example.com")?;
    data: Data = await response.json()?;
    pass(data);
}
```

---

## What Await Does

```aria
// Without await - returns Future
future: Future<Data> = async_function();

// With await - returns Data
data: Data = await async_function();
```

---

## Sequential Awaits

```aria
async func:sequential = Result<NIL>() {
    user: User = await fetch_user()?;       // Wait for user
    posts: []Post = await fetch_posts()?;   // Then wait for posts
    comments: []Comment = await fetch_comments()?;  // Then comments
    pass();
}
```

Total time: time(user) + time(posts) + time(comments)

---

## Concurrent Awaits

```aria
async func:concurrent = Result<NIL>() {
    // Start all tasks
    user_task = fetch_user();
    posts_task = fetch_posts();
    comments_task = fetch_comments();
    
    // Wait for all (runs concurrently)
    user: User = await user_task?;
    posts: []Post = await posts_task?;
    comments: []Comment = await comments_task?;
    
    pass();
}
```

Total time: max(time(user), time(posts), time(comments))

---

## Error Handling

```aria
async func:with_error_handling = Result<Data>() {
    match await fetch_data() {
        Ok(data) => pass(data),
        Err(e) => {
            print(`Error: &{e}`);
            fail(e);
        }
    }
}
```

---

## Await with Try Operator

```aria
async func:fetch_and_process = Result<Processed>() {
    data: Data = await fetch_data()?;     // Propagate error
    validated: Valid = await validate(data)?;
    processed: Processed = await process(validated)?;
    pass(processed);
}
```

---

## Timeout

```aria
use std.time.timeout;

async func:with_timeout = Result<Data>() {
    result = timeout(5000, async {  // 5 second timeout
        pass(await slow_operation());
    });
    
    match await result {
        Ok(data) => pass(data),
        Err(Timeout) => fail("Operation timed out"),
    }
}
```

---

## Common Patterns

### Retry Logic

```aria
async func:fetch_with_retry = Result<Data>(string:url, int32:retries) {
    till(retries - 1, 1) {
        match await fetch(url) {
            Ok(data) => pass(data),
            Err(e) => {
                if $ == retries - 1 {
                    fail(e);
                }
                await sleep(1000 * ($ + 1));  // Backoff
            }
        }
    }
}
```

---

### Race Condition

```aria
use std.async.race;

async func:fetch_fastest = Result<Data>() {
    task1 = fetch_from_server1();
    task2 = fetch_from_server2();
    task3 = fetch_from_server3();
    
    // Return whichever completes first
    Result: Data = await race([task1, task2, task3])?;
    pass(result);
}
```

---

### Join All

```aria
use std.async.join_all;

async func:fetch_multiple = Result<[]Data>([]string:urls) {
    tasks: []Future<Data> = [];
    till(urls.length - 1, 1) {
        tasks.push(fetch(urls[$]));
    }
    
    results: []Data = await join_all(tasks)?;
    pass(results);
}
```

---

## Best Practices

### ✅ DO: Await in Async Functions

```aria
async func:correct = Result<Data>() {
    data: Data = await fetch()?;  // ✅ In async function
    pass(data);
}
```

### ✅ DO: Handle Errors

```aria
async func:with_error_check = Result<Data>() {
    match await fetch() {
        Ok(data) => process(data),
        Err(e) => {
            log_error(e);
            fail(e);
        }
    }
}
```

### ❌ DON'T: Await in Sync Functions

```aria
func:sync_function = Data() {
    data: Data = await fetch();  // ❌ Error: can't await in sync
}
```

### ❌ DON'T: Forget to Await

```aria
async func:wrong = NIL() {
    fetch();  // ❌ Returns Future, doesn't execute!
    
    await fetch();  // ✅ Actually executes
}
```

---

## Async Iteration

```aria
async func:process_stream = NIL() {
    stream: AsyncStream<Data> = get_stream();
    
    items = await stream.collect();
    till(items.length - 1, 1) {
        await process(items[$]);
    }
}
```

---

## Related

- [async](async.md) - Async keyword
- [async_await](async_await.md) - Async/await pattern
- [concurrency](concurrency.md) - Concurrency overview

---

**Remember**: `await` **suspends** - other tasks can run while waiting!
