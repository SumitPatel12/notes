_____
**Created**: 12-10-2024 10:59 pm
**Status**: Completed
**Tags**: #Rust #RustBorrowChecker #RustLifetimes [[Rust]] [[Rust BorrowChecker]] [[Rust Lifetimes]]
**References**: 
______

### Borrowing a Local Variable
A local variable can be referenced only in its scope. This is achieved in a way by assigning each variable a *[[Rust Lifetime]]*.  The gist being that the lifetime of a reference must be contained within the lifetime of the referenced.
```rust
let r;
{
	let x = 1;
	r = &x;
}
// Following assert throws an error as we are out of scope of x.
assert_eq!(*r, 1);
```

### Receiving References as Function Arguments
When you try to assign a variable declared in a function to *static (global)* variable  we get an error. The thing is that every static must be initialised, and are by default not mutable. You can explicitly make them mutable but in that case those are only available to use in `unsafe` blocks.

```rust
static mut STASH: &i32 = &128;
// Note that this could be written as:
// f(p: &i32)
fn f<'a>(p: &'a i32) {
	unsafe {
		// Resutlts in error, because of conflicting lifetimes.
		STASH = p;
	}
}

fn f<'static>(p: &'static i32) {
	unsafe {
		// Works because both STASH and p have the same lifetime.
		STASH = p;
	}
}
```
The code given above writes out some syntax - that the language lets us omit otherwise - that is relevant to us understanding why the code is wrong.
`'a` is the lifetime parameter to the function, it can be interpreted as *for any lifetime a*. So the function takes in a parameter p with any given lifetime `'a`. As we are accepting any lifetime we need to make sure that even the smallest possible lifetime satisfies `STASH = p`. Given that `STASH` lives for the program's entire execution, the reference type it can hold must also have the same lifetime; in this case a `'static lifetime`. But lifetime of `'a` can be anything thus the compiler rejects it.

As you can see this is very helpful in reading the functions, i.e. **you know just by reading the function signature whether or not the parameters are going to outlive the function execution or not, because the lifetime signature would tell us all we need to know.**

### Passing References to Functions
```rust
// This could be written as: fn g(p: &i32)
fn g<'a>(p: &'a i32) {
	...
}

let x = 10;
g(&x);
```
From `g`'s signature we already know `p` is not being stored anywhere that might outlive the call; *any lifetime that encloses the call must work for 'a*.

### Returning References
```rust
fn smallest<'a>(v: &'a [i32]) -> &'a i32 {
	let mut s = &v[0];
	for r in &v[1..] {
		if *r < *s { s = r; }
	}
	s
}

let s;
{
	let arr = [1, 3, 0, 2, 9];
	s = smallest(&arr);
	// Passes as arr is within scope.
	assert_eq!(*s, 0);
}

// results in error as arr is already dropped.
assert_eq!(*s, 0);
```
When a function taken in a single reference as input and returns a single reference as the output, Rust assumes that the reference taken in and the reference returned by the function have the same lifetime.

### Structs Containing References
```rust
// Does not compile
struct S {
	// You must write out the lifetime otherwise Rust will not compile.
	r: &i32
}

// This will compile
struct S_Working<'a> {
	r: &'a i32
}

let s;
{
	let x = 1;

	// s = S { r = &x };
	s2 = S_Working { r = &x };
}
// Following assert throws an error as we are out of scope of x.
// Reads from dropped 'x'
assert_eq!(*s2.r, 1);
```

To define a reference type inside another types definition you must include the **lifetime** parameter in the definition.
Each instance of S that you make must now follow the lifetime constraint defined earlier. In our case `'a` will make sure that any reference to be help in `r` should enclose the lifetime of `struct S_Working`.

If you define `S_Working` within a different struct you'd have to provide a lifetime parameter on there as well. Also the enclosing struct must have it's own lifetime parameter.
```rust
// Does not compile. We need a lifetime for s.
struct Enclosing_Wrong {
	s: S_Working
}

// This gives another error stating that Enclosing_Wrong_2 still does not have its own lifetime.
// This is because we said S_Working can borrow from 'a but Enclosing_Wrong_2 can borrow from anywhere.
struct Enclosing_Wrong_2 {
	s: S_Working<'a>
}

// This works.
struct Enclosing_Wrong_2<'a> {
	s: S_Working<'a>
}
```
Similar to functions the type signature of structs will also tell us about the lifetime of its enclosing fields.

### Distinct Lifetime Paramaters
**This applies to both `structs` and `functions`.**
Consider the following piece of code
```rust
struct S<'a> {
	x: &'a i32,
	y: &'a i32
}

let x = 10;
let r;
{
	let y = 10;
	{
		let s = S { x = &x, y = &y };
		r = s.x;
	}
}
println!("{}", r);
```

One might expect it to work but it indeed does not. Reason being:
- Both fields of S are references with the same lifetime `'a`, thus the compiler must find a lifetime that encompasses both `s.x` and `s.y`.
- We reassign `r = s.x`, requiring `'a` to enclose the lifetime of `r`.
- We also initialise `s.y = &y`, requiring `'a` to be within the lifetime of `y`.
These constraints are impossible given the code thus the compiler complains.

To remedy it simply give different lifetimes to `x` and `y`, i.e. both the fields have different lifetimes making them independent:
```rust
struct S<'a, 'b> {
	x: &'a i32,
	y: &'b i32
}
```

### Omitting Lifetime Parameters
Rust lets you omit lifetime parameters whenever it is obvious which to use. In cases of multiple parameters and a return value of the function uses a reference then you must state the lifetime parameter of the return type, otherwise the compiler will infer that for you.