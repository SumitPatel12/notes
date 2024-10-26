_____
**Created**: 24-10-2024 04:05 pm
**Status**: Completed
**Tags**: #RustTraitsAndGenerics [[Traits And Generics]]
**References**: 
______

```rust
// Following are the same things, the ones after the first one - the more verbose ones are referred to as Qualified method calls.
"Hello".to_string();
str::to_string("Hello");
ToString::to_stirng("Hello");
<str as ToStirng>::to_string("Hello");
```

