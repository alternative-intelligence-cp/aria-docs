# Multiple Generics

**Category**: Functions → Generics  
**Concept**: Using multiple type parameters  
**Philosophy**: Each parameter serves a distinct purpose

---

## What Are Multiple Generics?

**Multiple generics** means using **two or more** type parameters (`T`, `U`, `V`, etc.) in a single function, struct, or type.

---

## Basic Two-Parameter Function

```aria
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}

// Different types
Result: (i32, string) = pair(42, "answer");

// Same types (but parameters are independent)
coords: (i32, i32) = pair(10, 20);
```

---

## Multiple Parameters in Structs

```aria
struct Pair<T, U> {
    first: T,
    second: U
}

// Create instances
mixed: Pair<i32, string> = Pair{
    first: 42,
    second: "answer"
};

same_type: Pair<i32, i32> = Pair{
    first: 10,
    second: 20
};
```

---

## Three or More Parameters

```aria
struct Triple<T, U, V> {
    first: T,
    second: U,
    third: V
}

func:create = Triple<T,(T:a, U:b, V:c)U, V> {
    return Triple{
        first: a,
        second: b,
        third: c
    };
}

// Usage
data: Triple<i32, string, bool> = create(42, "hello", true);
```

---

## Independent Type Parameters

Each parameter can be **any type**:

```aria
func:convert = To(From:value) {
    // From and To are completely independent
}

// i32 to string
text: string = convert::<i32, string>(42);

// string to i32
num: i32 = convert::<string, i32>("42");
```

---

## Related Type Parameters

Parameters can be **related** through constraints or usage:

```aria
// U depends on T through function parameter
func:map = U)([]T:array, fn(T:f)-> []U {
    Result: []U = [];
    till(array.length - 1, 1) {
        item: T = array[$];
        result.push(f(item));
    }
    pass(result);
}

// T and U are related by the transformation function
numbers: []i32 = [1, 2, 3];
strings: []string = map(numbers, |x| format("{}", x));
//                       T=i32             U=string
```

---

## Different Constraints

Each parameter can have **different constraints**:

```aria
func:process = Result(T:input, U:config)
    where T: Serialize,      // T must be serializable
          U: Display {        // U must be displayable
    // Implementation
}
```

---

## Common Multi-Generic Patterns

### Key-Value Pairs

```aria
struct Map<K, V> {
    entries: []Entry<K, V>
}

impl<K, V> Map<K, V> where K: Hashable {
    func:insert = NIL(K:key, V:value) { }
    func:get = V?(K:key) { }
}

// Usage
users: Map<i32, User> = Map::new();
sessions: Map<string, Session> = Map::new();
```

### Input-Output Transform

```aria
func:transform = Out)(In:data, fn(In:converter)-> Out {
    pass(converter(data));
}

// Convert i32 to string
text: string = transform(42, |x| format("{}", x));
```

### Error Types

```aria
enum Result<T, E> {
    Ok(T),
    Err(E)
}

func:parse = Result<i32,(string:input)E> {
    // T = i32, E is the error type
}
```

### Builder Pattern

```aria
struct Builder<T, U> {
    items: []T,
    metadata: U
}

impl<T, U> Builder<T, U> {
    func:add = Builder<T,(T:item)U> {
        self.items.push(item);
        pass(self);
    }
    
    func:with_metadata = Builder<T,(U:meta)U> {
        self.metadata = meta;
        pass(self);
    }
}
```

---

## Type Inference with Multiple Parameters

### All Inferred

```aria
func:pair = (T,(T:first, U:second)U) {
    pass((first, second));
}

// Both T and U inferred
Result: (i32, string) = pair(42, "hello");
```

### Partial Inference

```aria
func:convert = To(From:value) {
    // Implementation
}

// From inferred, To specified
Result: string = convert::<_, string>(42);

// Both specified
Result: string = convert::<i32, string>(42);
```

---

## Method with Additional Parameters

```aria
struct Box<T> {
    value: T
}

impl<T> Box<T> {
    // Method adds new parameter U
    func:map = U)(fn(T:f)-> Box<U> {
        pass(Box{value: f(self.value)});
    }
    
    // Method adds two new parameters
    func:zip = V)(Box<U>:other, fn(T, U:combiner)-> Box<V> {
        pass(Box{value: combiner(self.value, other.value)});
    }
}

// Usage
int_box: Box<i32> = Box{value: 42};

// T=i32, U=string
str_box: Box<string> = int_box.map(|x| format("{}", x));

// T=i32, U=bool, V=string
other: Box<bool> = Box{value: true};
Result: Box<string> = int_box.zip(other, |x, y| {
    pass(format("{}: {}", x, y));
});
```

---

## Nested Generics

```aria
// Generic containing generics
struct Container<T, U> {
    items: Vec<T>,
    metadata: Map<string, U>
}

// Triple nesting
complex: Vec<Map<string, Option<i32>>>;
```

---

## Type Parameter Order

Convention: **Most important/used first**:

```aria
// Good: Data type first, error type second
enum Result<T, E> {
    Ok(T),
    Err(E)
}

// Good: Key first, value second
struct Map<K, V> { }

// Good: Input first, output second
func:transform = Out(In:data) { }
```

---

## Best Practices

### ✅ DO: Use Descriptive Names

```aria
// Good: Clear purpose
func:convert = Target(Source:value) { }
func:cache = NIL(Key:k, Value:v) { }
```

### ✅ DO: Limit Parameter Count

```aria
// Good: 2-3 parameters
func:process = V(T:a, U:b) { }

// Consider refactoring if you need more:
struct Config<T, U> { }
func:process = V(Config<T, U>:config) { }
```

### ✅ DO: Group Related Types

```aria
// Good: Related types together
struct Request<Payload, Response> {
    data: Payload,
    handler: fn(Payload) -> Response
}
```

### ❌ DON'T: Use Too Many Parameters

```aria
// Wrong: Too many
func:complex = NIL(...) { }

// Right: Simplify
struct Params<T, U> { }
func:complex = V(Params<T, U>:params) { }
```

### ❌ DON'T: Make All Parameters Generic

```aria
// Wrong: Unnecessarily generic
struct Config<Host, Port, Timeout> {
    host: Host,      // Always string
    port: Port,      // Always i32
    timeout: Timeout // Always i32
}

// Right: Only genericize what varies
struct Config {
    host: string,
    port: i32,
    timeout: i32
}
```

---

## Real-World Examples

### API Client

```aria
struct ApiClient<Request, Response> {
    endpoint: string,
    _phantom_req: PhantomData<Request>,
    _phantom_res: PhantomData<Response>
}

impl<Req, Res> ApiClient<Req, Res> 
    where Req: Serialize,
          Res: Deserialize {
    
    async func:call = Res?(Req:request) {
        json: string = request.serialize();
        response: string = pass await http_post(self.endpoint, json);
        pass(Res::deserialize(response));
    }
}

// Usage
user_client: ApiClient<UserRequest, UserResponse>;
product_client: ApiClient<ProductRequest, ProductResponse>;
```

### State Machine

```aria
struct StateMachine<State, Event, Output> {
    current: State,
    transitions: Map<(State, Event), State>,
    actions: Map<State, fn() -> Output>
}

impl<S, E, O> StateMachine<S, E, O> 
    where S: Hash + Eq,
          E: Hash + Eq {
    
    func:transition = O?(E:event) {
        key: (S, E) = (self.current, event);
        next_state: S? = self.transitions.get(key);
        
        when next_state != nil then
            self.current = next_state;
            action: fn() -> O? = self.actions.get(next_state);
            when action != nil then
                pass(action());
            end
        end
        
        pass(nil);
    }
}
```

### Generic Pipeline

```aria
fn pipeline<A, B, C, D>(
    input: A,
    step1: fn(A) -> B,
    step2: fn(B) -> C,
    step3: fn(C) -> D
) -> D {
    result1: B = step1(input);
    result2: C = step2(result1);
    result3: D = step3(result2);
    pass(result3);
}

// Usage
Result: string = pipeline(
    42,
    |x: i32| -> f64 { x as f64 },
    |x: f64| -> i32 { (x * 2.0) as i32 },
    |x: i32| -> string { format("{}", x) }
);
```

---

## Related Topics

- [Generic Functions](generic_functions.md) - Generic functions
- [Generic Parameters](generic_parameters.md) - Parameter details
- [Generic Syntax](generic_syntax.md) - Syntax reference
- [Type Inference](type_inference.md) - Multi-parameter inference

---

**Remember**: Multiple generic parameters let you **express relationships** between different types - but keep the count reasonable!
