_____
**Created**: 05-10-2024 04:03 pm
**Status**: Completed
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______

### Arrays
The type `[T; N]` represents an array of `N` values of type `T`. The size of an array is *constant* and is determined at compile time and is a part of the type itself, meaning you cannot *append* to an array.

**Declaration**
```rust
let array: [u32; 6] = [1, 2, 3, 4, 5, 6];

// As you see we need to specify a default initialization value.
let mut sieve = [true; 10000];
for i in 2..100 {
	if sieve[i] {
		let mut j = i * i;
		while j < 10000 {
			sieve[j] = false;
			j += i;
		}
	}
}
assert!(sieve[211]);
assert!(!sieve[9876]);
```
>Rust has no notation to declare an uninitialised array, in general Rust does not provide ways to access uninitialised memory.

The normal methods of arrays such as sort, iterate, search, filter etc. are directly provided on *slices and not arrays*. Note that you can still use these methods on arrays, just that the definition is present of slices.

### Vectors
A `Vec<T>` is a resizable array of elements of type `T`, allocated on the **heap**.
**Declaration**
```rust
// vec! is equivalent to calling Vec::new and then pushing elements onto it.
let mut primes = vec![2, 3, 5, 7];
let someVector = vec![0; rows * cols];

// Another way is to build a vector from values of an iterator.
// We need to specify the type because .collect can bluild many different types.
let v: Vec<i32>  = (0..5).collect();

// Using Vec::with_capacity
let mut vector_with_capacity = Vec::with_capacity(2);
```

Unlike arrays vectors are resizable so, you can push to an array using the `.push()` method.

As with arrays, you can use slice methods on vectors:
```rust
let mut palindrome = vec!["a man", "a plan", "a canal", "panama"];
palindrome.reverse();
// Reasonable yet dissapinting
assert_eq!(palindrome, vec!["panama", "a canal", "a plan", "a man"]);
```
The `reverse` method is defined on slices, but the call implicitly *borrows* a `&mut [str]` slice from the vector and invokes the `reverse` on that.
A `Vec<T>` consists of three values:
1. A pointer to the heap-allocated buffer for the elements, which is created and owned by the `Vec<T>`.
2. The number of elements that the buffer has the capacity to store.
3. Number of elements the `Vec<T>` actually contains now i.e. the length.
When the buffer capacity is reached, adding another element to the vector entails allocating a **larger buffer**, **copying** the present contents into it, updating the pointer and capacity to that of the new buffer, and finally **freeing** the old one.

### Slices
A slice written as `[T]` without specifying length, is a region of an array or a vector. Since slices can be of *any length* they can't directly be stores in variables or passed as function arguments, they are always **passed by reference**.

A reference to a Slice is a [[Fat Pointer]] (a two word value comprising a pointer to the slices first element, and the number of elements in the slice).
```rust
let v: Vec<f64> = vec![0.0, 0.707, 1.0, 0.707];
let a: [f64; 4] = vec![0.0, -0.707, -1.0, -0.707];

let sv: &[f64] = &v;
let sa: &[f64] = &a;
```
The last two lines automatically convert the `&Vec<f64>` and `&[f64; 4]` reference to slice references that point directly to the data.
![[Slice Memory Layout.png|800]]
An ordinary reference is a non-owning pointer to a single value, while a reference to a slice is a non-owning pointer to range of consecutive values in memory.
Trying to borrow a slice that extends past the end of the data results in a [[Rust Panic|panic]].