_____
**Created**: 24-10-2024 01:23 pm
**Status**: Completed
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

**When you implement a trait, either the trait or the type must be new in the current crate.** This is because it helps Rust ensure that the implementations are unique, otherwise there will be many implementation of the same type leading to confusion.
Trait can be thought of as an interface in C# or Java at the beginning but they have more advanced features in Rust which we will dive into as we move along with the article. 
We give trait a name and define the type signatures of the methods. Sometimes they come with the default implementation of the functions they are defining.
```rust
trait Cursor {
	fn draw_cursor(&self, &location: Point);

	fn toggle_blinking_curosr(&self);
}

// Implementing the trait
impl Cursor for FatCursor {
	fn draw_cursor(&self, &location) {...}

	fn toggle_blinking_curosr(&self) {...}
}

impl Cursor {
	// Define other helper methods.
}
```

Note that the implementation block for the trait must contain the implementation of *all of the methods defined in the trait that need to be defined (default implementations are also a thing* and nothing else. If you want helper methods on the type you will need to use a **different `impl` block**, these helper methods can be used in the trait implementation block.

### Default Methods
Lets try this with writing the a `Write` implementation that discards all written bytes.
Here we can use `wrtie_all` without defining it because of the **default implementation**.
```rust
use std::io::{Write, Result}

trait Write {
	// Some definintions

	// What we are interested in.
	fn write_all(&mut self, buf: &[u8]) -> Result<()> {
		let mut bytes_written = 0;
		while bytes_writte < buf.len() {
			bytes_written += self.write(&buf[bytes_written..])?;
		}
		Ok(())
	}

	// More definitions
}

impl Write for Sink {
	fn write(&mut self, buf: &[u8]) {
		// Calim to have successfuly writte the whole buffer😆
		Ok(buf.len())
	}

	fn flush(&mut self) -> Result<()> {
		Ok(())
	}
}

fn main () {
	let mut out = Sink;
	// we are allowed to write this without implementing the method 🤔.
	out.write_all(b"Byte String")?;
}
```

### Traits and Other Peoples Types
Say you want to add a particular method to all of the types of certain kind, you can do that by using a concept called *extension trait.*
Consider you want all `Write` to have one more method, `write_in_format`. You can achieve this by:
```rust
use std::io::{self, Write}

trait WriteInFormat {
	fn wirte_in_format(&mut self, buf: &[u8], format: YourFormat) -> io::Result<()>;

	// some default implementaiton for the function.
}

// Applying to all types
impl<W: Write> WriteInFormat for W {
	fn wirte_in_format(&mut self, buf: &[u8], format: YourFormat) -> io::Result<()> {
		// Your implementation
	}
}
```


### Self in Traits
`Self` can be used as a type for the trait. Using it as a return type means that it is the same as itself. But any trait that uses the `Self` type becomes incompatible with trait objects.
```rust
pub trait Splicable {
	fn splice(&self) -> Self;
}

impl Splicable for Type1 {
	fn splice(&self) -> Self {...} // Return type is Type1
}

impl Splicable for Type2 {
	fn splice(&self) -> Self {...} // Return type is Type2
}

// Does not work
// ❎ The trait `Splicable` cannot be made into object
fn splice_anything(left: &dyn Splicable, right: &dyn Splicable) {
	let combo = left.splice(right);
}
```

This error is because we do not know if `left` and `right` are of the same type, and hence `&self` can possibly be different which the compiler of course doesn't allow.

### Subtraits
Subtraits are just traits that are extensions of other traits.
```rust
// Each creature of the game world is visible by default.
// Creature is a Subtrait of the trait Visible
trait Creature: Visible {
	fn position(&self) -> (i32, i32);
	fn facing(&self) -> Direction;
	...
}
```

Here `Creature: Visible` means that all creatures are visible. Every type that implements the `Creature` trait must also implement the `Visible` trait.

### Type Associated Method Calls
Traits can include type associated methods. 
```rust
trait StringSet {
	fn new() -> Self;
	fn from_slice(strings: &[&str]) -> Self;
	fn contains(&self, stirng: &str) -> bool;
	fn add(&mut self, stirng: &str); 
}
```

Trait Objects on the other hand are not something that can have type associated functions.