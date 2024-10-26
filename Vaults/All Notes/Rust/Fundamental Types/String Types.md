_____
**Created**: 06-10-2024 02:25 pm
**Status**: Completed
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______

### String Literals
String literals are enclosed in double quotes `"`. A string may span multiple lines.
In a few cases, the need to double every backslash becomes cumbersome, for such cases Rust provides *raw strings*. A *raw string* is tagged with lowercase letter `r` and all backslash and whitespaces inside it are included verbatim in the string. No escape sequences are recognised. 
```rust
let raw_string = r"C:\Program\something";
let pattern = Regex::new(r"\d+(\.\d+)*");
```

To include double quotes in raw string write it as `r##..n` i.e. r followed by n pound signs. It will make it so that the string ends only when it encounters a double quote followed by that many (n) pound signs.

### Byte Strings
A string literal with the `b` prefix is a byte string. Such a string is a slice of `u8` values rather than unicode text. Raw byte strings start with `br`.
```rust
let method = b"GET";
assert_eq!(method, &[&b'G', &b'E', &b'T']);
```


### Strings in Memory
Strings are stored in memory using *UTF-8*, a variable width encoding. Each ASCII character in a string is stored as one byte. 
Consider the following code:
```rust
let noodles = "noodles".to_stirng();
let oodles = &noodles[1..];
let poodes = "ಠ_ಠ";
```

A string has a resizable buffer holding UTF-8 text. The buffer is allocated on the **heap**, so it can be resized as required. In the above piece of code, `noodles` is a *String* that owns an eight byte buffer, of which seven are in use. Under the hood it is implemented as a `Vec<u8>` that is guaranteed to hold well-formed UTF-8.

A `&str` is reference to a run of UTF-8 text owned by someone else: it "[[References|borrows]]" the text. In the example `oodles` is a &str referencing to the last six bytes of the text belonging to noodles and is a [[Arrays, Vectors, and Slices#Slices|Slice]]. `&str` can be though of as being nothing but a &\[u8\] that is guaranteed to hold well-formed UTF-8.
![[String, &str and str.jpeg|800]]

A string literal is like a `&str` that refers to preallocated text, typically storied in *read-only memory along with the programs machine code*. `poodles` is a string literal, pointing to seven bytes that are created when the **program begin execution and that lasts until it exists**. Also note that it is *impossible* to modify a `&str`.

A `String` and `&str`'s `.len()` method returns its length in **bytes** not *characters*.
```rust
asser_eq!("ಠ_ಠ".len(), 7);
asser_eq!("ಠ_ಠ".chars().count(), 3);
```

### String
`String` is analogous to `Vec<T>` as seen in the table below:

|                                                             | Vec\<T>             | String              |
| ----------------------------------------------------------- | ------------------- | ------------------- |
| Automatically frees buffer                                  | Yes                 | Yes                 |
| Growable                                                    | Yes                 | Yes                 |
| `::new()` and `::with_capacity()` type-associated functions | Yes                 | Yes                 |
| `.reserve()` and `.capacity()` methods                      | Yes                 | Yes                 |
| `.push()` and `.pop()` methods                              | Yes                 | Yes                 |
| Range syntax `v[start..stop]`                               | Yes, returns `&[T]` | Yes, returns `&str` |
| Automatic conversion                                        | `&Vec<T>` to `&[T]` | `&String` to `&str` |
| Inhetrits methods                                           | Forn `&[T]`         | From `&str`         |
Like a `Vec` each `String` has its own heap-allocated buffer that is not shared.
Following are some ways to create a `String`
- `.to_string()`: Converts `&str` to `String`.
- `.to_owned()`: does the same thing.
- `format!()` macro works just like `println!()` just that it returns a new `String`.
- Array, slices, and vectors of strings have two methods, `.concat()` and `.join(separator)`.

`Strings` support the == and !== operators, along with comparison operators  like >, <, <=, >= etc. 
