# `async` Keyword

**Category**: Advanced Features → Concurrency  
**Syntax**: `async fn <name>() -> <type>`  
**Purpose**: Define asynchronous functions

---

## Overview

`async` marks functions that can **suspend** and **resume** execution, enabling non-blocking I/O.

---

## Basic Async Function

```aria
async func:fetch_data = Result<string>(string:url) {
    response: Response = await http.get(url)?;
    text: string = await response.text()?;
    pass(Ok(text));
}
```

---

## Calling Async Functions

```aria
// Must use await
async func:main = NIL() {
    data: string = await fetch_data("https://example.com")?;
    print(data);
}
```

---

## Async vs Sync

```aria
// Synchronous - blocks thread
func:sync_fetch = Result<string>(string:url) {
    // Blocks until complete
    pass(http_blocking.get(url)?);
}

// Asynchronous - doesn't block
async func:async_fetch = Result<string>(string:url) {
    // Suspends, other tasks can run
    pass(await http.get(url)?);
}
```

---

## Return Types

```aria
async func:get_number = int32() {
    pass(42);
}

async func:may_fail = Result<Data>() {
    data: Data = await fetch()?;
    pass(Ok(data));
}

async func:optional_data = ?Data() {
    if available {
        pass(Some(await fetch()));
    }
    pass(None);
}
```

---

## Async Blocks

```aria
func:main = NIL() {
    future = async {
        data: Data = await fetch_data();
        await process(data);
    };
    
    runtime.block_on(future);
}
```

---

## Concurrent Execution

```aria
async func:fetch_all = Result<NIL>() {
    // Start both requests concurrently
    task1 = fetch_data("url1");
    task2 = fetch_data("url2");
    
    // Wait for both to complete
    data1: Data = await task1?;
    data2: Data = await task2?;
    
    print(`Got &{data1} and &{data2}`);
    pass(Ok());
}
```

---

## Common Patterns

### Sequential Async

```aria
async func:sequential = Result<NIL>() {
    user: User = await fetch_user()?;
    posts: []Post = await fetch_posts(user.id)?;
    comments: []Comment = await fetch_comments(posts[0].id)?;
    pass(Ok());
}
```

---

### Concurrent Async

```aria
async func:concurrent = Result<NIL>() {
    // Start all at once
    user_task = fetch_user();
    posts_task = fetch_posts();
    stats_task = fetch_stats();
    
    // Wait for all
    user: User = await user_task?;
    posts: []Post = await posts_task?;
    stats: Stats = await stats_task?;
    
    pass(Ok());
}
```

---

### Async Iterator

```aria
async fn* async_generator() -> i32 {
    till(9, 1) {
        await async_sleep(100);
        yield $;
    }
}

async func:use_generator = NIL() {
    values = await async_generator().collect();
    till(values.length - 1, 1) {
        print(values[$]);
    }
}
```

---

## Best Practices

### ✅ DO: Use for I/O Operations

```aria
async func:read_files = Result<NIL>() {
    // Good - async I/O
    content1: string = await readFile("file1.txt")?;
    content2: string = await readFile("file2.txt")?;
    pass(Ok());
}
```

### ✅ DO: Run Independent Tasks Concurrently

```aria
async func:fetch_dashboard = Dashboard() {
    user = fetch_user();
    posts = fetch_posts();
    notifications = fetch_notifications();
    
    return Dashboard {
        user: await user,
        posts: await posts,
        notifications: await notifications,
    };
}
```

### ❌ DON'T: Use for CPU-Bound Work

```aria
async func:compute = int32() {
    // ❌ Bad - blocks thread with CPU work
    sum: i32 = 0;
    till(999999, 1) {
        sum += $;
    }
    pass(sum);
}

// Better - use regular function or thread
func:compute = int32() {
    // CPU work in sync function
}
```

---

## Related

- [await](await.md) - Await keyword
- [async_await](async_await.md) - Async/await pattern
- [concurrency](concurrency.md) - Concurrency overview

---

**Remember**: `async` enables **non-blocking** I/O - use for network, disk, and user input!
