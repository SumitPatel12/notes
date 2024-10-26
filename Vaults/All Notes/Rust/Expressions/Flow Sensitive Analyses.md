_____
**Created**: 15-10-2024 09:57 pm
**Status**: Completed
**Tags**: #Rust [[Rust]]
**References**: 
______
In Rust the flow-sensitive analyses do not examine the loop conditions at all, instead it simply assumes. that any condition in a program can be either *true* or *false*. This sometimes would result in bogus errors.
```rust
for wait_for_process(process: &mut Process) -> i32 {
	while true {
		if process.wait() {
			return process.exit_code();
		}
	}
} // error: mismatched types: expected i32, found ()
```

The above code is a valid program as the only way it exits is through the return statement. The loop statement is offered as a **say what you mean** solution to this.

Rust's type system is affected by control flow too. In [[If, Match and Loop]] we saw that these expressions must have the same type returned from all possible execution branches but it would be silly to enforce this rule on blocks that end in break or return expressions, an infinite loop, or a call to [[panic!()]] or [[std::process::exit()]]. The common theme among them is that they do not finish in the usual way, producing a value.

So in Rust, these expressions do not have a normal type. Expressions that don't finish normally are assigned a special type **!, and are exempt from the rule of types having to match.** *!* means that the function is **divergent**. And Rust considers a divergent function to be erroneous if it returns normally. 