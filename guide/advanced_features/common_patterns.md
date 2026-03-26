# Common Patterns

**Category**: Advanced Features → Best Practices  
**Purpose**: Frequently used code patterns in Aria

---

## Overview

Common patterns that appear frequently in idiomatic Aria code.

---

## Builder Pattern

```aria
struct Config {
    host: string,
    port: i32,
    timeout: i32,
    retries: i32,
}

struct ConfigBuilder {
    host: ?string,
    port: ?i32,
    timeout: ?i32,
    retries: ?i32,
}

impl ConfigBuilder {
    pub func:new = ConfigBuilder() {
        return ConfigBuilder {
            host: None,
            port: None,
            timeout: None,
            retries: None,
        };
    }
    
    pub func:host = ConfigBuilder(string:host) {
        self.host = Some(host);
        pass(self);
    }
    
    pub func:port = ConfigBuilder(int32:port) {
        self.port = Some(port);
        pass(self);
    }
    
    pub func:build = Config() {
        return Config {
            host: self.host.unwrap_or("localhost"),
            port: self.port.unwrap_or(8080),
            timeout: self.timeout.unwrap_or(30),
            retries: self.retries.unwrap_or(3),
        };
    }
}

// Usage
config: Config = ConfigBuilder.new()
    .host("example.com")
    .port(443)
    .build();
```

---

## Option and Result Chaining

```aria
func:get_user_email = ?string(int32:user_id) {
    return find_user(user_id)?
        .get_profile()?
        .email;
}

func:fetch_and_process = Result<Data>() {
    return fetch_data()?
        .validate()?
        .transform()?
        .save()?;
}
```

---

## Iterator Pattern

```aria
func:process_users = NIL([]User:users) {
    users
        .filter(|u| u.age >= 18)
        .map(|u| u.name)
        .forEach(|name| {
            print(name);
        });
}

func:sum_even_squares = int32([]i32:numbers) {
    return numbers
        .filter(|n| n % 2 == 0)
        .map(|n| n * n)
        .sum();
}
```

---

## Factory Pattern

```aria
enum DatabaseType {
    Postgres,
    MySQL,
    SQLite,
}

trait Database {
    func:connect = Result<NIL>()
    func:query = Result<[]Row>(string:sql)
}

struct DatabaseFactory;

impl DatabaseFactory {
    pub func:create = Box<dyn(DatabaseType:db_type)Database> {
        match db_type {
            DatabaseType.Postgres => return Box.new(PostgresDB.new()),
            DatabaseType.MySQL => return Box.new(MySQLDB.new()),
            DatabaseType.SQLite => return Box.new(SQLiteDB.new()),
        }
    }
}

// Usage
db: Box<dyn Database> = DatabaseFactory.create(DatabaseType.Postgres);
db.connect()?;
```

---

## Singleton Pattern

```aria
struct Logger {
    instance: ?*Logger = None,
}

impl Logger {
    pub func:get_instance = *Logger() {
        if Logger.instance is None {
            Logger.instance = Some(alloc(Logger {
                // Initialize
            }));
        }
        pass(Logger.instance?);
    }
    
    pub func:log = NIL(string:message) {
        print(`[LOG] &{message}`);
    }
}

// Usage
Logger.get_instance().log("Application started");
```

---

## Strategy Pattern

```aria
trait SortStrategy {
    func:sort = []i32;([]i32:data)
}

struct QuickSort;
impl SortStrategy for QuickSort {
    func:sort = []i32([]i32:data) {
        // Quick sort implementation
    }
}

struct MergeSort;
impl SortStrategy for MergeSort {
    func:sort = []i32([]i32:data) {
        // Merge sort implementation
    }
}

struct Sorter {
    strategy: Box<dyn SortStrategy>,
}

impl Sorter {
    pub func:new = Sorter(Box<dyn SortStrategy>:strategy) {
        pass(Sorter { strategy: strategy });
    }
    
    pub func:sort = []i32([]i32:data) {
        pass(self.strategy.sort(data));
    }
}

// Usage
sorter: Sorter = Sorter.new(Box.new(QuickSort));
sorted: []i32 = sorter.sort([3, 1, 4, 1, 5]);
```

---

## RAII Pattern

```aria
struct File {
    handle: *FileHandle,
}

impl File {
    pub func:open = Result<File>(string:path) {
        handle: *FileHandle = open_file(path)?;
        pass(Ok(File { handle: handle }));
    }
}

impl Drop for File {
    func:drop = NIL() {
        close_file(self.handle);
        print("File closed automatically");
    }
}

// Usage
func:process = Result<NIL>() {
    file: File = File.open("data.txt")?;
    // Use file
    pass(Ok());
}  // File automatically closed
```

---

## Newtype Pattern

```aria
struct UserId(i32);
struct ProductId(i32);

func:get_user = Result<User>(UserId:id) {
    // Implementation
}

// Type safety!
user_id: UserId = UserId(42);
product_id: ProductId = ProductId(42);

get_user(user_id);         // ✅ Works
// get_user(product_id);   // ❌ Compile error
```

---

## Type State Pattern

```aria
struct Locked;
struct Unlocked;

struct Door<State> {
    state: State,
}

impl Door<Locked> {
    pub func:unlock = Door<Unlocked>() {
        print("Door unlocked");
        pass(Door { state: Unlocked });
    }
}

impl Door<Unlocked> {
    pub func:open = NIL() {
        print("Door opened");
    }
    
    pub func:lock = Door<Locked>() {
        print("Door locked");
        pass(Door { state: Locked });
    }
}

// Usage
door: Door<Locked> = Door { state: Locked };
// door.open();              // ❌ Compile error - door is locked!
door = door.unlock();        // ✅ Unlock first
door.open();                 // ✅ Now can open
```

---

## Visitor Pattern

```aria
trait Expression {
    func:accept = i32;(*Visitor:visitor)
}

struct Number {
    value: i32,
}

impl Expression for Number {
    func:accept = int32(*Visitor:visitor) {
        pass(visitor.visit_number(self));
    }
}

struct Add {
    left: Box<dyn Expression>,
    right: Box<dyn Expression>,
}

impl Expression for Add {
    func:accept = int32(*Visitor:visitor) {
        pass(visitor.visit_add(self));
    }
}

trait Visitor {
    func:visit_number = i32;(*Number:num)
    func:visit_add = i32;(*Add:add)
}

struct Evaluator;

impl Visitor for Evaluator {
    func:visit_number = int32(*Number:num) {
        pass(num.value);
    }
    
    func:visit_add = int32(*Add:add) {
        left: i32 = add.left.accept(self);
        right: i32 = add.right.accept(self);
        pass(left + right);
    }
}
```

---

## Command Pattern

```aria
trait Command {
    func:execute = NIL();
    func:undo = NIL();
}

struct AddUserCommand {
    user: User,
}

impl Command for AddUserCommand {
    func:execute = NIL() {
        database.insert(self.user);
    }
    
    func:undo = NIL() {
        database.delete(self.user.id);
    }
}

struct CommandHistory {
    history: []Box<dyn Command>,
}

impl CommandHistory {
    pub func:execute = NIL(Box<dyn Command>:cmd) {
        cmd.execute();
        self.history.push(cmd);
    }
    
    pub func:undo = NIL() {
        if let Some(cmd) = self.history.pop() {
            cmd.undo();
        }
    }
}
```

---

## Lazy Initialization

```aria
struct LazyValue<T> {
    value: ?T,
    initializer: fn() -> T,
}

impl<T> LazyValue<T> {
    pub func:new = T)(fn(:initializer)-> LazyValue<T> {
        return LazyValue {
            value: None,
            initializer: initializer,
        };
    }
    
    pub func:get = *T() {
        if self.value is None {
            self.value = Some(self.initializer());
        }
        pass($self.value?);
    }
}

// Usage
expensive_data: LazyValue<Vec<i32>> = LazyValue.new(|| {
    print("Computing expensive data...");
    pass(compute_large_dataset());
});

// Not computed yet
// ...
// Computed on first access
data: *Vec<i32> = expensive_data.get();
```

---

## Related

- [best_practices](best_practices.md) - Best practices
- [idioms](idioms.md) - Aria idioms
- [code_examples](code_examples.md) - Code examples

---

**Remember**: Patterns are proven solutions to common problems - use them to write more maintainable code!
