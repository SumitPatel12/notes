_____
**Created**: 24-10-2024 06:07 pm
**Status**: Completed
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

Say we defined a function for a certain type but now we need it for more types, we could do that with generics but sometimes the default values we use change with types. For scenarios like this we are provided the standard `Default` trait for types.
```rust
use std::ops::{Add, Mul}

// This does not work, multuiplyin two of the same type can yield a different type. 
// After resolving that you might face the problem of moves as N is not necessarily a Copyable type.
fn dot<N: Add + Mul + Default>(v1: &[N], v2: &[N]) -> N {
	let mut total = N::default();
	for i in 0..v1.len() {
		total = total + v1[i] * v2[i];
	}
	total
}

// This works
fn dot<N>(v1: &[N], v2: &[N]) -> N {
	where N:Add<Output=N> + Mul<Output=N> + Default + Copy
		let mut total = N::default();
		for i in 0..v1.len() {
			total = total + v1[i] * v2[i];
		}
		total
}
```