_____
**Created**: 10-10-2024 12:55 am
**Status**: Completed
**Tags**: #Rust #RustOwnership [[Rust]] 
**References**: 
______
### Moves 
**Move** means that the source *relinquishes* ownership of the value to the destination and gets *uninitialised*. To understand this lets see the following code.
```rust
let s = vec!["udon".to_stirng(), "ramen".to_stirng(), "soba".to_stirng()];
let t = s;
let u = s;
```

After the initialisation of `s` the memory would look something like this:
![[Move1.excalidraw|800]]

After assigning s to t it would look like this:
![[Move2.excalidraw|800]]
What happened is that when `t = s` assignment happens the headers moved from `s` to `t`, after this move the compile considers the variable `s` as *uninitialised*.
It is important to note that s is now considered uninitialised, because after `t = s` we try to do `u = s` and as Rust prohibits assigning and using *uninitialised* values we get the following error.

```rust
  |
5 |     let s = vec!["udon", "ramen", "soba"];
  |         - move occurs because `s` has type `Vec<&str>`, which does not implement the `Copy` trait
6 |     let t = s;
  |             - value moved here
7 |     let u = s;
  |             ^ value used here after move
  |
help: consider cloning the value if the performance cost is acceptable
  |
6 |     let t = s.clone();
  |              ++++++++
```

#### More Operations that Move
When reassigning values to a variable the older values are dropped.
```rust
let mut s = "some_value";
s = "other_value"; // "some_value" is dropped

let mut o = "something";
let m = o; // value "something" is moved to m and o is uninitialised.
o = "something_els"; // Nothing is dropped as o was uninitialised.
```
 
 Operations such as returning values from a function, passing arguments to a function also move the ownership. 
 So, unlike other languages you'd have to be wary writing a while loop like this in rust:
```rust
let x = vec![1, 2, 3];
while f() {
	g(x); // x will get uninitialized in the first iteration,
		  // second iteration will fail as x is uninitialized.
}

// This works as x is always assigned a new value after getting uninitialized.
while f() {
	g(x); 
	x = some_new_value;
}
```

Similarly, assigning indexed values to some other variable fails because, it wouldn't make sense to have some random index of an array or vector uninitialised. Note that this would work if the vector consisted something that implements the **copy** trait.
```rust
let mut v = vec!["1", "2", "3", "4", "5"];
let x = v[0]; // Error: Cannot move index out of vector.
let y = v[1]; // Same
```


### Copy Types
Ownership as a concept makes sense for strings, vectors, structs etc. as they are large and have the potential to consume a lot of memory, but for simpler and smaller types such as integers and characters, ownership would make it unnecessarily complicated, so instead we just copy its values. These types are called **copy types**.

Copy type values are copied so the source remains initialised with the same value it had before and can be used further in the code.
*A fixed sized array or tuple consisting of only copy types is also considered as a copy type.*

**Only types for which simple bit-by-bit copy suffices can be copy type.**
By default user defined types `struct` and `enum` are not copy type but *if all of its constituents are copy type* you can use `#[derive(Copy, Clone)]` to make it a copy type. Rust throws an error if you use it on something that is not solely having copy type fields.

### Rc and Arc: Shared Ownership
Rc stands for `reference count` and Arc stands for `atomic reference count`. They are similar in most ways just that Arc is *thread safe* while Rc is not, Arc thus has some performance implications.

For any `T`, an `Rc<T>` value is a pointer to heap-allocated `T` with a count and reference. **Cloning** `Rc` would not result in a clone but just incrementing the counter. Note that the values owned by `Rc<T>` are **immutable**.
```rust
let s = Rc::new("stringValue".to_stirng());
let t = s.clone();
let u = s.clone();
```
![[Rc And Arc Memory.excalidraw|600]]

Each of the `Rc<string>` instances refer to the same memory. The string is only dropped when all three of the `Rc<string>` instances `s, t, u` are dropped.