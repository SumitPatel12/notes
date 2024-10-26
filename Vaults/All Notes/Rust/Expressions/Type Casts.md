_____
**Created**: 16-10-2024 08:57 am
**Status**: Completed
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______

Converting from one type to another generally requires explicit type casting in rust. The operator for this is `as`.
```rust
let x = 17;
let index = x as usize;
```

- Numbers may be cast from any of the built-in numeric types to any other.
- Values of a bool or char, or of a C-like enum type may be cast to any integer type.
	- Casting in the other direction (int to bool and so on) is not allowed. The exception being `u8`, all `u8` values can be cast to characters.
- Some casts involving unsafe pointers are also allowed. See [[Raw Pointers]].

We mentioned before that we generally need casting; the exceptions are:
- `&String` auto-convert to `&str` without a cast.
- `&V3c<i32>` to `&[i32]`
- `&Box<Chessboard>` to `&Chessboard`