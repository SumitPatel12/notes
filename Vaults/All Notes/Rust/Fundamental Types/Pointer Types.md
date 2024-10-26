_____
**Created**: 05-10-2024 10:20 am
**Status**: Completed
**Tags**: #Rust #RustTypes [[Rust]] [[Rust Types]]
**References**: 
______

There is a big difference between Rust and most languages with `garbage collection`, in Java if class *Rectangle* contains a field **Vector2D upperLeft**; then **upperLeft** is a reference to another separately created `Vector2D` object. Objects don't physically contain other objects.

For instance consider the following:
```rust
let value = ((0,0), (1440, 900));
```
The values here are nested by default, meaning that the variable `value` stores the data as **4 adjacent integers** i.e. *we've got a variable that is four integers wide*.
**Nothing is allocated on the heap.**

This is great for memory efficiency but, the trade-off is that whenever Rust needs a value pointing to other values it is required to use *pointers explicitly.* The good part is that in safe Rust pointers are constrained to eliminate undefined behaviour making it easier to use.

### Reference Pointers
You can think of reference type as Rust's basic pointer type.
At run time a reference to i32 is a single machine word holding the address of the i32 , which may be on the stack or the heap. &x produces a reference to x; in Rust terminology , it is referred to as **`borrows a reference to x`**. For a reference r, `*r` refers to the value r points to.

Unlike C pointers, Rust references are never `null`. Also, Rust tracks the ownership and lifetime of a value, so mistakes like dangling pointers, double frees, and pointer invalidation are ruled out at compile time.

Rust references come in the following two flavours:
1. **&T**: An immutable shared reference. You can have many shared references to a given value at any given time. The caveat being they are read-only.
2. **&mut T**: This is a *mutable exclusive reference*. The value can be both read and modified only so long as the reference exists and you may not have any other references of any kind to that value. In fact the only way to access that value would be through the mutable reference.

### Box Pointer
Box is the simplest way to allocate a value in heap. The syntax being `Box::new`.
```rust
let t = (12, "eggs");
let heapTuple = Box::new(t); // Allocate a tuple in the heap.
```

The call to `Box::new()` allocates enough memory to contain the tuple in the heap. When `heapTuple` goes out of scope, the memory is freed immediately, unless `heapTuple` was moved - by returning it, for example. [[Moves]] are essential to the way Rust handles heap-allocated memory/values.


### Raw Pointers
Rust also has raw pointer types as **`*mut T`** and **`*const T`**. Raw pointers are just like pointers in C++. Using a raw pointer is *unsafe*, because Rust makes no effort to track what it points to, meaning they may be `null` or point to freed memory among other things. 
Note that raw pointers can only be *dereferenced* in an unsafe block. 