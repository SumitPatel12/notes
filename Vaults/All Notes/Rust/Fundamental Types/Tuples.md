_____
**Created**: 05-10-2024 07:45 am
**Status**: Completed
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______

Each element of a tuple can have a different type. Tuples only allow constants as indices i.e. `t.0, t.1, etc.` are valid but `t.i or t[i]` are invalid.
You can use pattern matching syntax to assign each element of a return tuple type to different variables.

```rust
// Consider the function, which splits a string at given index and returns a tuple of type (stirng, stirng)
fn split_at(&self, mid: usize) -> (&str, &str)

let text = "I see the eigenvalue in thine eye";
let (head, tail) = text.split_at(21);
assert_eq!(head, "I see the eigenvalue");
assert_eq!(tail, "in thine eye");
```

One of the other commonly used tuple is the *zero-tuple ()*. This is traditionally called the *unit-tuple* because it has only one value, also written as **()**. It is used when there is no meaningful return type but we require a type nonetheless.
For example a function with no return type would have a return type of **()**.
```rust
fn swap<T>(x: &mut T, y: &mut T) -> ();
```