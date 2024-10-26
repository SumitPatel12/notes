_____
**Created**: 19-10-2024 02:28 pm
**Status**: Completed
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

The declaration of this will seem a bit off. It is a struct with no elements.
```rust
struct UnitLikeStruct;
```

*A value of such a type occupies no memory, much like `()`.* Rust doesn't store a unit-like struct in memory or generate any code to operateon them ecause it can tell every thing it might need to know about the value form its type alone.