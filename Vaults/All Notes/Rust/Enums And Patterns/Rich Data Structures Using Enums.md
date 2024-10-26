_____
**Created**: 20-10-2024 02:19 pm
**Status**: Completed
**Tags**: #RustEnumsAndPatterns [[Rust Enums And Patterns]]
**References**: 
______

Enums are also useful for implementing tree-like data structures. Say you need a JSON with arbitrary data, then in memory any JSON can be represented as:
```rust
enum Json {
	Null,
	Boolean(bool),
	Number(f64),
	String(String),
	Array(Vec<Json>),
	Onject(Box<HashMap<String, Json>>),
}
```

JSON standards specify that the data type that can appear inside a JSON data are `null, bool, number, string, array of JSON values, and objects`. The *Enum* just spells those out. The example is not just spun up for the sake of it, `serde_json` a crate widely used for JSON serialisation in rust uses a very similar *Enum*.

The [[Pointer Types#Box Pointer|Box]] around [[HashMap]] that represents the Object servers only to make all the JSON values more compact. In memory values of type Json take up four machine words. `String` and `Vec` values are three words and Rust adds one more for the `tag` byte. `Boolean` and `Null` would not take all the space but Json values must be the same size. The extra space goes **unused**.
If we did not use [[Pointer Types#Box Pointer|Box]] around [[HashMap]] then JSON would be eight words but as [[Pointer Types#Box Pointer|Box]] is a single word (a pointer to the heap-allocated data).

![[Json Memory Layout.excalidraw|800]]