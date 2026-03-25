# Aria Type System Design
**Date**: 2025-02-07  
**Status**: Design Phase - Ready for Implementation

## Philosophy

**"A little functional, a little OOP, a lot of imperative"**

### What We Have
- ✅ **Functional**: Closures, lambdas, first-class functions
- ✅ **OOP-lite**: UFCS method syntax (zero-cost)
- ✅ **Imperative**: Core strength - explicit, predictable control flow

### What We're Adding
**Type system** - Organized, encapsulated composable units with zero overhead

**NOT OOP** - No inheritance, no vtables, no runtime dispatch, no magic

## Core Principle: Zero Context Switching

**Aria's Consistency Law**: `type:name = value;` for EVERYTHING

```aria
// Variables
int32:x = 42;

// Functions  
func:add = int32(int32:a, int32:b) { pass(a + b); };

// Lambdas (just functions without names)
int32(int32:x) { pass(x * 2); };  // Can execute immediately with ()

// Structs
struct:Point = { int32:x; int32:y; };

// Types (new!)
Type:Counter = { /* regular Aria syntax inside */ };
```

**No special cases. No syntax changes. Just organization.**

## Type Declaration Syntax

### Complete Example

```aria
Type:Counter = {
    // Constructor - just a regular Aria function
    func:create = Counter(int32:initial) {
        Counter:c = { 
            secret_ptr = allocate_storage(),
            count = initial 
        };
        pass(c);
    };
    
    // Destructor - just a regular Aria function  
    func:destroy = void(Counter:self) {
        free(self.secret_ptr);
    };
    
    // Methods - just regular Aria functions with self parameter
    func:increment = void(Counter:self) {
        self.count = self.count + 1;
    };
    
    func:get_value = int32(Counter:self) {
        pass(self.count);
    };
    
    // Private members - only accessible within Type methods
    struct:internal = {
        int32->:secret_ptr;
        int64:internal_counter;
    };
    
    // Public members - accessible everywhere
    struct:interface = {
        int32:count;
        int8:flags;
    };
    
    // Static/type-level members
    struct:type = {
        int32:MAX_VALUE = 1000;
        int32:MIN_VALUE = 0;
    };
};
```

### Usage

```aria
// Instantiate - desugars to Counter_create(0)
Counter:myCounter = instance<Counter>(0);

// Method calls - UFCS desugars to Counter_increment(myCounter)
myCounter.increment();
int32:val = myCounter.get_value();

// Access public fields
myCounter.count = 5;

// Access static members
int32:max = Counter.MAX_VALUE;

// Cleanup - desugars to Counter_destroy(myCounter)
myCounter.destroy();
```

## Semantic Transformation (Compile-Time Only!)

### Input: Type Declaration

```aria
Type:Counter = {
    func:create = Counter(int32:initial) { /* ... */ };
    func:destroy = void(Counter:self) { /* ... */ };
    func:increment = void(Counter:self) { /* ... */ };
    
    struct:internal = { int32->:secret_ptr; };
    struct:interface = { int32:count; };
    struct:type = { int32:MAX_VALUE = 1000; };
};
```

### Output: Desugared Code

```aria
// Step 1: Merge internal + interface into actual struct
struct:Counter = {
    // From internal (private, but no runtime enforcement yet)
    int32->:secret_ptr;
    
    // From interface (public)
    int32:count;
};

// Step 2: Prefix all functions with TypeName_
pub func:Counter_create = Counter(int32:initial) {
    // Original body unchanged
};

pub func:Counter_destroy = void(Counter:self) {
    // Original body unchanged
};

pub func:Counter_increment = void(Counter:self) {
    // Can access both internal and interface fields
    // (Access control is semantic analysis concern)
};

// Step 3: Static members become module-level constants
pub func:Counter_MAX_VALUE = int32() { pass(1000); };

// Step 4: instance<Counter>(0) desugars to Counter_create(0)
Counter:myCounter = Counter_create(0);

// Step 5: myCounter.increment() uses existing UFCS
// Already implemented in codegen_expr.cpp:2727-2800!
```

## Implementation Phases

### Phase 1: Parser (Syntax Recognition)
**File**: `src/frontend/parser/parser.cpp`

Add case for `Type:` declarations:
```cpp
if (current_token.type == TOKEN_TYPE && peek().type == TOKEN_COLON) {
    return parseTypeDecl();
}
```

**New AST Node**:
```cpp
class TypeDeclStmt : public ASTNode {
public:
    std::string typeName;
    
    // Required functions
    ASTNodePtr createFunc;   // Constructor
    ASTNodePtr destroyFunc;  // Destructor (optional)
    
    // Struct definitions
    ASTNodePtr internal;     // struct:internal
    ASTNodePtr interface;    // struct:interface  
    ASTNodePtr typeMembers;  // struct:type
    
    // Additional methods
    std::vector<ASTNodePtr> methods;
};
```

### Phase 2: Semantic Analysis (Desugaring)
**File**: `src/frontend/sema/type_checker.cpp`

```cpp
void TypeChecker::checkTypeDecl(TypeDeclStmt* stmt) {
    // 1. Validate required components
    if (!stmt->createFunc) {
        addError("Type '" + stmt->typeName + "' missing required 'create' function");
    }
    
    // 2. Merge internal + interface into combined struct
    auto combinedStruct = mergeStructs(stmt->internal, stmt->interface);
    
    // 3. Register combined struct as actual type
    typeSystem->registerStruct(stmt->typeName, combinedStruct);
    
    // 4. Validate all methods
    for (auto& method : stmt->methods) {
        validateMethodSignature(method, stmt->typeName);
    }
    
    // 5. Export public symbols (interface + methods)
    exportTypeSymbols(stmt);
}
```

### Phase 3: Code Generation (No Changes Needed!)
**Existing code handles it all**:
- Struct codegen already works ✅
- Function codegen already works ✅  
- UFCS already implemented ✅
- Just need to emit desugared AST nodes

### Phase 4: Access Control (Future Enhancement)
**Optional**: Enforce `internal` vs `interface` at semantic analysis

```cpp
void TypeChecker::validateMemberAccess(MemberAccessExpr* expr) {
    // Check if accessing internal member from outside Type methods
    if (isInternalMember(expr->member) && !isWithinTypeMethod(currentFunction)) {
        addError("Cannot access internal member '" + expr->member + 
                 "' outside of Type methods");
    }
}
```

## Naming Strategy: OOP Deflection

### Terminology Mapping

| Traditional OOP | Aria Type System | Rationale |
|----------------|------------------|-----------|
| `class` | `Type:` | Different word = different thing |
| `constructor` | `func:create` | It's just a function you write |
| `destructor` | `func:destroy` | Explicit, not magic |
| `new Class()` | `instance<Type>()` | Shows it's calling `create` |
| `private` | `internal` | Implementation internals |
| `public` | `interface` | What you expose |
| `static` | `type` | Type-level, not instance-level |
| `this` | `self` | More functional-style |
| `method` | UFCS function | It's a function with prefix! |

### Defense Mechanisms

**Critic**: "This is just OOP with different names!"

**Response**: "Not at all. Look at the generated code - it's structs and functions. No inheritance, no vtables, no dynamic dispatch. Just organized composition with zero overhead. If you want OOP, use C++ or Java. Aria gives you the ergonomics without the cost."

**Critic**: "Why not use `class` keyword?"

**Response**: "Because it's not a class. It's a Type declaration that organizes related functions and data. The word matters - it sets expectations correctly."

**Critic**: "You're missing inheritance!"

**Response**: "We left that out on purpose :-) Use composition. Put one Type as a field in another. Studies show composition is more maintainable anyway. Favor composition over inheritance, right?"

**Critic**: "This is incomplete OOP!"

**Response**: "Well of course it isn't. Who told you it was? We explicitly avoided OOP complexity while keeping the useful organizational patterns. Zero-cost abstractions over convenient abstractions."

## Example: Type-Based Atomic Counter

```aria
// stdlib/concurrent/atomic_counter.aria

Type:AtomicCounter = {
    // Constructor
    func:create = AtomicCounter(int32:initial) {
        wild int8->:mem = malloc(4);
        int32->:ptr = mem;
        <- ptr = initial;
        AtomicCounter:ac = { storage = ptr, last_value = initial };
        pass(ac);
    };
    
    // Destructor
    func:destroy = void(AtomicCounter:self) {
        free(self.storage);
    };
    
    // Methods
    func:load = int32(AtomicCounter:self, int32:order) {
        int32:val = __atomic_load_4(self.storage, order);
        pass(val);
    };
    
    func:store = void(AtomicCounter:self, int32:val, int32:order) {
        __atomic_store_4(self.storage, val, order);
    };
    
    func:increment = int32(AtomicCounter:self) {
        int32:old = __atomic_fetch_add_4(self.storage, 1, 5);
        pass(old);
    };
    
    // Private - implementation detail
    struct:internal = {
        int32->:storage;
    };
    
    // Public - visible state
    struct:interface = {
        int32:last_value;
    };
    
    // Static members
    struct:type = {
        int32:RELAXED = 0;
        int32:ACQUIRE = 2;
        int32:RELEASE = 3;
        int32:SEQ_CST = 5;
    };
};

// Usage
use "stdlib/concurrent/atomic_counter.aria";

func:main = int32() {
    AtomicCounter:counter = instance<AtomicCounter>(0);
    
    // Method syntax (UFCS)
    counter.store(42, AtomicCounter.SEQ_CST);
    int32:prev = counter.increment();
    int32:current = counter.load(AtomicCounter.RELAXED);
    
    // Access public field
    counter.last_value = current;
    
    // Cleanup
    counter.destroy();
    
    pass(current);
};
```

## Composition Over Inheritance

```aria
// No inheritance needed - just compose!

Type:Logger = {
    func:create = Logger() { /* ... */ };
    func:log = void(Logger:self, int8->:msg) { /* ... */ };
    struct:interface = { int8->:name; };
};

Type:Database = {
    func:create = Database(Logger:logger) {
        Database:db = { logger = logger };
        pass(db);
    };
    
    func:query = void(Database:self, int8->:sql) {
        self.logger.log("Executing query");  // Delegate!
        // ... execute query
    };
    
    struct:interface = {
        Logger:logger;  // Composition!
    };
};

// Usage
func:main = void() {
    Logger:log = instance<Logger>();
    Database:db = instance<Database>(log);
    
    db.query("SELECT * FROM users");
    
    db.destroy();
    log.destroy();
};
```

## Advantages Over Traditional OOP

### Zero-Cost Abstractions ✅
```c
// Your Aria code:
counter.increment();

// Compiles to (LLVM IR):
call @AtomicCounter_increment(%struct.AtomicCounter %counter)

// NOT (OOP with vtable):
%vtable = load ptr, ptr %counter.vtable
%fn_ptr = getelementptr %vtable, i64 3
%fn = load ptr, ptr %fn_ptr
call void %fn(ptr %counter)  // Indirect call!
```

### Predictable Memory Layout ✅
```aria
Type:Point = {
    struct:interface = { int32:x; int32:y; };
};

// Memory layout (exactly):
// [x: 4 bytes][y: 4 bytes]
// Total: 8 bytes (no vtable pointer!)
```

### FFI Compatible ✅
```c
// C code can use Aria Types directly
struct Point {
    int32_t x;
    int32_t y;
};

void use_point(struct Point* p) {
    // Works perfectly!
}
```

### Compile-Time Resolution ✅
- All method calls resolved at compile time
- No runtime type checking
- LLVM can inline everything
- Cache-friendly (no pointer chasing)

## Current Implementation Status

### ✅ Already Working
- Struct definitions
- Function definitions
- UFCS method syntax
- Module exports
- Public visibility

### 🚧 Need to Implement
- `Type:` parser
- Type AST node
- Desugaring logic
- `instance<T>()` syntax
- Static member access (`Type.MEMBER`)

### 📋 Future Enhancements
- Access control enforcement (`internal` checking)
- Better error messages for Type usage
- Type-aware editor completion
- Automatic `destroy()` calls (RAII)

## Next Steps

1. **Draft parser changes** - Add Type: recognition
2. **Create AST node** - TypeDeclStmt structure
3. **Implement desugaring** - Transform to structs + functions
4. **Test with atomic example** - Verify zero-cost
5. **Document patterns** - Stdlib development guide

---

**Motto**: "Composition over inheritance, explicitness over magic, zero-cost over convenience"

**Tagline**: "It's not OOP, it's organized imperative programming :-)"
