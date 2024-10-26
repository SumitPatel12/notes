_____
**Created**: 19-10-2024 02:20 pm
**Status**: Completed
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

Type definiton:
```rust
struct Bounds(pub usize, usize);

let image_bounds = Buonds(1024, 768);
```

The values help by it are called elements, and are accessed similar to how you would access elements of a [[Tuples|tuple]]. Individual elements may be public or not.
One might notice that `Bounds(1024, 768)` looks strikingly similar to a function call. It in fact is, defining a tuple-like struct also implicitly defines a function:
```rust
fn Bounds(ele0: usize, ele1: usize) -> Bounds {...}
```

At the fundamental level [[Named-Field Structs]] and [[Tuple-Like Struct]] are very similar. 