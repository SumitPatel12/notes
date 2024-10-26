_____
**Created**: 20-10-2024 02:49 pm
**Status**: Completed
**Tags**: #RustEnumsAndPatterns [[Rust Enums And Patterns]]
**References**: 
______

*Rust can represent `Option<Box<u32>>` as a single machine word: 0 for `None` and non-zero for `Some` pointer.*
As with [[Rust Structs|structs]] *enums* can be generic as well. Examples from the std library and one for user defined `BinaryTree` of any type `T`.
```rust
enum Option<T> {
	None,
	Some(T),
}

enum Result<T, E> {
	Ok(T),
	Err(E),
}

enum BinaryTree<T> {
	Empty,
	NonEmpty(Box<TreeNode<T>>),
}

// Part of the binary tree.
struct TreeNode<T> {
	element: T,
	left: BinrayTree<T>,
	right: BinrayTree<T>,
}
```

Lets unpack some of the information available to us in the `BinaryTree` and `TreeNode` definitions:
- Each `BinaryTree` is either *Empty* or *NonEmpty*.
	- If it is empty it does not contain any data.
	- If it is non empty, it has a [[Pointer Types#Box Pointer|Box pointer]] to a heap-allocated `TreeNode`.
- Each `TreeNode` value contains one actual element of type `T` and two `BinaryTree` values.
- This means that each *NonEmpty* tree can have any number of descendants. 

Building the tree:
```rust
use self::BinaryTre::*;
let jupiter = NonEmpty(Box::new(TreeNode {
	element: "Jupiter",
	left: Empty,
	right: Empty,
}))

let mars = NonEmpty(Box::new(TreeNode {
	element: "Mars",
	left: jupiter,
	right: mercury,
}))
```
In line with the rules of [[Ownership]] and [[Moves]] the assignment of `jupiter` and `mars` transfers to a new parent node.