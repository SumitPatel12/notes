_____
**Created**: 05-10-2024 07:35 am
**Status**: Completed
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______

### Introduction
It represents a single Unicode character, as a 32-bit value.
Rust uses `char` type for single characters but uses **UTF-8 encoding** for strings. So, a string is represented as a sequence of **UTF-8 bytes** and not as an array of characters.

Rust never implicitly converts between `char` and other types. You can still use the `as` operator for manual conversion to integer; for types smaller than 32-bits the upper bits are truncated.
```rust
asser_eq!('*' as i32, 42); // True
```

**`u8`** is the only type that Rust will convert to `char`. 