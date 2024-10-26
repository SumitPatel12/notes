_____
**Created**: 19-10-2024 02:33 pm
**Status**: In Progress
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

In memory both the [[Named-Field Structs]] and the [[Tuple-Like Struct]] are the same thing. They are represented as a collection of values of possibly mixed types, laid out in a particular way.
```rust
struct GreyscaleMap {
	pixels: Vec<u8>,
	size: (usize, usize)
}
```

 Unlike C and C++, Rust doesn't make specific promises about how it will order a struct's field in memory. The diagram below shows one of many possible combinations. 
 Rust does promise to store the **fields' values directly in the structs block of memory,** i.e. Rust embeds pixels and size directly int the `GreyscaleMap` value. Only the heap-allocated buffer owned by pixels vector remains in its own block.

![[Struct Memory Layout.excalidraw|1000]]