_____
**Created**: 03-10-2024 08:33 am
**Status**: In Progress
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______
### Integer Types

| Size (bits)  | Unsigned Integer | Signed Integer | Floating-point |
| ------------ | ---------------- | -------------- | -------------- |
| 8            | u8               | i8             |                |
| 16           | u16              | i16            |                |
| 32           | u32              | i32            | f32            |
| 64           | u64              | i64            | f64            |
| 128          | u128             | i128           |                |
| Machine Word | usize            | isize          |                |

*Machine word* is a value the size of an address on the machine the code runs on i.e. 32 or 64 bits.
Unlike other languages a char is distinct from numeric types in **Rust**, it is not u8 or u32 (though it is indeed 32 bits long).

The prefix **0x**, **0o**, and **0b** designate *hexadecimal*, *octal*, and *binary* literals.

#### Type conversion
You can convert from one integer type to another using the ***as*** operator.
```rust
assert_eq!(10_i8 as u16, 10_u16); // in range
assert_eq!(2525_u16 as i16, 2525_i16); // in range
```

#### Checked, Wrapping, Saturating, and Overflowing Arithmetic
When an integer arithmetic operation overflows, Rust [[Rust Panic|panics]], in a debug build. In a release build the operations *wraps around*: it produces the value equivalent to the mathematically correct result modulo the range of the value.

```rust
let mut i = 1;
loop {
	i *= 10; // panic: attempt to multiply with overflow, only in debug builds
			 // In a production/release build multiplication wraps to a negative number hence creating an infinite loop.
}
```

Most of the times you'd want it to panic in all builds this could be done using:
```rust
let mut i: i32 = 1;
loop {
	// panic: mulitplication overflowed (in any build)'
	i = i.checked_mul(10).expect("mulitplication overflowed");
}
```

##### Checked Operations
These return an [[Option]] of the result: [[Some(v)]] if the mathematically correct result can be represented as a value type, or None if it cannot. 
```rust
assert_eq!(10_u8.checked_add(20), Some(30));
// 100 and 200 cannot be u8 so we get None.
assert_eq!(100_u8.checked_add(200), None);

// Do the addition panic if it overflows
leet sum = x.checked_add(y).umwrap();
```

##### Wrapping Operations
These return a value equivalent to the mathematically correct result modulo the range of the value.
```rust
// The first product can be represented as u16.
// Second cannot so we get 250000 modulo 2^16.
assert_eq!(100_u16.wrapping_mul(200), 20000);
assert_eq!(500_u16.wrapping_mul(500), 53392);


// Operations on signed type cna wrap to negative values
assert_eq!(100_i16.wrapping_mul(500), -12214);
```

##### Saturating Operations
These return the representable value that is closed to the mathematically correct result, in other words, the result is "clamped" to the maximum or the minimum values the type can represent.
*There are no saturating division, multiplication or shift methods.*
```rust
assert_eq!(32760_i16.saturating_add(10), 32767);
assert_eq!(-32760_i16.saturating_sub(10), -32768);
```

##### Overflowing Operations
It returns a tuple *(result, overflowed)*, where the result is what the wrapping version of the function would return, and overflowed is a boolean value indication whether or not an overflow occurred.
```rust
assert_eq!(255_u8.overflowing_sub(2), (253, false));
assert_eq!(255_u8.overflowing_add(2), (1, ture));
```

### Floating-Point Types
Rusts floating type number f32 and f64 correspond to float and double types of C and C++ as well as java.
They have the general form as diagramed:
![[Floating Type Form.excalidraw]]

Each part of the floating point number after the integer part is optional, but at least one of the fractional part, exponent or type must be present to distinguish it from a integer literal. Fractional part may consist of a lone decimal point, making `5.` a valid floating-point constant.

*Unlike C and C++, Rust performs no numeric conversions implicitly.*

