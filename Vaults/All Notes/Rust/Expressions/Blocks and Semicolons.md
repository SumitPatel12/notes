_____
**Created**: 14-10-2024 09:37 am
**Status**: In Progress
**Tags**: #Rust #RustExpressions [[Rust]] [[Rust Expressions]] 
**References**: 
______
A **block** produces a value and can be used anywhere the value is needed.
```rust
let display_name = match post.author() {
	Some(author) => author.name, // This is a simple expression.
	// This is a block
	None => {
		let netweork_info = post.get_netweork_metadat()?;
		let ip = network_info.client_address();
		ip.to_string()
	}
};
```

The value of the block is the last expression, in this case `ip.to_string()`. Note that there is no *semicolon* after `ip.to_string()`.
Rust does use semicolon as other languages does just that it has one more caveat, if all expressions in a block end in semicolons then the value of that block is **`()`**; if the last statement of the block does not end in a semicolon then the value of that block is **the value of that expression.**

