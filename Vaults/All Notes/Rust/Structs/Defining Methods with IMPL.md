_____
**Created**: 19-10-2024 04:24 pm
**Status**: Completed
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

As opposed to other popular languages, implementation of `struct` methods are done inside the `impl` block, which is just a collection of `fn` definitions, each of which represent a method on the `struct` type named at the top of the block.
Note that there can be more than one `impl` block for a given `struct` just that they need to be within the same [[Crates And Modules|crate]] that defines the `struct` type.

```rust
pub struct Queue {
	older: Vec<char>,
	younger: Vec<char>
}

impl Queue {
	pub fn push(&mut self, c: char) {
		self.younger.push(c);
	}

	pub fn pop(&mut self) -> Option<char> {
		if self.older.is_empty() {
			if self.younger.is_empty() {
				return None;
			}
		
			// Bring the elements in younger over to older, and put them in the promised order.
			use std::mem::swap;
			swap(&mut self.older, &mut self.younger);
			self.older.reverse();
		}

		self.older.pop()
	}

	pub fn is_empty(&self) -> bool {
		self.older.is_empty() && self.younger.is_empty()
	}

	// This is a worrying function, we'll see why.
	pub fn split(self) -> (Vec<char>, Vec<char>) {
		(self.older, self.younger)
	}
}
```

Functions defined in the `impl` block are called *associated functions*. A Rust method must explicitly use `self` to refer to the value it was called on similar to the way python methods use `self`, and JavaScript uses `this`.
Since *push* and *pop* need to borrow mutable references they both take `&mut self` as the input, while *is_empty* takes `&self`. When you call a method on the Queue you don't need to borrow the mutable or shared reference yourself. As the method signature already tells rust what is needed, Rust takes care of borrowing the right type of reference. 
```rust
// so you can write 
q.push('1');
// insted of 
(&mut q).push('1');

// similarly you can write the following without having to worry about what type of reference you are borrowing.
q.pop();
q.is_empty();
```

By now you are familiar with the borrowing and moves and it might be visible what is wrong with the `split` function. 
Split takes `self` by value, this would then move the `Queue` out o q, and leave q uninitialised. Since `split`'s `self` now own the Queue, it's able to move the individual vectors out of it and return them to the caller.
This is mighty inconvenient, so Rust lets you pass self by using [[Smart Pointers]] as well.
```rust
// leaves q uninitialised and therefore unusable further down the road.
let (older, younger) = q.split()
```


### Passing Self as a Box, Rc, or Arc
A method that takes in a `Box<Self>`, `Rc<Self>`, or `Arc<Self>` can only be called on the value of the given pointer type, calling the method would pass the ownership of the pointer to it.
You do not need to worry about method that don't explicitly use these pointers. A method that expects `self` by reference would work just fine with any of these pointer types.
```rust
let mut bg = Box::new(Queue::new());
// This works fine.
bg.push('u');
```
Scenarios:
- If it can pass [[Ownership|ownership]] of `Rc` , it simply hands over the pointer.
- If it needs to retain the [[Ownership|ownership]] of the `Rc`, it just bumps the reference count.
- If it owns the node, then we must call `Rc::new()`  to allocate heap space and move the Node into it. Since parent will insist on referring to its children via `Rc<Node>` pointers.


### Type-Associated Functions
Type associated functions are methods in the `impl` block that do not take `self` as an argument. They are widely used as constructor functions like so and are called as `StructName::TypeAssociatedFunction()`
```rust
impl Queue {
	pub fn new() -> Queue {
		Queue { older: Vec::new(), younger: Vec::new() }
	}
}

let mut q = Queue::new();
```

### Associated Consts
The constant definition is simple, in the `impl` block you define a const value and it can be accessed as `StructName::ConstName`. A constant value of a `struct` need not be of that type, you can have constants of different types in the `impl` block.
```rust
impl MyVectora {
	const ZERO: MyVector = MyVector { //predefined values. }
	const OTHER_TYPE: u32 = 10;
}

let myZeroVector = MyVector::ZEOR;
```