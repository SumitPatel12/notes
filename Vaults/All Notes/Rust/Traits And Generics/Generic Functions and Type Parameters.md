_____
**Created**: 23-10-2024 10:54 pm
**Status**: Completed
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

```rust
fn say_hello<W: Write> (out: &mut W) -> std::io::Result<()> {
	out.write(b"Hello Generic!");
	out.flush();
}

// More than one type chekc on a generic
fn top_ten<T: Debug + Hash + Eq>(values: &Vec<T>) {...}

// Mulitple generics.
fn run_query<M: Mapper + Serialize, R: Reducer + Serialize>(
	data: &DataSet, map: M, reuce: R
) -> Results
{...}

// Alternative way to do it
fn run_query(data: &DataSet, map: M, reuce: R) -> Results
where M: Mapper + Serialize,
	  R: Reducer + Serialize
{...}

// Type aliases can be generic too
type YourResult<T> = Result<T, SomeError>;
```

The phrase `<W: Write>` makes the function generic, and is called a `type parameter`. It signifies that in the function `say_hello`, `W` stands for some type that implements the *Write* trait.

The type of `W` is generally inferred by the language without need of pointing out explicitly what it is, buy sometimes you might need to spell it out for the compiler.
```rust
// Like calling a genreic method collect<C>() that takes no arguments
let v1 = (0..1000).collect(); // Error cannot infer type.
let v2 = (0..1000).collect::<Vec<i32>>(); // This is acceptable.
```

