_____
**Created**: 24-10-2024 11:41 am
**Status**: Completed 
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

Trait objects are the way to go when your usecase requires you to have a collection of mixed types.
```rust
trait Vegetable {...}

// The not so correct veriosn
struct Salad<V: Vegetable> {
	veggies: Vec<V>
}

// The error version
struct Salad {
	veggies: Vec<dyn Vegetable> // error: `dyn Vegetable` does not have a constant size.
}

// The actually correct version.
struct Salad {
	veggies: Vec<Box<dyn Vegetable>>
}
```

- The first implementation is incorrect because it only lets us use just one vegetable for a salad, which just doesn't seem that appealing.
- The second one just gives a compilation error as not all `Vegetable` types are going to be the same size.
- Third one is correct because
	- The [[Pointer Types#Box Pointer|Box]] itself has a constant size suitable for storing in a vector.
	- It would work well for many scenarios like shapes in a drawing, monsters in a game, pluggable routing algorithms in a network router.
One more reason for using traits would be to reduce the binary size. Rust has to compile a generic function may times - once for each included type - which increase the binary size. 

### Advantages of Generics over Trait Objects
Outside of these situations where you might want to *reduce the binary size* generics have some important advantages over trait objects:
1.  *Speed:* In generic functions we don't use the `dyn` keyword, this might look not so significant but indeed it is. Using generics the compiler knows exactly what function to call because the type is either specified or inferred. `dyn` means we dynamically dispatch incurring some additional overhead on the calls.
	-  The generic functions are called as if we had written separate functions for each type. The compiler can inline it like any other function.
	- Say we have a type that implements trait `Write`, and it silently discards all of the bytes written to it.
		- Rust would not know which type of the trait is being used until run time hence compile time optimisations would be limited.
		- If it were a generic Rust would know and make the relevant optimisations effectively reducing the overhead of virtual function calls.
2. Not every trait can support trait objects.
3. It is easy to bind a generic type parameter to several trait objects at once. Trait Objects do not allow such things.