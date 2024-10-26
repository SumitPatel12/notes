_____
**Created**: 19-10-2024 05:55 pm
**Status**: Completed
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

Say you want to have a structure that is common among different types (`i32`, `u32`, `f64`, etc.), it is cumbersome to declare the same `struct` multiple times for each type. In conditions like these *Generic structs* come into play. They are like a template into which you can plug whatever types you like.

```rust
pub struct Queue<T> {
	older: Vec<T>,
	younger: Vec<T>
}

impl<T> Queue {
	// Both of the new funcitons are the same.
	// Self (notice the uppercase) refers to the type impl block is being written for.
	pub fn new() -> Queue { 
		Queue { Vec::new(), Vec::new() }
	}
	
	pub fn new() -> Self {
		Self { Vec::new(), Vec::new() }
	}

	pub fn push(&mut self, t: T) {
		self.younger.push(t);
	}
}

// Say you want some extra methods for Queue of type i32. Then you can do this as well
impl Queue<i32> {
	// Some type specific methods.
}

// Using the genreic structs
let mut qi32 = Queueu::<i32>::new();

// Or you can let the compiler decide the type for you
let mut qchar = Queue::new();
let mut qf64 = Queue::new();

qchar.push('3'); // Compiler knows its a Queue<char> now.
qf64.push(3.0); // Compiler knows its a Queue<f64> now.
```
