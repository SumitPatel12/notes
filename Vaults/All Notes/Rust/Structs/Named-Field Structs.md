_____
**Created**: 19-10-2024 01:57 pm
**Status**: Completed
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

The definition of a named struct is as follows:
```rust
struct NamedSturct {
	fieldName: FieldType,
	fieldName2: FieldType2
}

// If you have a variable with the same name as a field you can use a shorthand for the sturct initialisation.
NamedStruct {fieldName, fieldName2}
// This is shorthand for
NamedStruct {fieldName: fieldName, fieldName2: fieldName2}


// To access a field
NamedStruct.fieldNamel;
NamedStruct.fieldName2;
```

Like most of the other items, `sturct` and its `fields` are private by default and are available only to the [[Modules|module]] enclosing it and its submodules. You can make them public by using the `pub` modifier. Even if you declare a struct as `pub` its fields can still be private.
```rust
// The struct is public but its fields are private
pub struct GreyscaleMap {
	pixels: Vec<u8>,
	size: (usize, usize)
}

// The struct and its fields are public.
pub struct GreyscaleMap {
	pub pixels: Vec<u8>,
	pub size: (usize, usize)
}
```

A **struct** value can only be created using *all o the struct's fields*. In the above example the first `struct` can be used by other modules but they will not be able to create a struct using the struct expression.

When creating a named-field struct, you can use another struct of the same type to supply values for fields that you omit.
```rust
struct Broom {
	name: String,
	height: u32,
	health: u32,
	position: (f32, f32, f32),
	intent: BroomIntent
}

// say b is an existing Broom type then you can do the following
let mut broom = {name: "some name", ..b};
```