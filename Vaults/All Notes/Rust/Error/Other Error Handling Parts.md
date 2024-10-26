_____
**Created**: 17-10-2024 07:59 am
**Status**: Completed
**Tags**: #Rust #RustErrorHandling [[Rust]] [[Rust Error Handling]]
**References**: 
______
### Dealing with Errors that "Can't Happen"
Consider a scenario where you know for sure that the next line you're reading is an integer string because its a config file and you know it. 
The problem is that the parse method `std.parse<u64>` doesn't return a `u64`. But we know for a fact we are parsing digits only. The choice would then be to use `unwrap()`, a Result method that panics if the result is Err, but simply returns the success value of an `Ok`.

### Ignoring Errors
Not writing now, doesn't seem that useful.

### Handling Errors in main()
Normally `main()` cannot use `?` because its return type is not [[Result]]. The simplest way would be to use **`.except()`**, an error would cause an error to be printed and exit the program with a nonzero code. You can change the signature of the `main()` method to a `Result` type but it is not advisable in most cases.

### Declaring Custom Error Types
Say you are writing your own parser and want it to have its own error type:
```rust
#[derive(Debug, Clone)]
pub struct JsonError {
	pub message: String,
	pub line: usize,
	pub column: usize,
}

reutrn Err(JsonError {
	message: "message",
	line: cur_line,
	column: cur_col
})
```
The struct will be called `json::error::JsonError` and can be raised as in the example above.
If you want the error to work like standard errors then there is a bit more work to do.

```rust
iml fmt::Display for JsonError {
	fn fmt(&self, f: &mt fmt::Formatter) -> Result<(), fmt::Error> {
		write!(f, "{}, ({}, {}))", self.message, self.line, self.column)
	}
}
```
