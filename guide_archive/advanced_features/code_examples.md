# Code Examples

**Category**: Advanced Features → Best Practices  
**Purpose**: Complete code examples demonstrating Aria features

---

## Overview

Practical examples showing how to use Aria features in real-world scenarios.

---

## HTTP Server

```aria
use std.net.{TcpListener, TcpStream};
use std.io.{read, write};

struct Server {
    listener: TcpListener,
}

impl Server {
    pub func:new = Result<Server>(int32:port) {
        listener: TcpListener = TcpListener.bind(`0.0.0.0:&{port}`)?;
        pass(Server { listener });
    }
    
    pub func:run = Result<NIL>() {
        print("Server listening on port &{self.listener.port()}");
        
        while true {
            stream: TcpStream = self.listener.accept()?;
            Thread.spawn(|| {
                self.handle_client(stream);
            });
        }
    }
    
    func:handle_client = NIL(TcpStream:stream) {
        defer stream.close();
        
        request: string = stream.read_to_string()?;
        response: string = self.process_request(request);
        stream.write_all(response.as_bytes())?;
    }
    
    func:process_request = string(string:request) {
        if request.starts_with("GET /") {
            pass("HTTP/1.1 200 OK\r\n\r\nHello, World!");
        }
        pass("HTTP/1.1 404 Not Found\r\n\r\n404 Not Found");
    }
}

func:main = Result<NIL>() {
    server: Server = Server.new(8080)?;
    server.run()?;
    pass();
}
```

---

## JSON Parser

```aria
enum Json {
    Null,
    Bool(bool),
    Number(f64),
    String(string),
    Array([]Json),
    Object(HashMap<string, Json>),
}

struct JsonParser {
    input: string,
    pos: i32,
}

impl JsonParser {
    pub func:parse = Result<Json>(string:input) {
        parser: JsonParser = JsonParser { input, pos: 0 };
        pass(parser.parse_value());
    }
    
    func:parse_value = Result<Json>() {
        self.skip_whitespace();
        
        c: char = self.peek()?;
        
        match c {
            'n' => return self.parse_null(),
            't' | 'f' => return self.parse_bool(),
            '"' => return self.parse_string(),
            '[' => return self.parse_array(),
            '{' => return self.parse_object(),
            _ if c.is_digit() || c == '-' => return self.parse_number(),
            _ => fail("Unexpected character"),
        }
    }
    
    func:parse_string = Result<Json>() {
        self.expect('"')?;
        start: i32 = self.pos;
        
        while self.peek()? != '"' {
            if self.peek()? == '\\' {
                self.advance();  // Skip escape
            }
            self.advance();
        }
        
        value: string = self.input[start..self.pos];
        self.expect('"')?;
        
        pass(Json.String(value));
    }
    
    func:parse_array = Result<Json>() {
        self.expect('[')?;
        items: []Json = [];
        
        self.skip_whitespace();
        if self.peek()? == ']' {
            self.advance();
            pass(Json.Array(items));
        }
        
        while true {
            item: Json = self.parse_value()?;
            items.push(item);
            
            self.skip_whitespace();
            if self.peek()? == ']' {
                self.advance();
                break;
            }
            
            self.expect(',')?;
        }
        
        pass(Json.Array(items));
    }
}

// Usage
json: Json = JsonParser.parse('{"name": "Alice", "age": 30}')?;
```

---

## Database Connection Pool

```aria
use std.sync.{Mutex, Semaphore};

struct Connection {
    id: i32,
    // Connection details
}

impl Connection {
    func:execute = Result<[]Row>(string:query) {
        // Execute query
    }
}

struct ConnectionPool {
    connections: Mutex<[]Connection>,
    semaphore: Semaphore,
    max_size: i32,
}

impl ConnectionPool {
    pub func:new = ConnectionPool(int32:max_size) {
        connections: []Connection = [];
        
        till(max_size - 1, 1) {
            connections.push(Connection { id: $ });
        }
        
        return ConnectionPool {
            connections: Mutex.new(connections),
            semaphore: Semaphore.new(max_size),
            max_size: max_size,
        };
    }
    
    pub func:acquire = Result<Connection>() {
        // Wait for available connection
        self.semaphore.acquire();
        
        // Get connection from pool
        lock = self.connections.lock();
        conn: Connection = lock.pop()?;
        
        pass(conn);
    }
    
    pub func:release = NIL(Connection:conn) {
        // Return connection to pool
        lock = self.connections.lock();
        lock.push(conn);
        
        // Signal availability
        self.semaphore.release();
    }
}

// Usage
pool: ConnectionPool = ConnectionPool.new(10);

func:query_database = Result<[]Row>() {
    conn: Connection = pool.acquire()?;
    defer pool.release(conn);
    
    results: []Row = conn.execute("SELECT * FROM users")?;
    pass(results);
}
```

---

## File Watcher

```aria
use std.fs.{watch, Event, EventKind};
use std.path.Path;

struct FileWatcher {
    path: Path,
    handlers: HashMap<EventKind, fn(Event)>,
}

impl FileWatcher {
    pub func:new = FileWatcher(Path:path) {
        return FileWatcher {
            path: path,
            handlers: HashMap.new(),
        };
    }
    
    pub func:on = NIL(EventKind:kind, fn(Event:handler)) {
        self.handlers.insert(kind, handler);
    }
    
    pub func:start = Result<NIL>() {
        watcher = watch(self.path)?;
        
        while event = watcher.recv()? {
            if let Some(handler) = self.handlers.get(event.kind) {
                handler(event);
            }
        }
        
        pass();
    }
}

// Usage
func:main = Result<NIL>() {
    watcher: FileWatcher = FileWatcher.new(Path.new("./watched"));
    
    watcher.on(EventKind.Create, |event| {
        print("File created: &{event.path}");
    });
    
    watcher.on(EventKind.Modify, |event| {
        print("File modified: &{event.path}");
    });
    
    watcher.on(EventKind.Delete, |event| {
        print("File deleted: &{event.path}");
    });
    
    watcher.start()?;
    pass();
}
```

---

## Command Line Parser

```aria
struct Args {
    program: string,
    flags: HashMap<string, bool>,
    options: HashMap<string, string>,
    positional: []string,
}

impl Args {
    pub func:parse = Args([]string:args) {
        Result: Args = Args {
            program: args[0],
            flags: HashMap.new(),
            options: HashMap.new(),
            positional: [],
        };
        
        i: i32 = 1;
        while i < args.len() {
            arg: string = args[i];
            
            if arg.starts_with("--") {
                // Long option
                if arg.contains("=") {
                    parts: []string = arg[2..].split("=");
                    result.options.insert(parts[0], parts[1]);
                } else {
                    if i + 1 < args.len() && !args[i + 1].starts_with("-") {
                        result.options.insert(arg[2..], args[i + 1]);
                        i += 1;
                    } else {
                        result.flags.insert(arg[2..], true);
                    }
                }
            } else if arg.starts_with("-") {
                // Short flag
                result.flags.insert(arg[1..], true);
            } else {
                // Positional argument
                result.positional.push(arg);
            }
            
            i += 1;
        }
        
        pass(result);
    }
    
    pub func:get_flag = bool(string:name) {
        pass(self.flags.get(name).unwrap_or(false));
    }
    
    pub func:get_option = ?string(string:name) {
        pass(self.options.get(name));
    }
}

// Usage
func:main = NIL() {
    args: Args = Args.parse(std.env.args());
    
    if args.get_flag("help") {
        print_help();
        pass(NIL);
    }
    
    port: i32 = args.get_option("port")
        .unwrap_or("8080")
        .parse()?;
    
    till(args.positional.length - 1, 1) {
        process_file(args.positional[$]);
    }
}
```

---

## Async HTTP Client

```aria
use std.http.{Request, Response};

struct HttpClient {
    base_url: string,
    timeout: i32,
}

impl HttpClient {
    pub func:new = HttpClient(string:base_url) {
        return HttpClient {
            base_url: base_url,
            timeout: 30,
        };
    }
    
    pub async func:get = Result<Response>(string:path) {
        url: string = `&{self.base_url}&{path}`;
        
        request: Request = Request.builder()
            .url(url)
            .method("GET")
            .timeout(self.timeout)
            .build();
        
        response: Response = await request.send()?;
        
        if !response.ok() {
            fail("HTTP &{response.status}");
        }
        
        pass(response);
    }
    
    pub async func:post = Result<Response>(string:path, string:body) {
        url: string = `&{self.base_url}&{path}`;
        
        request: Request = Request.builder()
            .url(url)
            .method("POST")
            .header("Content-Type", "application/json")
            .body(body)
            .build();
        
        response: Response = await request.send()?;
        pass(response);
    }
}

// Usage
async func:main = Result<NIL>() {
    client: HttpClient = HttpClient.new("https://api.example.com");
    
    response: Response = await client.get("/users")?;
    users: []User = await response.json()?;
    
    till(users.length - 1, 1) {
        print(users[$].name);
    }
    
    pass();
}
```

---

## Related

- [best_practices](best_practices.md) - Best practices
- [common_patterns](common_patterns.md) - Common patterns
- [idioms](idioms.md) - Aria idioms

---

**Remember**: These examples demonstrate real-world usage of Aria features!
