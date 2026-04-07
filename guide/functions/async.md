# Async Functions

## Declaration

```aria
async func:fetch_data = string(string:url) {
    // ... async operations
    pass data;
};
```

The `async` keyword marks a function as asynchronous. Async functions return
`Result<T>` like all other functions.

## Await

```aria
string:data = raw fetch_data("https://example.com");
```

## Related

- [advanced_features/concurrency.md](../advanced_features/concurrency.md) — threads, channels
- [result_system.md](result_system.md) — error handling in async context
