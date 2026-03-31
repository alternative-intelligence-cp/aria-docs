# Abstract Syntax Tree (AST)

**Category**: Advanced Features → Compiler  
**Purpose**: Hierarchical representation of program structure

---

## Overview

The **Abstract Syntax Tree** represents program structure as a **tree** of nodes, abstracting away syntactic details.

---

## AST Node Types

```aria
enum ASTNode {
    // Program
    Program([]Declaration),
    
    // Declarations
    FunctionDecl {
        name: string,
        parameters: []Parameter,
        return_type: ?Type,
        body: Block,
    },
    
    StructDecl {
        name: string,
        fields: []Field,
    },
    
    VarDecl {
        name: string,
        type: ?Type,
        initializer: ?Expression,
    },
    
    // Statements
    ExpressionStmt(Expression),
    ReturnStmt(?Expression),
    IfStmt {
        condition: Expression,
        then_block: Block,
        else_block: ?Block,
    },
    WhileStmt {
        condition: Expression,
        body: Block,
    },
    
    // Expressions
    BinaryExpr {
        left: Expression,
        operator: BinaryOp,
        right: Expression,
    },
    UnaryExpr {
        operator: UnaryOp,
        operand: Expression,
    },
    CallExpr {
        callee: Expression,
        arguments: []Expression,
    },
    Literal(Value),
    Variable(string),
}
```

---

## AST for Simple Function

```aria
// Source code:
func:add = int32(int32:x, int32:y) {
    pass(x + y);
}

// AST representation:
Program {
    declarations: [
        FunctionDecl {
            name: "add",
            parameters: [
                Parameter { name: "x", type: Type.i32 },
                Parameter { name: "y", type: Type.i32 },
            ],
            return_type: Some(Type.i32),
            body: Block {
                statements: [
                    ReturnStmt(
                        Some(
                            BinaryExpr {
                                left: Variable("x"),
                                operator: BinaryOp.Add,
                                right: Variable("y"),
                            }
                        )
                    )
                ]
            }
        }
    ]
}
```

---

## AST Structure

```aria
// Base node with common information
struct ASTNode {
    kind: NodeKind,
    span: Span,        // Source location
    type: ?Type,       // Type (filled during analysis)
}

struct Span {
    file: string,
    start: i32,
    end: i32,
    line: i32,
    column: i32,
}

// Specific node types
struct FunctionDecl {
    base: ASTNode,
    name: string,
    parameters: []Parameter,
    return_type: ?Type,
    body: *Block,
}

struct BinaryExpr {
    base: ASTNode,
    left: *Expression,
    operator: BinaryOp,
    right: *Expression,
}
```

---

## Building the AST

```aria
impl Parser {
    func:parse_function = Result<*FunctionDecl>() {
        // Parse syntax
        name: string = self.parse_identifier()?;
        params: []Parameter = self.parse_parameters()?;
        return_type: ?Type = self.parse_return_type()?;
        body: *Block = self.parse_block()?;
        
        // Create AST node
        node: *FunctionDecl = alloc(FunctionDecl {
            base: ASTNode {
                kind: NodeKind.FunctionDecl,
                span: self.get_span(start, end),
                type: None,
            },
            name: name,
            parameters: params,
            return_type: return_type,
            body: body,
        });
        
        pass(node);
    }
}
```

---

## Traversing the AST

```aria
// Visitor pattern
trait ASTVisitor {
    func:visit_program = Result<NIL>(*Program:node)
    func:visit_function_decl = Result<NIL>(*FunctionDecl:node)
    func:visit_binary_expr = Result<NIL>(*BinaryExpr:node)
    // ... other visit methods
}

impl ASTVisitor for TypeChecker {
    func:visit_binary_expr = Result<NIL>(*BinaryExpr:node) {
        // Visit children first
        self.visit_expression(node.left)?;
        self.visit_expression(node.right)?;
        
        // Type check
        left_type: Type = node.left.type?;
        right_type: Type = node.right.type?;
        
        if left_type != right_type {
            fail("Type mismatch in binary expression");
        }
        
        // Set node type
        node.base.type = Some(left_type);
        
        pass();
    }
}
```

---

## AST Transformations

```aria
// AST → AST transformation
func:optimize_ast = *Program(*Program:ast) {
    optimizer: ASTOptimizer = ASTOptimizer.new();
    pass(optimizer.transform(ast));
}

impl ASTOptimizer {
    func:transform_binary_expr = *Expression(*BinaryExpr:node) {
        // Constant folding: 2 + 3 → 5
        if node.left is Literal && node.right is Literal {
            left_val: i64 = node.left.value;
            right_val: i64 = node.right.value;
            
            Result: i64 = match node.operator {
                BinaryOp.Add => left_val + right_val,
                BinaryOp.Sub => left_val - right_val,
                BinaryOp.Mul => left_val * right_val,
                BinaryOp.Div => left_val / right_val,
            };
            
            pass(alloc(Literal { value: result }));
        }
        
        pass(node);
    }
}
```

---

## Pretty Printing AST

```aria
impl FunctionDecl {
    func:pretty_print = string(int32:indent) {
        output: string = "";
        output += " " * indent ++ `FunctionDecl '&{self.name}'\n`;
        
        output += " " * (indent + 2) ++ "Parameters:\n";
        till(self.parameters.length - 1, 1) {
            param = self.parameters[$];
            output += " " * (indent + 4) ++ "&{param.name}: &{param.type}\n";
        }
        
        output += " " * (indent + 2) ++ "Body:\n";
        output += self.body.pretty_print(indent + 4);
        
        pass(output);
    }
}

// Output:
// FunctionDecl 'add'
//   Parameters:
//     x: i32
//     y: i32
//   Body:
//     ReturnStmt
//       BinaryExpr (+)
//         Variable 'x'
//         Variable 'y'
```

---

## Common AST Operations

### Type Checking

```aria
func:type_check = Result<NIL>(*Program:ast) {
    checker: TypeChecker = TypeChecker.new();
    
    till(ast.declarations.length - 1, 1) {
        checker.visit_declaration(ast.declarations[$])?;
    }
    
    pass();
}
```

---

### Code Generation

```aria
func:generate_code = string(*Program:ast) {
    generator: CodeGenerator = CodeGenerator.new();
    
    till(ast.declarations.length - 1, 1) {
        generator.visit_declaration(ast.declarations[$]);
    }
    
    pass(generator.get_output());
}
```

---

### Optimization

```aria
func:optimize = *Program(*Program:ast) {
    // Dead code elimination
    ast = eliminate_dead_code(ast);
    
    // Constant folding
    ast = fold_constants(ast);
    
    // Inline small functions
    ast = inline_functions(ast);
    
    pass(ast);
}
```

---

## Best Practices

### ✅ DO: Include Source Location

```aria
struct ASTNode {
    span: Span,  // Always track source location
    // ... other fields
}

// Enables better error messages:
// "Error at main.aria:10:5: Type mismatch"
```

### ✅ DO: Use Visitor Pattern

```aria
trait ASTVisitor {
    func:visit_node = Result<NIL>(*ASTNode:node) {
        match node.kind {
            NodeKind.FunctionDecl => self.visit_function_decl(node),
            NodeKind.BinaryExpr => self.visit_binary_expr(node),
            // ... other cases
        }
    }
}
```

### ✅ DO: Validate AST After Construction

```aria
func:validate_ast = Result<NIL>(*Program:ast) {
    validator: ASTValidator = ASTValidator.new();
    validator.check(ast)?;
    pass();
}
```

### ⚠️ WARNING: Deep Nesting

```aria
// AST can get very deep
// Use iteration or trampolining to avoid stack overflow
func:visit_deep_ast = NIL(*ASTNode:node) {
    stack: []*ASTNode = [node];
    
    while !stack.is_empty() {
        current: *ASTNode = stack.pop();
        process(current);
        stack.extend(current.children());
    }
}
```

---

## Related

- [parser](parser.md) - Build AST from tokens
- [lexer](lexer.md) - Tokenization
- [tokens](tokens.md) - Token types

---

**Remember**: The AST is the **central data structure** for compilation - it represents program semantics in a form suitable for analysis and transformation!
