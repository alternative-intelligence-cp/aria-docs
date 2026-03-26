# Error Propagation

**Category**: Advanced Features → Patterns  
**Purpose**: Propagate errors through call stack elegantly

---

## Overview

Error propagation handles errors **cleanly** without deeply nested error checking.

---

## The `?` Operator

```aria
func:read_file = Result<string>(string:path) {
    file: File = open(path)?;  // Returns Err if open fails
    content: string = file.read_all()?;  // Returns Err if read fails
    pass(Ok(content));
}

// Without ?
func:read_file_verbose = Result<string>(string:path) {
    match open(path) {
        Ok(file) => {
            match file.read_all() {
                Ok(content) => return Ok(content),
                Err(e) => return Err(e),
            }
        }
        Err(e) => return Err(e),
    }
}
```

---

## Propagating Multiple Errors

```aria
func:process_user_file = Result<Data>(int32:user_id) {
    user: User = fetch_user(user_id)?;          // Network error?
    file: File = open(user.file_path)?;         // IO error?
    content: string = file.read_all()?;         // Read error?
    data: Data = parse_content(content)?;       // Parse error?
    validated: Data = validate(data)?;          // Validation error?
    pass(Ok(validated));
}
```

---

## Error Context

```aria
func:load_config = Result<Config>(string:path) {
    content: string = readFile(path)
        .context("Failed to read config file")?;
    
    config: Config = parse_config(content)
        .context("Failed to parse config")?;
    
    validated: Config = validate_config(config)
        .context("Invalid configuration")?;
    
    pass(Ok(validated));
}
```

---

## Early Return Pattern

```aria
func:validate_user = Result<NIL>(User:user) {
    if user.name == "" {
        pass(Err("Name cannot be empty"));
    }
    
    if user.age < 0 {
        pass(Err("Age cannot be negative"));
    }
    
    if user.age > 150 {
        pass(Err("Age too high"));
    }
    
    if !user.email.contains("@") {
        pass(Err("Invalid email"));
    }
    
    pass(Ok());
}
```

---

## Combining Results

```aria
func:fetch_all_data = Result<AllData>() {
    user: User = fetch_user()?;
    posts: []Post = fetch_posts()?;
    comments: []Comment = fetch_comments()?;
    
    return Ok(AllData {
        user: user,
        posts: posts,
        comments: comments,
    });
}
```

---

## Optional to Result

```aria
func:get_user = Result<User>(int32:id) {
    user: ?User = database.find(id);
    
    // Convert Option to Result
    pass(user.ok_or("User not found"));
}

func:parse_number = Result<int32>(string:s) {
    num: ?i32 = s.parse();
    pass(num.ok_or("Invalid number"));
}
```

---

## Common Patterns

### File Processing

```aria
func:process_file = Result<NIL>(string:input_path, string:output_path) {
    // Read input
    content: string = readFile(input_path)?;
    
    // Process
    processed: string = transform(content)?;
    
    // Write output
    writeFile(output_path, processed)?;
    
    pass(Ok());
}
```

---

### HTTP Request Chain

```aria
async func:fetch_user_posts = Result<[]Post>(int32:user_id) {
    // Fetch user
    user_resp: Response = await http.get(`/users/&{user_id}`)?;
    user: User = await user_resp.json()?;
    
    // Fetch posts
    posts_resp: Response = await http.get(`/posts?user=&{user_id}`)?;
    posts: []Post = await posts_resp.json()?;
    
    pass(Ok(posts));
}
```

---

### Database Transaction

```aria
func:create_user = Result<User>(string:name, string:email) {
    tx: Transaction = db.begin()?;
    
    // Insert user
    user_id: i32 = tx.execute(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        name, email
    )?;
    
    // Create profile
    tx.execute(
        "INSERT INTO profiles (user_id) VALUES (?)",
        user_id
    )?;
    
    // Commit transaction
    tx.commit()?;
    
    // Fetch created user
    user: User = get_user(user_id)?;
    pass(Ok(user));
}
```

---

### Validation Chain

```aria
func:register_user = Result<User>(UserData:data) {
    // Validate each field
    validate_name(data.name)?;
    validate_email(data.email)?;
    validate_password(data.password)?;
    validate_age(data.age)?;
    
    // Check uniqueness
    check_email_unique(data.email)?;
    
    // Create user
    user: User = create_user(data)?;
    
    pass(Ok(user));
}
```

---

### Resource Cleanup

```aria
func:copy_file = Result<NIL>(string:src, string:dst) {
    src_file: File = open(src)?;
    defer src_file.close();
    
    dst_file: File = create(dst)?;
    defer dst_file.close();
    
    // Copy content
    content: []u8 = src_file.read_all()?;
    dst_file.write_all(content)?;
    
    pass(Ok());
}
```

---

## Error Transformation

```aria
func:fetch_data = Result<Data,()CustomError> {
    // Transform error type
    response: Response = http_get("/data")
        .map_err(|e| CustomError.NetworkError(e))?;
    
    data: Data = parse_response(response)
        .map_err(|e| CustomError.ParseError(e))?;
    
    pass(Ok(data));
}
```

---

## Fallible Iterator

```aria
func:process_files = Result<NIL>([]string:paths) {
    till(paths.length - 1, 1) {
        // Each iteration can fail
        content: string = readFile(paths[$])?;
        processed: string = process(content)?;
        writeFile(paths[$] ++ ".processed", processed)?;
    }
    pass(Ok());
}
```

---

## Collecting Results

```aria
func:fetch_all_users = Result<[]User>([]i32:ids) {
    users: []User = [];
    
    till(ids.length - 1, 1) {
        user: User = fetch_user(ids[$])?;  // Fail on first error
        users.push(user);
    }
    
    pass(Ok(users));
}

// Or collect all results
func:try_fetch_all = []Result<User>([]i32:ids) {
    results: []Result<User> = [];
    
    till(ids.length - 1, 1) {
        results.push(fetch_user(ids[$]));  // Continue on errors
    }
    
    pass(results);
}
```

---

## Best Practices

### ✅ DO: Use ? for Clean Error Handling

```aria
func:load_and_process = Result<Data>() {
    config: Config = load_config()?;
    input: Input = load_input()?;
    data: Data = process(config, input)?;
    pass(Ok(data));
}
```

### ✅ DO: Provide Context

```aria
func:load_user_data = Result<UserData>(int32:user_id) {
    user: User = fetch_user(user_id)
        .context(`Failed to fetch user &{user_id}`)?;
    
    profile: Profile = fetch_profile(user_id)
        .context(`Failed to fetch profile for user &{user_id}`)?;
    
    pass(Ok(UserData { user, profile }));
}
```

### ✅ DO: Use Early Returns

```aria
func:validate = Result<NIL>(Data:data) {
    if data.is_empty() {
        pass(Err("Data cannot be empty"));
    }
    
    if !data.is_valid() {
        pass(Err("Invalid data format"));
    }
    
    pass(Ok());
}
```

### ⚠️ WARNING: Don't Ignore Errors

```aria
// ❌ Bad - silently ignores error
func:bad_process = NIL() {
    _ = risky_operation();  // Error ignored!
}

// ✅ Good - handle or propagate
func:good_process = Result<NIL>() {
    risky_operation()?;  // Propagate error
    pass(Ok());
}

// ✅ Or explicitly handle
func:handle_process = NIL() {
    match risky_operation() {
        Ok(_) => print("Success"),
        Err(e) => stderr_write(`Error: &{e}`),
    }
}
```

### ❌ DON'T: Use ? in Non-Result Functions

```aria
// ❌ Error - can't use ? here
func:bad = NIL() {
    content: string = readFile("file.txt")?;  // Error!
}

// ✅ Good - return Result
func:good = Result<string>() {
    content: string = readFile("file.txt")?;
    pass(Ok(content));
}
```

---

## Related

- [pattern_matching](pattern_matching.md) - Pattern matching
- [destructuring](destructuring.md) - Destructuring

---

**Remember**: The `?` operator makes error handling **elegant** while maintaining **safety**!
