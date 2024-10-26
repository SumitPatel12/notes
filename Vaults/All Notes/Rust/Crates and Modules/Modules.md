_____
**Created**: 19-10-2024 09:07 am
**Status**: Completed
**Tags**: #RustCratesAndModules [[Crates And Modules]]
**References**: 
______
Modules like in other languages is a way to organise code within your project. It is akin to `napespaces` of **Java and C#**.
```rust
mod module {
	pub struct someStruct {...}

	pub fn someFunction {...}

	pub(crate) fn someOtherFuncton {...}

	fn somePrivateFunciton {...}
}
```

- Module declared by the keyword `mod` is a collection of items. In the example above it consists of `someStruct`, `someFunction`, and `someOtherFunction`.
- The `pub` keyword makes the definition *public* which means the defined entity will be available for use outside the module as well. You can think of it as exporting the definition.
- The `someOtherFunciton` is marked as `(crate)` which means that it is available for use inside the crate only. It is not accessible outside of it i.e. it cannot be used by other crates and won't be a part of the documentation.
- Anything not marked `pub` is `private`, in this case the `somePrivateFunction`.

### Nested Modules
```rust
mod parentModule {
	pub mod child1 {...}

	pub mod child2 {...}

	pub mod child3 {...}
	
	pub mod child4 {...}
}
```

Fist things first, if you want a function or definition of a module public, that module and all of its enclosing modules must be public.
Otherwise you can use **`pub(super)`** or **`pub(in path)`** to make the definition available in the parent module or to the module specified in the path and all of its descendants.

### Modules in Separate Files
Module can also be written like this:
```rust
// in main.rs
mod spores;
```
Earlier we included the body of the module wrapped in curly braces. Here we're instead telling rust compiler that spores module lives in a separate file called `spores.rs`.
```rust
// spores.rs

pub struct Spore {...}

pub fn produce_spore {...}

pub (crate) fn genes {...}

fn recombine {...}
```

`spores.rs` contains only the items that make up the module. It doesn't need any kind of boilerplate to declare that it is a module.
A module can have its own directory. When rust sees `mod spores`; it checks for both `spores.rs` and `spores/mod.rs` if neither or both exists it is an error. For this example `spores.rs` was used cause it did not have any submodules.

### Paths and Imports
```rust
//common syntax
module::submodule::..::publicFunction

// OR
use std::mem // Import one
use std::collecitons::{HashMap, HashSet} // Multiple import
use std::fs::{self, File} // import both std::fs and std::fs::File
use std::io::prelude::* // Import everything
use std::io::Result as IOResult // Import as an alias

if s1 >  s2 {
	mem::swap(&mut s1, &mut s2);
}
```


Module do not *automatically inherit* names from their parent modules.
```rust
// proteinds/mod.rs
pub enum AminoAcid {...}
pub mod systhesis;

// protein/synthesis.rs
pub fn synthesize(seq: &[AminoAcid]) // error: can't find type AminoAcid

// Working protein/synthesis.rs
use super::AminoAcids

pub fn synthesize(seq: &[AminoAcid]) // error: can't find type AminoAcid
```

The keywords **super and crate** have a special meaning in the path. **Super** is the parent module and **crate** is the crate containing the current module.
```rust
// proteins/synthesis.rs
use crate::proteinds::AminoAcids
pub fn synthesize(seq: &[AminoAcid]) // error: can't find type AminoAcid
```

Submodules can access private members of their parent modules with `super::*`.

**Rust behaves as though every module including the root module started with the following import**
```rust
use std::prelude::v1::*
```

#### Making Use Declarations Public
```rust
pub use self::leaves::Leaf;
pub use self::roots::Root;
```
 This means that `Leaf` and `Root` are public items of the `plant_structure` module. Just that they are simple aliases.
 The standard prelude is written as a series of `pub imports`

#### Making Struct Fields Public
```rust
pub struct Fern {
	pub roots: RootSet,
	pub stems: StemSet
}
```
A structs private fields are accessible throughout the module it is declared in but outside only the ones marked as `pub` are available


#### Statics and Constants
Modules as other languages can define *statics* and *constants*.
```rust
pub const MODULE_CONSTANT: i32 = 5;
pub static MODULE_STATIC: i32 = 10;
```

- A **constant** is compiled into your code every place it is used. 
- A **static** on the other hand is something that's set up before the program starts running and lasts until it is done executing.
- Constant can be used for smaller values and statics can be used for larger constant data to which you can borrow references at will.
- There are of curse no mutable constants. **Statics** on the other hand can be mutable.
	- Rust has no way to enforce rules about exclusive access on *mutable statics,* making then non-thread safe and unusable within safe code.