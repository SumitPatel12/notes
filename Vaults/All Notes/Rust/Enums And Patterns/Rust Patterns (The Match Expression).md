_____
**Created**: 20-10-2024 03:52 pm
**Status**: Completed
**Tags**: #RustEnumsAndPatterns [[Rust Enums And Patterns]]
**References**: 
______

```rust
enum TimeUnit {
	Secondes,
	Minutes,
	Hours,
	Days,
	Months,
	Years
}

enum RoughTime {
	InThePast(TimeUnit, u32),
	JustNow,
	InTheFuture(TimeUnit, u32)
}
```
Given the enum suppose we need to show the the value somewhere. So, we'd like to access the `TimeUnit` and `u32` fields inside of the `RoughTime` enum. Rust prohibits direct access to these fields, as the value could possibly be `RoughTime::JustNow` which has no fields.

To circumvent this problem *pattern matching* is used, and the way to do it is using the `match` expression.
```rust
fn rough_time_to_readable(rt: RoughTime) -> Stirng {
	match rt {
		RoughtTime::InThePast(units, count) => format!("{} {} ago", count, units.plural()),
		RoughtTime::JustNow => format!("Just Now"),
		RoughtTime::InTheFuture(units, count) => format!("{} {} form now", count, units.plural()),
	}
}
```
*Expressions produce values, patterns consume values.*

Other patterns
```rust
match someIntegerCount {
	0 => {},
	1 => {...},
	n => println!("n is: {}", n), // n is a variable and serves as a catchall phrase.
}

match someStruct {
	// .. means that you dont really care about any other fields. Just match the specified ones.
	Point { x: matchValue, y: matchValue, .. } => {},
	Point { x: matchValue2, y: matchValue2 } => {},
	_ => println!("n is: {}", n), // n is a variable and serves as a catchall phrase.
}

// Slice patterns match length as well. .. in a slice pattern matches any number of elements
fn greet_people(names: &[str]) {
	match names {
		[] => {},
		[a] => { // mathces one element }
		[a, b] => { // mathces two element }
		[a, .., b] => { // mathces any number of element }
	}
}
```


### Reference Patterns
Rust supports two features for working with references:
- `ref` pattern borrows parts of a matched value.
- `&` pattern match references.

```rust
match account {
	Account { name, language } => {
		ui.greet(&name, &language);
		ui.show_settings(&account); // error: borrow of moved value: `account`
	}
}
```

Here the fields `account.name` and `account.language` are moved into local variables `name` and `language`, and the rest of the account is **dropped**. That's why we cannot borrow a reference to `account` afterwards.
If `name` and `language` were copyable values, then Rust would copy the fields instead of moving them and our code would be fine. So we need to borrow the matched values instead of moving them. `ref` is used for this, in case you want mutable references you can use `ref mut`.
```rust
match account {
	Account { ref name, ref language } => {
		ui.greet(name, language);
		ui.show_settings(&account); // works now.
	}
}
```

The opposite would be `&` it matches a reference:
```rust
match sphere.center() {
	&Point3d { x, y, z } => ...
}
```
In the above example suppose `sphere.center` returns a reference to a private field of `sphere`, a common patter you might find yourself using. The value returned is the address of a `Point3d`. If the center is at origin, then we get `&Point { x: 0, y: 0, z: 0 }`.
*This only works because x, y, and z are copyable fields. In case they were not copyable Rust would throw and error.*


### Match Guard
Sometimes we need additional conditions met before we'd consider a match to be satisfied. Say we got a hexagonal board and clicks decide that the piece on the board needs to move. To confirm a click was valid we might need more checks than just matching a valid click.
```rust
fn check_move (current_hex: Hex, click: Point) {
	match point_to_hex(click) {
		None => Err("Some error"),
		Some(current_hex) => Err("Can't move to the position you are currenty at."),
		Some(other_hex) => Ok(other_hex) // unreachabel
	}
}
```
The above given code fails because identifiers in patterns introduce new variables. The pattern `Some(current_hex)` creates a new local variable `current_hex` shadowing the argument `current_hex`. And also the last arm is unreachable as `Some(current_hex)` would match everything.

```rust
fn check_move (current_hex: Hex, click: Point) {
	match point_to_hex(click) {
		None => Err("Some error"),
		// if the pattern matches but the condition is false we move on  to the next arm.
		Some(hex) if hex == current_hex => Err("Can't move to the position you are currenty at."),
		Some(hex) => Ok(other_hex) // reachable now
	}
}
```

### Matching Multiple Possibilities
```rust
let at_end = match chars.peek() {
	Some(&'\r') | Some(&'\n') | None => true,
	'0'..='9' => println!("number"), // matches a range including '9'
	_ => false,
}
```

### Binding with \@ Patterns
`x @ pattern` matches exactly like the given pattern, but on success instead of creating variables for parts of the matched value, copies or [[Moves]] the the whole value into `x`. 
```rust
match self.get_selection() {
	Shape::Rect(top_left, bottom_right) => {
		optimized_paint(&Shape::Rect(top_left, bottom_right))
	}
	other_shape => { // do something }
}

// This could be written as
match self.get_selection() {
	rec @ Shape::Rect(..) => {
		optimized_paint(&rect)
	}
	other_shape => { // do something }
}

// Also useful with patterns
match chars.next() {
	Some(digit @ '0'..='9') => read_number(digit);
}
```


### Where Patterns are Allowed
Patterns aren't only available in Match expressions only. They are allowed in place of an *identifier*. The meaning is the same instead of storing a value in a single variable Rust uses pattern matching to take the value apart.
```rust
// Unpack into new local variables
let Track { album, track_number, title, .. } = song;

// Unpack function argument thats a tuple
fn distance_to((x, y): (f64, f64)) -> f64 { ... }

// Iterate over key, vlaues of a HashMap
for (id, document) in &cache_map { ... }

// Automatically dereference an argument to a closure.
let sum = numbers.fold(0, |a, &num| a + num);
```

Note that  the patterns we use here are *guaranteed* to match every possible value of the type we are unpacking from. These are called **irrefutable patterns**, and they are the *only* patterns allowed in the fours places shown in the example above.
A **refutable** pattern on the other hand is something that does match all possible values of the patter. They are allowed inside `if let` and `while let` expressions.