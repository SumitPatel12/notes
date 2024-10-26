_____
**Created**: 16-10-2024 10:39 pm
**Status**: Completed
**Tags**: #Rust #RustErrorHandling [[Rust]] [[Rust Error Handling]]
**References**: 
______

Result doesn't have exceptions and instead the function that can fail have a return type that say so.
```rust
fn get_weather(location: LatLang) => Result<WeatherReport, io::Error> {...}
```
`Result` type indicates possible failure. In the above example `get_weather` will either return a **`Ok(weather)`** or **`Err(error_vlaue)`**, where error_value is of type io::Error.
### Catching Errors
The most through way of dealing with errors is using the [[If, Match and Loop#Match|match]] expression. You can think of it as Rusts equivalent of `try, catch`.
```rust
match get_weather(hometown) {
	Ok(report) => {
		display_weather(hometown, &report);
	}
	Err(err) => {
		println!("error quering the weather {}", err);
		schedule_weather_retry();
	}
}
```

Many of you would have observed that `match` is a bit verbose, so [[Result]] offers a variety of methods that are useful in particular common cases.
- *result.is_ok(), result.is_err()*
	- Returns a bool telling if result is success or an error.
- *result.ok(
	- Returns success value if any, as an `Option<T>`. If [[Result]] is success result, returns `Some(success_value)` otherwise returns `None`.
- *result.err()*
	- Returns error value if any as `Opiton<T>`.
- *result.unwrap_or_fallback(fallback
	- Returns success value if the result is success, otherwise returns a fallback and discards the error value.
- *result.unwrap_or_else(fallback_fn)*
	- Same as above just instead of the fallback value we have a fallback function.
- *result.unwrap()*
	- Returns success result in case of success but panics otherwise.
- *result.except(message)*
	- Same as `.unwrap`, but lets the user provide the message to print in case of panic.
- *result.as_ref()*
	- Converts `Result<T, E>` to `Result<&T, &E>`.
- *result.as_mut()*
	- Borrows a mutable reference `Result<T, E>` to `Result<&mut T, &mut E>`.
**Note that all methods listed above except `.is_ok, .is_err, .as_ref, .as_mut` consume the result they operate on.**

### Result Type Aliases
Type Alias is a shorthand for type names, module often define a `Result` type alias to avoid having to repeat an error type that's used consistently by almost every function in the module.
```rust
pub type Result<T> = result::Result<T, Error>;
```

### Printing Errors
- *println!()*: All standard error types are printable using this. Using `{}` prints a brief message, while `{?}` prints in debug view.
- *err.to_stirng()*: Returns error message as string..
- *err.source()*: Returns `Option` of the underlying error if any, that caused that error.
Printing an error value does not print its **source**, if you want to print all the available information you might have to write something of your own.