_____
**Created**: 15-10-2024 10:12 pm
**Status**: Completed
**Tags**: #Rust [[Rust]]
**References**: 
______
```rust
let x = someFunciton() // function cal
let y = treasure.location() // method call
let z = Vec::new() // type-associated funciton call
```

Rust makes a sharp distinction between reference an values they refer to, if you provide an `i32` to a function expecting `&i32`, it will result in a type error.
But you notice that the **`.`** operator relaxes those rules a bit. In the method call above `treasure` might be a class, [[Box]], or [[Moves#Rc and Arc Shared Ownership|Rc]], hence it might take the `treasure` by reference or value. The same **`.location`** syntax works in all cases because the **`.`** operator dereferences or borrows a reference to it as needed.

Like most of the languages method chaining is allowed. One difference being the in a function call or method, usual syntax for *generics* does not work. The problem is that `<` is considered as the less than operator, so you'd use `::` instead. Of course you can omit types altogether wherever they can be inferred.
```rust
return Vec<i32>::with_capacity(5); // error: something about chained comparisions
let ramp = (0..n).collect<Vec<i32>>(); // same error


// Correct code
return Vec::<i32>::with_capacity(5); // error: something about chained comparisions
let ramp = (0..n).collect::<Vec<i32>>(); // same error

```