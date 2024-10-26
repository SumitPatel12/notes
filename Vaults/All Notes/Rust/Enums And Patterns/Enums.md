_____
**Created**: 20-10-2024 01:33 pm
**Status**: Completed
**Tags**: #RustEnumsAndPatterns [[Rust Enums And Patterns]]
**References**: 
______

*You cannot convert from integer to enum directly in Rust. You will have to write a function yourself for that.*
The members of the enum are called **variants**
### C-Style Enums
```rust
// C style enums.
/*
These are stored as integers. You can additionally tell the compiler what integers to use.
*/
enum BookType {
	// The enclosed types i.e. the following fields are called constructors or variants.
	PaperBack = 200,
	HardBound = 300,
	LeatherBound = 400,
	Digital = 500
}
assert_eq!(size_of::<BookType>, 2) // since 300, 400, 500 cannot fit in u8.

// Compiler will implement features like == operator only if asked.
#[derive(Copy, Clone, Debug, PartialEq, Eq)]
enum TimeUnit {
	Secondes,
	Minutes,
	Hours,
	Days,
	Months,
	Years
}
```
By default C-style enums are stored as integers and the compiler chooses the smallest built-in integer type that can accommodate them. `u8` in the case above.

### Enums With Data
Say we need to display time to the millisecond in some places and at some places just and approximation, like `3 days ago`. Enums can be used in this case
```rust
#[derive(Copy, Clone, Debug, PartialEq)]
enum RoughTime {
	InThePast(TimeUnit, u32),
	JustNow,
	InTheFuture(TimeUnit, u32)
}

let four_score_and_seven_years_ago = RoughTime::InThePast(TimeUnit::Years, 87);
let three_hours_from_now = RoughTime::InThePast(TimeUnit::Hours, 3);
```

You can see that some of the variants of the enum take arguments. These are called *tuple variants*
Enums can also have `struct variants` which contain named fields.
```rust
enum Shape {
	Sphere: { center: Point, radius: f32 },
	Cuboid: { topLeft: Point32, bottomRight: Point32 }
}
```


### Enums in Memory
In memory enums with data are stored as a small integer **tag**, plus enough memory to hold all the fields of the largest variant. The *tag* field is for Rust's internal use. It indicated which constructor created the value and thereby which fields it contains.
Rust makes no promise about the enum layout. Some generic structs can be stores without *tags*.

![[Enum Memory Layout.excalidraw|800]]