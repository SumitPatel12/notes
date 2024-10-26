_____
**Created**: 14-10-2024 08:56 pm
**Status**: Completed
**Tags**: #Rust #RustExpressions [[Rust]] [[Rust Expressions]]
**References**: 
______

In addition to expressions and semicolons, [[Blocks and Semicolons|blocks]] can contain any number of declarations. You might be most familiar with `let` used for local variable declaration. `let` lets you declare variables without initialising them. It can also contain *item declarations* such as **fn, struct, or use**.
In the following example you do not declare `name` as a mutable, yet it is updated in the constructor. It is because it is initialised in just one of the control flow path thus Rust allows it.
```rust
let name: type = expr;

let name;
if user.has_nickname() {
	name = user.nickname();
} else {
	name = generate_unique_name();
	user.register(&name);
}
```

Following is a piece of code that at first seems like erroneous, but is actually not:
```rust
for line in file.lines() {
	let line = line?;
	....
}
```
The let declaration creates a new second variable, of a different type. The type of the first variable `line` is `<Stirng, io::Error>`. The second **`line` is `String`**. The second ones declaration *supersedes* the first one for the rest of the block. It is called **shadowing** and is a commonly used practice. The code is equivalent to:
```rust
for line_result in file.lines() {
	let line = line_result?;
	....
}
```

When a function is declared inside a block its lifetime is the entire block but, it may not access any of the local expressions or arguments of the block.