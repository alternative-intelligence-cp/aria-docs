# Async/Await Pattern

**Category**: Advanced Features → Concurrency  
**Purpose**: Write asynchronous code that looks synchronous

---

## Overview

Async/await provides **non-blocking concurrency** with **readable, sequential-looking code**.

---

## Basic Pattern

```aria
async func:fetch_user = Result<User>(int32:id) {
    response: Response = await http.get(`/users/&{id}`)?;
    user: User = await response.json()?;
    pass(Ok(user));
}
```

---

## The Problem Async/Await Solves

### Without Async/Await (Callbacks)

```aria
func:fetch_user_callback = NIL(int32:id, fn(Result<User>:callback)) {
    http.get(`/users/&{id}`, fn(response) {
        response.json(fn(user) {
            callback(Ok(user));
        });
    });
}
// Callback hell!
```

---

### With Async/Await

```aria
async func:fetch_user = Result<User>(int32:id) {
    response: Response = await http.get(`/users/&{id}`)?;
    user: User = await response.json()?;
    pass(Ok(user));
}
// Clean and readable!
```

---

## Sequential Operations

```aria
async func:process_user = Result<NIL>(int32:id) {
    // Execute one after another
    user: User = await fetch_user(id)?;
    posts: []Post = await fetch_posts(user.id)?;
    comments: []Comment = await fetch_comments(posts[0].id)?;
    
    await save_to_cache(user, posts, comments)?;
    pass(Ok());
}
```

---

## Concurrent Operations

```aria
async func:fetch_dashboard = Result<Dashboard>(int32:user_id) {
    // Start all requests concurrently
    user_future = fetch_user(user_id);
    posts_future = fetch_posts(user_id);
    notifications_future = fetch_notifications(user_id);
    
    // Wait for all to complete
    user: User = await user_future?;
    posts: []Post = await posts_future?;
    notifications: []Notification = await notifications_future?;
    
    return Ok(Dashboard {
        user: user,
        posts: posts,
        notifications: notifications,
    });
}
```

---

## Error Handling

```aria
async func:fetch_with_fallback = User(int32:id) {
    match await fetch_user(id) {
        Ok(user) => return user,
        Err(e) => {
            print(`Primary fetch failed: &{e}`);
            
            match await fetch_user_from_cache(id) {
                Ok(cached) => return cached,
                Err(_) => return User.default(),
            }
        }
    }
}
```

---

## Common Patterns

### HTTP Request

```aria
async func:api_request = Result<Data>(string:endpoint) {
    response: Response = await http.get(endpoint)?;
    
    if !response.ok() {
        pass(Err(`HTTP &{response.status}`));
    }
    
    data: Data = await response.json()?;
    pass(Ok(data));
}
```

---

### Database Query

```aria
async func:fetch_user_with_posts = Result<UserWithPosts>(int32:id) {
    user: User = await db.query("SELECT * FROM users WHERE id = ?", id)?;
    posts: []Post = await db.query("SELECT * FROM posts WHERE user_id = ?", id)?;
    
    return Ok(UserWithPosts {
        user: user,
        posts: posts,
    });
}
```

---

### File I/O

```aria
async func:process_file = Result<NIL>(string:path) {
    content: string = await readFile(path)?;
    processed: string = await process_content(content)?;
    await writeFile(path ++ ".processed", processed)?;
    pass(Ok());
}
```

---

### WebSocket

```aria
async func:handle_websocket = NIL(WebSocket:socket) {
    messages = await socket.collect();
    till(messages.length - 1, 1) {
        match messages[$] {
            Text(text) => {
                response: string = await process_message(text)?;
                await socket.send(response)?;
            }
            Close => break,
        }
    }
}
```

---

## Joining Multiple Tasks

```aria
use std.async.join_all;

async func:fetch_all_users = Result<[]User>([]i32:ids) {
    tasks: []Future<User> = [];
    
    till(ids.length - 1, 1) {
        tasks.push(fetch_user(ids[$]));
    }
    
    users: []User = await join_all(tasks)?;
    pass(Ok(users));
}
```

---

## Racing Tasks

```aria
use std.async.race;

async func:fetch_fastest = Result<Data>() {
    primary = fetch_from_primary();
    backup = fetch_from_backup();
    
    // Return whichever completes first
    Result: Data = await race([primary, backup])?;
    pass(Ok(result));
}
```

---

## Timeout Pattern

```aria
use std.async.timeout;

async func:fetch_with_timeout = Result<Data>(string:url) {
    result = timeout(5000, fetch(url));  // 5 second timeout
    
    match await result {
        Ok(data) => return Ok(data),
        Err(Timeout) => return Err("Request timed out"),
    }
}
```

---

## Retry Pattern

```aria
async func:fetch_with_retry = Result<Data>(string:url, int32:max_retries) {
    retries: i32 = 0;
    
    loop {
        match await fetch(url) {
            Ok(data) => return Ok(data),
            Err(e) => {
                retries += 1;
                if retries >= max_retries {
                    pass(Err("Max retries exceeded"));
                }
                
                await sleep(1000 * retries);  // Exponential backoff
            }
        }
    }
}
```

---

## Best Practices

### ✅ DO: Use Async for I/O

```aria
async func:io_operations = Result<NIL>() {
    // Network I/O
    data: Data = await fetch_from_api()?;
    
    // File I/O
    content: string = await readFile("config.txt")?;
    
    // Database I/O
    users: []User = await db.query("SELECT * FROM users")?;
    
    pass(Ok());
}
```

### ✅ DO: Run Independent Tasks Concurrently

```aria
async func:concurrent_tasks = Result<NIL>() {
    // Start both
    task1 = expensive_operation1();
    task2 = expensive_operation2();
    
    // Wait for both
    result1 = await task1?;
    result2 = await task2?;
    
    pass(Ok());
}
```

### ✅ DO: Handle Errors Gracefully

```aria
async func:robust_fetch = Result<Data>() {
    match await fetch_data() {
        Ok(data) => return Ok(data),
        Err(e) => {
            log_error(e);
            pass(await fetch_from_cache());
        }
    }
}
```

### ❌ DON'T: Block the Executor

```aria
async func:blocking = NIL() {
    // ❌ Bad - blocks all async tasks
    till(999999, 1) {
        compute($);
    }
    
    // ✅ Good - yield periodically
    till(999999, 1) {
        if $ % 1000 == 0 {
            await yield_now();
        }
        compute($);
    }
}
```

---

## Related

- [async](async.md) - Async keyword
- [await](await.md) - Await keyword
- [concurrency](concurrency.md) - Concurrency overview

---

**Remember**: Async/await makes **asynchronous code look synchronous** while remaining **non-blocking**!
