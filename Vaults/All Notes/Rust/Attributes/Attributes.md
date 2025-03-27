_____
**Created**: 19-10-2024 10:14 am
**Status**: In Progress
**Tags**: #Rust_Attributes [[Rust]]
**References**: 
______

You can decorate items in Rust with *Attributes*. They are rust's catchall syntax for writing miscellaneous instructions and advice to the compiler.
```rust
// do not show warnign for camel case
[#allow(non_camel_case_types)]
pub struct git_revspec {...}
```