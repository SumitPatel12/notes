_____
**Created**: 08-10-2024 08:59 am
**Status**: Completed
**Tags**: #Rust #RustOwnership [[Rust]]
**References**: 
______
Ownership generally means that the object owning the pointer to a object decides when to free it. The owned properties/possessions are gone along with the owner. In Rust every value has a *single owner* and the value is **dropped** (freed) when the owner is dropped. This helps identify the lifetime of a value just by looking at the code. 

A variable owns its value. When the control leaves the block in which the variable is declared, the variable is dropped and by extension its value. Consider the following code:
```rust
fn print_padovan() {
	let mut padovan = vec![1, 1, 1]; // allocated here
	for in in 3..10 {
		let next = padovan[i - 3] + padovan[i - 2];
		padovan.push(next);
	}
println!("P(1..10) - {?}", padovan);
} // dropped here
```

Memory Layout:
![[Ownership Memory Layou.excalidraw|1000]]

As explained before the vector own the buffer holding its elements when `padovan` goes out of scope at the end of the function, the program. drops the vector and its owned buffer. Note that the *stack frame* holds the variable `padovan` and the pointer to the heap space is present on the stack as well.

For a more complex scenario consider the following code:
```rust
struct Person {
	name: String,
	birth: i32
}

let mut composers = Vec::new();
composers.push("Palestrina".to_stirng(), 1525);
composers.push("Dowland".to_stirng(), 1563);
composers.push("Lully".to_stirng(), 1632);
```
Here composer is a Vector of structs each of which holds a string and a number. In memory the final value of composer looks like this:

![[Vec<Struct>.excalidraw|1000]]
In the above snippets you can observe the following ownerships:
- `cpmposers` owns a vector.
- The vector owns its elements, which in this case are structs.
- The struct in turn owns its constituent fields.
- The string field of the struct owns its text.
When the control leaves the scope of declaration, it drops composers and everything its owned entities owned along with it.

You can think of this as a tree structure. The owner is the parent and the values owned are children, these children can in turn have their own owned values (you can see a tree now), at the root of the tree is a variable. When the variable is **dropped** the whole **tree** goes with it.

This seems rigid at a glance but Rust provides the following to compliment this:
- You can [[Moves|move]] values from one owner to another. This allows rearranging, building and tearing of the tree we discussed earlier.
- Simple types like integer, floating-points, and characters are excluded from ownership. They are called as *Copy* type.
- The standard Library provides reference-counted pointers **[[Rc]]** and **[[Arc]]** which allow a value to have multiple owners under certain restrictions.
- You can borrow references to a value, these are non owning pointers with limited lifetimes.