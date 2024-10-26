_____
**Created**: 14-10-2024 09:16 pm
**Status**: Completed
**Tags**: #Rust #RustExpressions [[Rust]] [[Rust Expressions]]
**References**: 
______

### If and Match
#### If
The form of if expression is the normal one used in most of the languages. Note that unlike some other languages Rust does not implicitly convert `numbers` or other such types to boolean values.
*The parentheses are not required around the condition.* Rust throws an error if you use one.
```rust
if conidtion {
	block
} else if other_condition {
	other block
} else {
	yet another block
}
```
All of the blocks in the if statement (`block`, `other block`, and `yet another block`) must return the same **value type**.
#### Match
Match is much like `switch` in other languages albeit a tad bit more flexible. Also, similar to `if blocks` the `arms` of a `match` must all return the same **value type**.
```rust
// General form of match
match value {
	pattern => expr, // This comma can be dropped if the expr is a block.
	...
}

match code {
	0 => println!("OK"),
	1 => println!("Wires Tangles"),
	2 => println!("User Asleep"),
	_ => println!("Unrecongnised error {}", code)
}
```
The compiler can optimise these kinds of `match` using a jump table. A similar optimisation is applied when each arm of the `match` produces a constant value, in which case these constants are compiled into an array and the **`match`** is compiled into an array access.

The benefit of `match` is that it supports a variety of patterns.
Also rust does not let us have a `match` that do not cover all of the possible values of the `value` in question.


#### If let
Another form of `if` is `if let` expression
```rust
if let pattern = expr {
	if_block
} else {
	else_block
}
```
In short the given expression matches the pattern to execute `if_block` or it does not an executes `else_block`.
It is the same as a `match` with just one parameter.


### Loops
Following are the four loops supported in Rust:
```rust
// The condition for while loop is strictly of type bool
// Also it works similar to C++ while.
while condition {
	block
}

// At the beginning of each loop iteration, the value of the expr matched and we execute the block or exit the loop in the other case.
while let pattern = expr {
	block
}

// Executes infinitely until a return, break is encountered or the theread *panics*
loop {
	block
}

// Evaluates the iterable expressionand then evaluates the block once for each value in the resulting iterator.
for pattern in iterable {
	block
}
```

Loops are [[An Expression Language|expressions]] in rust, just that the value of `while` and `for` loops are always `()`. A `loop` on the other hand can return a value if you specify one.
Keeping in line with the semantics of [[Moves|moves]] a `for loop` consumes the value:
```rust
let strings: Vec<String> = error_messages();
for s in strings { // Each string is moved into s here.
	println!("{}", s); // s and by extension the moved stirng is dropped here.
}
println!("{} error(s)", strings.len()); // error: use of moved value.
```

#### Control Flow in Loops
- `break` as in other languages exists the enclosing loop. You can optionally give an `expression` to the `break` which will in turn become the value of the loop. As with the other control flow statements, all `break` within a loop must return the same value type.
```rust
loop {
	break expr;
}
```

- `continue` skips the current iteration of the loop. If there are no more values, the loop exists. 

A loop can be *labeled* with a lifetime and that label can then be used in conjunction with `break` to exit the labeled loop or with `continue` to skip an iteration of the **labelled loop**. You can have both `label` and `expression` with a `break`.
```rust
'search:
for room in apartment {
	for spot in room.hiding_spots() {
		if spot.contains(keys) {
			println!("Your keys are {} in the {}", spot, room);
			break 'search;
		}
	}
}

// Find sqrt of the first perfect square.
let sqrt = 'outer: loop {
	let n = next_number();
	for i in 1.. {
		let square = i * i;
		if square == n {
			// found a square root
			break 'outer i;
		}
		if square > n {
			// n is not a perfect square try the next one.
			break;
		}
	}
}
```


### Return Expressions
`return` exits the current function and returns the given value to the caller; calling return without a value would just return `()`.
Unlike other languages Rust functions need not have explicit return statements, the body itself works like a [[Blocks and Semicolons|block expression]]: if the last expression does not have a semicolon then its value is the functions return value. 