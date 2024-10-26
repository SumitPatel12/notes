_____
**Created**: 06-10-2024 03:27 pm
**Status**: Completed
**Tags**: #Rust #RustBorrowChecker [[Rust]] 
**References**: 
______

>Reference are to non-owning pointers that do not affect the lifetime of the referent.

### References to Values
```rust
use std::collections::HashMap;
type Table = HashMap<Stirng, Vec<String>>;

// The incorrect way
fn sohw(table: Table) {
	for (artist, works) in table {
		println!("works by {}:", artist);
		for work in works {
			println!(" {}", work);
		}
	}
}

// The correct way
fn sohw_borrowed(table: &Table) {
	for (artist, works) in table {
		println!("works by {}:", artist);
		for work in works {
			println!(" {}", work);
		}
	}
}

fn main() {
	let mut table = Table::new();
	table.insert("artist1", vec!["work1", "work2", "work3"]);
	table.insert("artist2", vec!["work1", "work2", "work3"]);
	table.insert("artist3", vec!["work1", "work2", "work3"]);
	// Correct way to do it
	show_borrowed(&table);
	asser_eq!(table["artist1"][1], "work2"); // Works fine
	
	// Moves ownership and hence making table unusableafterwards.
	show(table);
	asser_eq!(table["artist1"][1], "work2"); // Results in error
}
```
If you understand ownership then you'd know that you just destroyed the whole table variable; how you ask? See the following:
- When you call `show(table)` the ownership of table moves to the function.
- When the outer`for` loop iterates over the vector it consumes it entirely.
- The inner `for` loop does the same for each of the vectors.

To avoid something like this references are used. Common syntax for references:
- `&valueToReference` **(shared access)**: Yields a shared reference to valueToReference. At a given time you can have as many shared references to a value as you like. Note that shared references cannot modify the referenced value.
	- While an object has active shared references the value is locked down, i.e. even the owner cannot modify the value.
- `&mut valueToReference` **(exclusive access)**: Yields a mutable reference i.e. it can both read and modify the referenced value. The catch is that when you are using a mutable reference, you cannot have any other kind of reference active on that value at the same time.

### Working with References
- You can have references to references.
- Comparing "see through" any number of references.
```rust
let x = 10;
let y = 10;

let rx = &x;
let ry = &y;

let rrx = &&x;
let rry = &&y;

// This is true
assert!(rrx <= rry);
assert!(rrx == rry);

assert!(rx == ry); // Their referents are equal.
// This checks if they are pointing to the same addresss.
assert!(!std::ptr::eq(rx, ry));
```
- Rust references are never null.
	- If a value that can be null or a value of type `T`, use `Option<&T>`. At machine level `None` is a *null pointer* and `Some(r)`, where r is `&T` value, as the nonzero address.

### Borrowing References to Arbitrary Expressions
Any kind of expression can borrow values:
```rust
fn factorial(n: usize) -> usize {
	(1..n+1).product();
}
let r = &factorial(6);
// Arithmetic operations can see through one level of references.
assert_eq!(r + &1009, 1729);
```

`&1009` creates an anonymous variable to hold the value 1009. The lifetime is dependent on what happens with the expression. In our case it is dropped at the end of the `asset_eq!`.