_____
**Created**: 23-10-2024 08:49 am
**Status**: In Progress
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

The trait for writing bytes is called `std::io::Write`, and its definition in the standard library starts out like:
```rust
trait Write {
	fn write(&mut self, buf: &[u8]) -> Result<usize>;
	fn flush(&mut self) ->Result<()>;
	....
}

// Using anything that uses the Write trait
// &mut dyn Write means it axcepts anything that implements the std::io::Write trait.
fn print_something_using_write(out: &mut dyn Write) {
	out.write(b"byte string");
	out.flush();
}
```

### Using Traits
There is one unusual rule about using trait methods; the trait itself must be in scope. Otherwise, all its methods are hidden.
```rust
// Uncomment below given line to fix the error.
// use std::io::Write

let mut buf: Vec<u8> = vec![];
buf.write_all(b"Byte String"); // Gives error: No method named `write_all`
```


### Trait Objects
Rust doesn't permit variables o `dyn Write`. A variables size needs to be known at compile time and declaring as `dyn Write` conflicts with this, as different implementations of the trait would have different sizes.
```rust
use std::io::Write

let mut buf: Vec<u8> = vc![];
let writer: dyn Write = buf; // Error: `Write` does not have a constant size.
```

This may look odd the first time around, Java and C# allow this; that's because they use pointers to the type implementing the trait, while in Rust pointers are explicit. So, we'd need to write out a reference for it to work in rust.
```rust
use std::io::Write

let mut buf: Vec<u8> = vc![];
let writer: &mut dyn Write = &mut buf; // Error: `Write` does not have a constant size.
```

#### Trait Object Layout
In memory, a trait is a [[Fat Pointer]] consisting of a pointer to the value plus a pointer to a table representing the value's type. Each trait object therefore takes up **two machine words.**

![[Trait Memory Layout.43.11.excalidraw|800]]