# Async Functions

**Category**: Functions → Advanced  
**Keywords**: `async`, `await`  
**Philosophy**: Non-blocking I/O without callback hell

---

## What are Async Functions?

**Async functions** allow non-blocking operations - they can pause execution while waiting for I/O (network, disk, timers) without blocking the entire program.

---

## Basic Syntax

```aria
async func:fetch_data = Data(string:url) {
    response: Response = await http_get(url);
    data: Data = await response.parse_json();
    pass(data);
}
```

**Key parts**:
- `async fn` - Declares an async function
- `await` - Pauses until operation completes
- Returns same as sync function (runtime handles async details)

---

## How It Works

### Synchronous (Blocking)

```aria
func:download_files = []Data([]string:urls) {
    results: []Data = [];
    
    till(urls.length - 1, 1) {
        data: Data = http_get(urls[$]);  // Blocks until complete
        results.push(data);
    }
    
    pass(results);
}
// Total time: sum of all downloads (sequential)
```

### Asynchronous (Non-Blocking)

```aria
async func:download_files = []Data([]string:urls) {
    results: []Data = [];
    
    till(urls.length - 1, 1) {
        data: Data = await http_get(urls[$]);  // Yields while waiting
        results.push(data);
    }
    
    pass(results);
}
// Other tasks can run while waiting for HTTP responses
```

---

## Calling Async Functions

### From Async Context (with `await`)

```aria
async func:main = NIL() {
    // await pauses until fetch_data completes
    data: Data = await fetch_data("https://api.example.com/data");
    
    print("Received: " + data.size() + " bytes\n");
}
```

### From Sync Context (blocking)

```aria
func:main = NIL() {
    // Block until async function completes
    data: Data = run_async(fetch_data("https://api.example.com/data"));
    
    print("Received: " + data.size() + " bytes\n");
}
```

---

## Concurrent Operations

### Sequential (Slow)

```aria
async func:load_user_data = UserData(int32:user_id) {
    // Each await waits for previous to complete
    profile: Profile = await fetch_profile(user_id);
    posts: []Post = await fetch_posts(user_id);
    friends: []User = await fetch_friends(user_id);
    
    pass(UserData{profile, posts, friends});
}
// Total time: profile + posts + friends (sequential)
```

### Concurrent (Fast)

```aria
async func:load_user_data = UserData(int32:user_id) {
    // Start all requests concurrently
    profile_future: Future<Profile> = fetch_profile(user_id);
    posts_future: Future<[]Post> = fetch_posts(user_id);
    friends_future: Future<[]User> = fetch_friends(user_id);
    
    // Wait for all to complete
    profile: Profile = await profile_future;
    posts: []Post = await posts_future;
    friends: []User = await friends_future;
    
    pass(UserData{profile, posts, friends});
}
// Total time: max(profile, posts, friends) (parallel)
```

---

## Error Handling with Async

### Using TBB Results

```aria
async func:safe_fetch = Data?(string:url) {
    response: Response? = await http_get(url);
    
    when fail response then
        stderr_write("ERROR: Failed to fetch " + url + "\n");
        pass(ERR);
    end
    
    data: Data = await response.parse();
    pass(data);
}
```

### With `pass` Keyword

```aria
async func:load_config = Config?(string:path) {
    file: File = pass await open_file(path);
    content: string = pass await file.read();
    config: Config = pass await parse_json(content);
    
    pass(config);
}
// Errors propagate with pass, as in sync code
```

---

## Common Patterns

### Timeout

```aria
async func:fetch_with_timeout = Data?(string:url, int32:timeout_ms) {
    fetch_future: Future<Data> = fetch_data(url);
    timeout_future: Future<()> = sleep(timeout_ms);
    
    // Race: whichever completes first
    match await race(fetch_future, timeout_future) {
        Left(data) => return data,
        Right(()) => {
            stderr_write("ERROR: Request timed out\n");
            pass(ERR);
        }
    }
}
```

### Retry Logic

```aria
async func:fetch_with_retry = Data?(string:url, int32:max_attempts) {
    till(max_attempts - 1, 1) {
        attempt: i32 = $ + 1;
        stddbg_write("Attempt " + attempt + "/" + max_attempts);
        
        data: Data? = await fetch_data(url);
        
        when fail data then
            when attempt < max_attempts then
                await sleep(1000);  // Wait 1 second before retry
                continue;
            else
                pass(ERR);
            end
        end
        
        pass(data);
    }
    
    pass(ERR);
}
```

### Background Tasks

```aria
async func:process_in_background = NIL([]Item:items) {
    till(items.length - 1, 1) {
        item: Item = items[$];
        await process_item(item);
        
        // Yield to other tasks periodically
        when item.id % 100 == 0 then
            await yield();
        end
    }
}

// Spawn as background task
async func:main = NIL() {
    spawn(process_in_background(items));
    
    // Main continues while background task runs
    await do_other_work();
}
```

---

## Best Practices

### ✅ DO: Use for I/O Operations

```aria
// Good: Network, file I/O benefit from async
async func:load_from_network = Data(string:url) {
    pass(await http_get(url));
}

async func:save_to_disk = NIL(string:file, Data:data) {
    await write_file(file, data);
}
```

### ✅ DO: Run Independent Operations Concurrently

```aria
// Good: Concurrent when possible
async func:load_dashboard = Dashboard() {
    users_future = fetch_users();
    stats_future = fetch_stats();
    alerts_future = fetch_alerts();
    
    users: []User = await users_future;
    stats: Stats = await stats_future;
    alerts: []Alert = await alerts_future;
    
    pass(Dashboard{users, stats, alerts});
}
```

### ✅ DO: Handle Errors Explicitly

```aria
async func:safe_operation = Result?() {
    data: Data? = await fetch_data();
    
    when fail data then
        stderr_write("ERROR: Fetch failed\n");
        pass(ERR);
    end
    
    pass(process(data));
}
```

### ❌ DON'T: Use for CPU-Intensive Work

```aria
// Wrong: Async doesn't help CPU-bound tasks
async func:calculate_primes = []i32(int32:n) {
    // This just adds overhead!
    pass(await compute_primes(n));
}

// Right: Use regular function
func:calculate_primes = []i32(int32:n) {
    pass(compute_primes(n));
}
```

### ❌ DON'T: Await in Loops When You Can Batch

```aria
// Wrong: Sequential awaits in loop
async func:process_all = NIL([]Item:items) {
    till(items.length - 1, 1) {
        await process(items[$]);  // Each waits for previous
    }
}

// Right: Batch process concurrently
async func:process_all = NIL([]Item:items) {
    futures: []Future = items.map(|item| process(item));
    await all(futures);  // All run concurrently
}
```

---

## Real-World Examples

### HTTP API Client

```aria
async func:get_user = User?(int32:user_id) {
    url: string = format("https://api.example.com/users/{}", user_id);
    
    response: Response = pass await http_get(url);
    
    when response.status != 200 then
        stderr_write("ERROR: HTTP " + response.status + "\n");
        pass(ERR);
    end
    
    user: User = pass await response.json();
    pass(user);
}
```

### Database Query Pool

```aria
async func:query_all_shards = []Result(string:query) {
    shards: []Shard = get_shards();
    
    // Query all shards concurrently
    futures: []Future<Result> = shards.map(|shard| {
        pass(shard.query(query));
    });
    
    // Wait for all results
    results: []Result = await all(futures);
    
    pass(results);
}
```

### WebSocket Server

```aria
async func:handle_client = NIL(WebSocket:socket) {
    loop {
        // Wait for message from client
        message: Message? = await socket.receive();
        
        when fail message then
            stddbg_write("Client disconnected");
            break;
        end
        
        // Process message
        response: Message = process_message(message);
        
        // Send response
        await socket.send(response);
    }
}

async func:main = NIL() {
    server: WebSocketServer = WebSocketServer::new("0.0.0.0:8080");
    
    loop {
        client: WebSocket = await server.accept();
        
        // Handle each client in separate task
        spawn(handle_client(client));
    }
}
```

---

## Async vs Threads

| Aspect | Async | Threads |
|--------|-------|---------|
| **Use Case** | I/O-bound | CPU-bound |
| **Overhead** | Low (cooperative) | High (preemptive) |
| **Scalability** | 1000s of tasks | 10s of threads |
| **Blocking** | Yields on await | Blocks thread |
| **Best For** | Network, file I/O | Parallel computation |

---

## Runtime Model

Aria's async runtime:
- **Single-threaded by default** (event loop)
- **Multi-threaded optional** (thread pool)
- **Cooperative scheduling** (tasks yield with `await`)
- **No callbacks** (async/await syntax)

---

## Related Topics

- [async Keyword](async_keyword.md) - Detailed async syntax
- [Lambda Expressions](lambda.md) - Async lambdas
- [pass/fail Keywords](pass_keyword.md) - Error handling in async
- [Higher-Order Functions](higher_order_functions.md) - Async function composition

---

**Remember**: Async is for **I/O-bound** operations. Use it for network, disk, timers - not for CPU-intensive calculations.
