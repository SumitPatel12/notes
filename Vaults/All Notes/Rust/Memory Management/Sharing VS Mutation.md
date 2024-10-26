_____
**Created**: 13-10-2024 04:47 pm
**Status**: Completed
**Tags**: #Rust #RustBorrowChecker [[Rust]] [[Rust BorrowChecker]]
**References**: 
______
[[Sharing VS Mutation.excalidraw]]

### Rules of Mutation and Sharing
1. **Shared access is read only**
	- The object that is the target of a shared reference is read-only. Over the lifetime of the shared reference anything reachable from that reference can be changed by anything. The owner cannot have any mutable reference to its structure.
2. **Mutable access is exclusive access**
	- The value referenced by the mutable reference is only reachable from that one reference i.e. over the lifetime of the mutable reference no other path can exist to that value or anything that can be reached from that value. The references whose lifetimes may overlap with a mutable reference are those that you borrow from the mutable reference itself.
```rust
fn extend(vec: &mut Vec<f64>, slice: &[f64]) {
	for elt in slice {
		vec.push(*elt);
	}
}

let mut wave = Vec::new();
let head = vec![0.0, 1.0];
let tail = [0.0, -1.0];

extend(&mut wave, &head);
extend(&mut wave, &tail);

// This is incorrect as it is vilating Rule 2.
// Also rule 1 if you see as we created a read only reference to a mutable reference.
extend(&mut wave, &wave);
```

Each kind of reference affects what we can do with the values along the owning path to the reference.
- Note that in both the cases, the path of ownership leading to the reference cannot be changed for the reference's lifetime.
	- For shared reference the path is *read-only*.
	- For mutable reference the path is completely *inaccessible*.
![[Ownership and Borrowing.excalidraw|1000]]

```rust
let mut x = 10;
let r1 = &x;
let r2 = &x; // Multiple shared borrows are permitted
x += 10; // error as we cannot assign to `x` as it is borrowed.
let m = &mut x; // We have a shared reference so we cannot borrow a mutable reference.

println!("{} {} {}", r1, r2, m); // lifetime should last at least this long cause they are used here.

// Scenario 2
let mut y = 20;
let m1 = &mut y;
let m2 = &mut y; // Cannot borrow again as we already borrowed a mutable reference already.
let z = y; // Cannot assing to z since we have a mutable reference to y.

// Scenario 3: Can reborrow from a shared reference
let mut w = (107, 109);
let r = &w;
let r0 = &r.0; // Reborrowing shared as shared is allowed.
let m1 = &mut r.1; // cannot reborrow as a mutable reference.

// Scenario 4: Can reborrow from a mutable reference
let mut v = (136, 139);
let m = &mut v;
let m0 = &mut m0.0; // Can reborrow from mutable as mutable.
*m0 = 137; // This is ok to do
let r1 = &m.1 // Can't have a shared reference on mutable reference.
v.1; // Error as access to the other paths is still forbidden, m0 is not dropped yet.
```