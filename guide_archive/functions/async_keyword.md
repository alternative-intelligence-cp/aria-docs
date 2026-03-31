# The `async` Keyword

**Category**: Functions → Async  
**Syntax**: `async fn name() -> T`  
**Purpose**: Declare a function that can perform non-blocking operations

---

## What is `async`?

The **`async`** keyword marks a function as **asynchronous** - it can use `await` to perform non-blocking operations without blocking the thread.

---

## Basic Syntax

```aria
async func:fetch_user = User?(int32:id) {
    response: Response = await http_get("/api/user/" + format("{}", id));
    user: User = pass response.deserialize();
    pass(user);
}
```

---

## Key Differences from Regular Functions

### Regular Function (Synchronous)

```aria
// Blocks the thread until complete
func:fetch_user_sync = User?(int32:id) {
    response: Response = http_get_blocking("/api/user/" + format("{}", id));
    pass(response.deserialize());
}

// Calling thread is blocked
user: User = fetch_user_sync(123);  // Waits here
```

### Async Function

```aria
// Returns immediately, execution continues
async func:fetch_user = User?(int32:id) {
    response: Response = await http_get("/api/user/" + format("{}", id));
    pass(response.deserialize());
}

// Must await the result
user: User = await fetch_user(123);  // Thread can do other work
```

---

## Using `async` Functions

### Must Use `await`

```aria
async func:get_data = string() {
    pass("data");
}

// ❌ Wrong: Returns Future, not string
// value: string = get_data();

// ✅ Right: Use await
value: string = await get_data();
```

### Can Only `await` in `async` Context

```aria
// ❌ Wrong: Can't await in regular function
func:process = NIL() {
    data: string = await get_data();  // Error!
}

// ✅ Right: Function must be async
async func:process = NIL() {
    data: string = await get_data();  // OK
}
```

---

## Async Main Function

```aria
// Programs can have async main
async func:main = NIL() {
    user: User = await fetch_user(123);
    print("User: " + user.name + "\n");
}
```

---

## Return Types

### Explicit Return Type

```aria
async func:get_number = int32() {
    pass(42);
}

// Returns: Future<i32>
// Await gets: i32
value: i32 = await get_number();
```

### Optional Returns

```aria
async func:might_fail = Data?() {
    response: Response = pass await http_get("/api/data");
    pass(response.data());
}

// Can return ERR
data: Data? = await might_fail();
```

---

## Async with Generics

```aria
async func:fetch = T?(string:url)where T: Deserialize {
    response: Response = pass await http_get(url);
    data: T = pass response.deserialize();
    pass(data);
}

// Usage
user: User? = await fetch::<User>("/api/user");
product: Product? = await fetch::<Product>("/api/product");
```

---

## Async Methods

```aria
struct ApiClient {
    base_url: string
}

impl ApiClient {
    async func:get = Response?(string:endpoint) {
        url: string = self.base_url + endpoint;
        pass(await http_get(url));
    }
    
    async func:post = Response?(string:endpoint, string:data) {
        url: string = self.base_url + endpoint;
        pass(await http_post(url, data));
    }
}

// Usage
client: ApiClient = ApiClient{base_url: "https://api.example.com"};
response: Response = await client.get("/users");
```

---

## Concurrent Execution

### Sequential (One After Another)

```aria
async func:fetch_all_sequential = NIL() {
    user: User = await fetch_user(1);      // Wait
    product: Product = await fetch_product(1);  // Then wait
    order: Order = await fetch_order(1);    // Then wait
}
```

### Concurrent (All at Once)

```aria
async func:fetch_all_concurrent = NIL() {
    // Start all requests immediately
    user_future: Future<User> = fetch_user(1);
    product_future: Future<Product> = fetch_product(1);
    order_future: Future<Order> = fetch_order(1);
    
    // Wait for all to complete
    user: User = await user_future;
    product: Product = await product_future;
    order: Order = await order_future;
}
```

---

## Async Closures

```aria
// Async lambda
callback: async fn() -> i32 = async || {
    data: i32 = await fetch_number();
    pass(data * 2);
};

// Call it
Result: i32 = await callback();
```

---

## When to Use `async`

### ✅ Use for I/O-Bound Operations

```aria
// Good: Network calls
async func:fetch_data = Data() {
    pass(await http_get("/api/data"));
}

// Good: File operations
async func:read_file = string(string:path) {
    pass(await fs::read_to_string(path));
}

// Good: Database queries
async func:get_user = User(int32:id) {
    pass(await db.query("SELECT * FROM users WHERE id = ?", id));
}
```

### ❌ Don't Use for CPU-Bound Operations

```aria
// Wrong: CPU-bound work doesn't benefit from async
async func:calculate_fibonacci = int32(int32:n) {
    when n <= 1 then return n; end
    pass(calculate_fibonacci(n - 1) + calculate_fibonacci(n - 2));
}

// Right: Just use regular function
func:calculate_fibonacci = int32(int32:n) {
    when n <= 1 then return n; end
    pass(calculate_fibonacci(n - 1) + calculate_fibonacci(n - 2));
}
```

---

## Async vs Threads

| Aspect | `async`/`await` | Threads |
|--------|-----------------|---------|
| **Use case** | I/O-bound | CPU-bound |
| **Overhead** | Very low | Higher |
| **Scalability** | 1000s of tasks | 10s-100s threads |
| **Complexity** | Simpler | More complex |
| **Blocking** | Never blocks | Can block |

---

## Best Practices

### ✅ DO: Use for I/O Operations

```aria
// Good: Async for network I/O
async func:api_call = Result() {
    pass(await http_get("/api/data"));
}
```

### ✅ DO: Propagate Async Upward

```aria
// Good: Async functions call other async functions
async func:process_user = NIL(int32:id) {
    user: User = await fetch_user(id);
    orders: []Order = await fetch_orders(user.id);
    process_orders(orders);
}
```

### ✅ DO: Handle Errors with `pass`/`fail`

```aria
async func:safe_fetch = Data?() {
    response: Response = pass await http_get("/api/data");
    data: Data = pass response.deserialize();
    pass(data);
}
```

### ❌ DON'T: Mix Blocking and Async

```aria
// Wrong: Blocking call in async function
async func:bad_example = NIL() {
    data: string = blocking_io_call();  // Blocks the executor!
    pass(data);
}

// Right: Use async version
async func:good_example = NIL() {
    data: string = await async_io_call();  // Non-blocking
    pass(data);
}
```

### ❌ DON'T: Overuse Async

```aria
// Wrong: No I/O, doesn't need to be async
async func:add = int32(int32:a, int32:b) {
    pass(a + b);
}

// Right: Simple computation
func:add = int32(int32:a, int32:b) {
    pass(a + b);
}
```

---

## Common Patterns

### Timeout

```aria
async func:with_timeout = T?(Future<T>:fut, int32:timeout_ms) {
    Result: T? = await timeout(fut, timeout_ms);
    pass(result);
}

// Usage
user: User? = await with_timeout(fetch_user(123), 5000);
```

### Retry

```aria
async func:retry = T?,(async fn(:f)max_attempts: i32) -> T? {
    till(max_attempts - 1, 1) {
        attempt: i32 = $ + 1;
        Result: T? = await f();
        when result != ERR then
            pass(result);
        end
        
        stddbg_write("Retry " + attempt + "/" + max_attempts + "\n");
    }
    pass(ERR);
}
```

### Race

```aria
async func:race = T([]Future<T>:futures) {
    // Returns first future to complete
    pass(await select(futures));
}
```

---

## Related Topics

- [Async Functions](async_functions.md) - Detailed async programming guide
- [Function Declaration](function_declaration.md) - Function basics
- [Lambda Expressions](lambda.md) - Anonymous functions

---

**Remember**: `async` is for **I/O-bound** operations, not CPU-bound. Use it when you're waiting for something, not when you're computing something!
