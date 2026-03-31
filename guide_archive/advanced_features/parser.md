# Parser

**Category**: Advanced Features → Compiler  
**Purpose**: Build abstract syntax tree from tokens

---

## Overview

The **parser** performs **syntax analysis**, converting tokens into an **Abstract Syntax Tree (AST)**.

---

## Parsing Process

```
Tokens:       [FN] [IDENTIFIER:add] [(] [IDENTIFIER:x] [:] [TYPE:i32] ... [)]
              ↓
Parser:       Build syntax tree
              ↓
AST:          FunctionDecl {
                name: "add",
                parameters: [
                  Param { name: "x", type: "i32" },
                  Param { name: "y", type: "i32" }
                ],
                return_type: "i32",
                body: Block { ... }
              }
```

---

## Parser Structure

```aria
struct Parser {
    tokens: []Token,
    current: i32,
}

impl Parser {
    pub func:new = Parser([]Token:tokens) {
        return Parser {
            tokens: tokens,
            current: 0,
        };
    }
    
    pub func:parse = Result<AST>() {
        pass(self.parse_program());
    }
    
    func:peek = Token() {
        pass(self.tokens[self.current]);
    }
    
    func:advance = Token() {
        token: Token = self.peek();
        if !self.is_at_end() {
            self.current += 1;
        }
        pass(token);
    }
    
    func:is_at_end = bool() {
        pass(self.peek().type == TokenType.Eof);
    }
    
    func:check = bool(TokenType:type) {
        pass(self.peek().type == type);
    }
    
    func:expect = Result<Token>(TokenType:type, string:message) {
        if self.check(type) {
            pass(self.advance());
        }
        fail(message);
    }
}
```

---

## Recursive Descent Parsing

```aria
impl Parser {
    // program → declaration* EOF
    func:parse_program = Result<Program>() {
        declarations: []Declaration = [];
        
        while !self.is_at_end() {
            decl: Declaration = self.parse_declaration()?;
            declarations.push(decl);
        }
        
        pass(Program { declarations });
    }
    
    // declaration → function_decl | var_decl | struct_decl
    func:parse_declaration = Result<Declaration>() {
        if self.check(TokenType.Fn) {
            pass(self.parse_function());
        } else if self.check(TokenType.Struct) {
            pass(self.parse_struct());
        } else if self.check(TokenType.Let) {
            pass(self.parse_variable());
        }
        
        fail("Expected declaration");
    }
}
```

---

## Function Parsing

```aria
impl Parser {
    // fn name(params) -> return_type { body }
    func:parse_function = Result<FunctionDecl>() {
        self.expect(TokenType.Fn, "Expected 'fn'")?;
        
        name: Token = self.expect(TokenType.Identifier, "Expected function name")?;
        
        self.expect(TokenType.LParen, "Expected '('")?;
        params: []Parameter = self.parse_parameters()?;
        self.expect(TokenType.RParen, "Expected ')'")?;
        
        return_type: ?Type = None;
        if self.check(TokenType.Arrow) {
            self.advance();
            return_type = Some(self.parse_type()?);
        }
        
        body: Block = self.parse_block()?;
        
        pass(FunctionDecl {
            name: name.lexeme,
            parameters: params,
            return_type: return_type,
            body: body,
        });
    }
    
    func:parse_parameters = Result<[]Parameter>() {
        params: []Parameter = [];
        
        if !self.check(TokenType.RParen) {
            loop {
                name: Token = self.expect(TokenType.Identifier, "Expected parameter name")?;
                self.expect(TokenType.Colon, "Expected ':'")?;
                type: Type = self.parse_type()?;
                
                params.push(Parameter {
                    name: name.lexeme,
                    type: type,
                });
                
                if !self.check(TokenType.Comma) {
                    break;
                }
                self.advance();
            }
        }
        
        pass(params);
    }
}
```

---

## Expression Parsing (Precedence Climbing)

```aria
impl Parser {
    // Operator precedence table
    func:get_precedence = int32(TokenType:op) {
        match op {
            TokenType.Or => return 1,
            TokenType.And => return 2,
            TokenType.EqEq | TokenType.Ne => return 3,
            TokenType.Lt | TokenType.Le | TokenType.Gt | TokenType.Ge => return 4,
            TokenType.Plus | TokenType.Minus => return 5,
            TokenType.Star | TokenType.Slash => return 6,
            _ => return 0,
        }
    }
    
    func:parse_expression = Result<Expression>() {
        pass(self.parse_binary_expression(0));
    }
    
    func:parse_binary_expression = Result<Expression>(int32:min_precedence) {
        left: Expression = self.parse_primary()?;
        
        while self.get_precedence(self.peek().type) >= min_precedence {
            op: Token = self.advance();
            precedence: i32 = self.get_precedence(op.type);
            right: Expression = self.parse_binary_expression(precedence + 1)?;
            
            left = BinaryExpression {
                left: left,
                operator: op,
                right: right,
            };
        }
        
        pass(left);
    }
    
    func:parse_primary = Result<Expression>() {
        if self.check(TokenType.Integer) {
            value: Token = self.advance();
            pass(IntegerLiteral { value: value.value });
        } else if self.check(TokenType.Identifier) {
            name: Token = self.advance();
            pass(Variable { name: name.lexeme });
        } else if self.check(TokenType.LParen) {
            self.advance();
            expr: Expression = self.parse_expression()?;
            self.expect(TokenType.RParen, "Expected ')'")?;
            pass(expr);
        }
        
        fail("Expected expression");
    }
}
```

---

## Statement Parsing

```aria
impl Parser {
    func:parse_statement = Result<Statement>() {
        if self.check(TokenType.Return) {
            pass(self.parse_return());
        } else if self.check(TokenType.If) {
            pass(self.parse_if());
        } else if self.check(TokenType.While) {
            pass(self.parse_while());
        } else if self.check(TokenType.LBrace) {
            pass(self.parse_block());
        } else {
            pass(self.parse_expression_statement());
        }
    }
    
    func:parse_return = Result<ReturnStatement>() {
        self.expect(TokenType.Return, "Expected 'return'")?;
        
        value: ?Expression = None;
        if !self.check(TokenType.Semicolon) {
            value = Some(self.parse_expression()?);
        }
        
        self.expect(TokenType.Semicolon, "Expected ';'")?;
        
        pass(ReturnStatement { value });
    }
    
    func:parse_if = Result<IfStatement>() {
        self.expect(TokenType.If, "Expected 'if'")?;
        
        condition: Expression = self.parse_expression()?;
        then_block: Block = self.parse_block()?;
        
        else_block: ?Block = None;
        if self.check(TokenType.Else) {
            self.advance();
            else_block = Some(self.parse_block()?);
        }
        
        pass(IfStatement {
            condition: condition,
            then_block: then_block,
            else_block: else_block,
        });
    }
}
```

---

## Error Recovery

```aria
impl Parser {
    func:synchronize = NIL() {
        // Skip tokens until we find a statement boundary
        self.advance();
        
        while !self.is_at_end() {
            if self.peek(-1).type == TokenType.Semicolon {
                pass(NIL);
            }
            
            match self.peek().type {
                TokenType.Fn | TokenType.Let | TokenType.If | 
                TokenType.While | TokenType.Return => return,
                _ => self.advance(),
            }
        }
    }
    
    func:parse_declaration_with_recovery = ?Declaration() {
        match self.parse_declaration() {
            Ok(decl) => return Some(decl),
            Err(e) => {
                self.report_error(e);
                self.synchronize();
                pass(None);
            }
        }
    }
}
```

---

## Best Practices

### ✅ DO: Provide Clear Error Messages

```aria
func:expect = Result<Token>(TokenType:type, string:message) {
    if !self.check(type) {
        token: Token = self.peek();
        fail(
            "Parse error at &{token.line}:&{token.column}\n" ++
            `  &{message}\n` ++
            "  Found: &{token.lexeme}"
        );
    }
    pass(self.advance());
}
```

### ✅ DO: Implement Error Recovery

```aria
func:parse_program = Result<Program>() {
    declarations: []Declaration = [];
    errors: []string = [];
    
    while !self.is_at_end() {
        match self.parse_declaration() {
            Ok(decl) => declarations.push(decl),
            Err(e) => {
                errors.push(e);
                self.synchronize();
            }
        }
    }
    
    if errors.len() > 0 {
        fail(errors.join("\n"));
    }
    
    pass(Program { declarations });
}
```

### ✅ DO: Support Operator Precedence

```aria
// Use precedence climbing or Pratt parsing
// 1 + 2 * 3 should parse as 1 + (2 * 3), not (1 + 2) * 3
```

---

## Related

- [lexer](lexer.md) - Lexical analysis
- [tokens](tokens.md) - Token types
- [ast](ast.md) - Abstract syntax tree

---

**Remember**: The parser converts **flat tokens** into a **hierarchical AST** representing program structure!
