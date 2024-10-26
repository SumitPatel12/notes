_____
**Created**: 17-10-2024 07:35 am
**Status**: In Progress
**Tags**: #Rust #RustErrorHandling [[Rust]] [[Rust Error Handling]]
**References**: 
______

The `?` operator is used from propagating error in Rust. You can add `?` to the end of any expression that produces a [[Result]], e.g. result of a function call. The behaviour of `?` depends on what the function returns; `success` or `error`.
- On success it unwraps the result to get the success value inside. 
- On error, it immediately returns from the enclosing function passing the `error result` up the call chain.
`?` can also be used similarly with something that returns an `Option`.

```rust
// Here weather is of type WeatherReport and not Result<WeatherReport, io::Error>
let weather = get_weather(hometwon)?;
let weather2 = get_weather(hometown).ok()?;
```

### Working with Multiple Error Types
It is likely that more than one thing can go wrong in the piece of code you are writing take for example:
```rust
use std::io::{self, BufRead}

fn read_numbers(file: &mut dyn Bufread) -> Result<Vec<i64>, io::Error> {
	let mut numbers = vec![];
	for line_result in file.lines() {
		let line = line_result?; // reading lines can fail
		numbers.push(line.parse()?); // parsing lines can fail
	}
	Ok(numbers)
}
```
You get a compiler error saying `couldn't convert the error to std::io::Error`.  Essentially it could not convert `std::nu::ParseIntError` to `std::io::Error`.
You can do one of the two things here:
- Define your own error type  that implements conversions from all the possible error types you can encounter.
- Use what's build into rust. All standard library errors can be converted to the type `Box<dyn std::error::Error + Send + Sync + 'static>`, this just represents that any error and `Send + Sync + 'static` means it is safe to pass between threads.

```rust
type GenericError = Box<dyn std::error::Error + Send + Sync + 'static>;
type GenericResult<T> = Result<T, GenericErorr>
```
You can now use the GenericResult and GenericError types but the tradeoff would be that the erorr would not communicate precisely what happened. It would be up to the user to interpret.

If you're calling a function that returns a `GenreicResult` and you want to handle one particular kind of error but let all other propagate out, use the generic method `error.downcast_ref::<ErrorType>()`. It borrows a reference to the error, if it happens to be the particular error you're looking for.