_____
**Created**: 19-10-2024 10:24 am
**Status**: In Progress
**Tags**: #Rust #RustTests [[Rust]]
**References**: 
______

Tests are ordinary functions marked with `#[test]` attribute. `carge test` runs all the tests in your project, it works the same whether the project under test is a crate or library.
Functions marked with `#[test]` are compiled conditionally, `cargo build --release` would skip the testing code.