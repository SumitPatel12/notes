_____
**Created**: 24-10-2024 04:12 pm
**Status**: Completed
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

As the title suggests, traits can be more than just interfaces. They can describe the relationship between types.
- The `std::itr::Iterator` trait related each iterator type with the type of value it produces.
- The `std::ops::Mul` trait relates types that can be multiplied.
- The `rand` crate includes both a trait fro random number generators (`rand::Rng`) and a trait for types that can be randomly generated (`rand::Rng`). The traits themselves define exactly how these types work.

### Associated Types: How Iterators Work
Rust's standard `Iterator` trait is defined as follows:
```rust
pub trait Iterator {
	// Item is an associated type.
	type Item;

	fn next(&mut self) -> Option<Self::Item>;
	...
}

impl Iterator for Args {
	type Item = String;

	fn next(&mut self) -> Option<String>;
}

// Generics can also make use of the assoicated types.
fn collect_into_vec<I: Iterator>(iter: I) -> Vec<I::Item> {
	...
}
```
Here `type Item` is an *associated type*. Each type that implements `Iterator` trait now has to specify what type of value it is going to produce.

Another example and its variations:
```rust
fn dump<I>(iter: I)
	where I:Iterator
{
	for (index, value) in iter {
		// error: `<I as Iterator>::Item` cannot be formatted using {?} because it doesn't implement `Debug`.
		println!("{}: {?}", index, value); 
	}
}

// Correct way of doing it.
fn dump<I>(iter: I)
	where I:Iterator, I::Item: Debug
{
	for (index, value) in iter {
		// Works now.
		println!("{}: {?}", index, value); 
	}
}

// Iterate over stirngs only.
fn dump<I>(iter: I)
	where I:Iterator<Item = String>
{
	for (index, value) in iter {
		// Works only for Stirngs.
		println!("{}: {?}", index, value); 
	}
}

// Can be used with Generics as well.
fn dump(iter: Iterator<Item = Stirng>) {
	for (index, value) in iter {
		println!("{}: {?}", index, value); 
	}
}
```


### Generic Traits: How Operator Overloading Works
```rust
// Multiplication Generic Trait
// RHS = Self mean that by default it is self, but is subject to change if the implementor sees fit.
pub trait Mul<RHS=Self> {
	tupe Output;

	// The method for the `*` operator.
	fn mul(self, rhs: RHS) -> Self::Output;
}
```

Generic traits get a special dispensation to the the *orphan rule.* You can implement a foreign trait for a foreign type os long as one of the traits type parameter is type defined in the current crate.

### impl Trait
Combination of too many generics would make the code messy and hard to read, `impl trait` comes into play here:
```rust
use std::itr;
use std::vec::IntoIter;

fn cyclical_zip(v:Vec<u8>, u: Vec<u8>) -> 
	iter::Cycle<iter::Chain<IntoIter<u8>, IntoIter<u8>>> {
		v.into_iter().chain(u.into_iter()).cycle()
}

// This is not a good implementation, you makethe code cleaner to look at, but at what cost?
fn cyclic_zip_trait_obj(v: Vec<u8>, u: Vec<u8>) -> Box<dyn Iterator<Item = u8>> {
	v.into_iter().chain(u.into_iter()).cycle()
}

// This is the best approach among these three.
fn cyclic_zip_impl_trait(v: Vec<u8>, u: Vec<u8>) -> impl Iterator<Item=u8> {
	v.into_iter().chain(u.into_iter()).cycle()
}
```

The second method `cyclic_zip_trait_obj` while looking good had a very high performance penalty. The heap allocations and overhead of dispatching dynamically would soon become massive problems. Not so worthy tradeoff.

So, for situations like these Rust provides an alternative `impl trait`. It allows us to "erase" the type of a return value specifying only the `trait(s)` it implements without dynamic dispatch or the heap allocations.

Do not mistake `impl trait` for a convenient shorthand. It has an overwhelming advantage for uses in libraries and places where some basics might remain constant but the type itself is subject to change. Using *impl trait* means that you can change the output type in the future as long as the one you intend to replace it with also implements the same trait. This reduces the refactors required down the line.

### Associated Consts
```rust
trait Cursor {
	const MAX_SIZE: &'static i32 = some_size;
	// Some functions
	...
}

trait Float {
	const ZERO: Self;
	const ONE: Self;
}

impl Float for f32 {
	const ZERO: f32 = 0.0;
	const ONE: f32 = 1.0;
}

// Can be used with generic code
fn add_one<T: Float + Add<Output=T>>(value: T) -> T {
	value + T::ONE
}
```
Associated constants cannot be used with trait objects as once again compiler needs to know some information beforehand which trait object usage fails to provide.