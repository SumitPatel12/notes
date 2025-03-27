_____
**Created**: 16-10-2024 09:05 am
**Status**: In Progress
**Tags**: #Rust #Rust_Closures [[Rust]] [[Rust Closures]]
**References**: 
______

*Closures* are lightweight function-like values. It usually consists of an argument list, given between vertical bars, followed by an expression.
```rust
let is_even = |x| x % 2 == 0;
```

Rust infers the argument types and return types, you can also spell them out if you want, but if you do spell them out the body of the closure needs to be a block.
```rust
let is_even = |x: u64| -> bool x % 2 == 0; // Error
let is_even = |x: u64| -> bool { x % 2 == 0 };
```

Calling a closure has the same syntax as calling a function.
```rust
assert_eq!(is_even(14), true);
```