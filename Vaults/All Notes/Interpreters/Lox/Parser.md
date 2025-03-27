```rust
pub struct Parser {
    tokens: Vec<Token>,
    current: usize,
}
```

**Grammer:**
```
program        → declaration* EOF ;

declaration    → varDecl
               | statement ;

statement      → exprStmt
               | printStmt ;

exprStmt       → expression ";" ;

printStmt      → "print" expression ";" ;

expression     → equality ;

equality       → comparison ( ( "!=" | "==" ) comparison )* ;

comparison     → term ( ( ">" | ">=" | "<" | "<=" ) term )* ;

term           → factor ( ( "-" | "+" ) factor )* ;

factor         → unary ( ( "/" | "*" ) unary )* ;

unary          → ( "!" | "-" ) unary
               | primary ;

primary        → NUMBER | STRING | "true" | "false" | "nil"
               | "(" expression ")" ;
```
### Prase
We try and read all statements and add them to a vector which we later return.

The parsing logic works as the grammar dictates.
The program is made up of declarations or statements. which in turn are expressions or print statements and so on.