_____
**Created**: 19-10-2024 06:32 pm
**Status**: Completed
**Tags**: #RustStructs [[Rust Structs]]
**References**: 
______

Say you have a spider robot 
- It has a control system as a central struct, `SpiderRobot`. It contains the settings and I/O and is set-up at boot and is never changed afterwards. 
- Every major system of the robot is handled by a different `struct`, and each one had a pointer back to the `SpiderRobot`.
- The `struct` for web construction, predation, venom flow control, etc. also all have a `Rc<SpiderRobot>` smart pointer. [[Moves#Rc and Arc Shared Ownership|Rc]] is always shared and immutable.
```rust
use std::rc::Rc

pub struct SpiderRobot {
	species: String,
	web_enabled: bool,
	leg_devices: [fd::FileDesc; 8],
	...
}

pub struct SpiderSense {
	robot: Rc<SpiderRobot>,
	eyes: [Camers; 32],
	...
}
```

Now suppose we need some logging on the `SpiderRobot` struct, using the standard `File` type. The problem we face here is that `File` has to be mutable but `Rc<SpiderRobot>` is immutable. 
We need some mutable data inside of a otherwise immutable value. This scenario is referred to as **interior mutability**. Rust offer several. flavours of it. We will discuss `Cell<T>` and `RefCell<T>` here. 

`Cells` and `RefCells` and any `type` that contain them are **not thread safe**.

#### Cell\<T>
A `Cell<T>` is a struct that contains a single private value of type `T`. The only special thing about a `Cell` is that it provides methods to `get` and `set` the contained field, even if you do not have mutable access to the `Cell` itself. Following are the common methods:
- **`Cell:new()`**
- **`cell::get()`**: Returns a copy of the value in the cell.
- **`cell::set()`**: 
	- Stores the given value in the `Cell` and drops the previous value.
	- Takes `self` as a non-mutable reference.

Say we wanted to add a simple counter to our `SpiderRobot`. A `Cell` would be handy in that case.
```rust
use std::cell::Cell;

pub struct SpiderRobot {
	...
	hardware_error_count: Cell<u32>,
	...
}

// Now even non-mutable methods can access this.
impl SpiderRobot {
	// Note the refrence is &self and not &mut self.
	pub fn add_hardware_error(&self) {
		let n = self.hardware_error_count.get();
		self.hardware_error_count.set(n+1);
	}
}
```

This is good but `.get()` gives us a copy and not the actual thing so it only works if `T` implements the copy trait which is not true for our logging case.


#### RefCell\<T>
`RefCell<T>` allows borrowing references to `T`. Some methods:
- **`RefCell::new(value)`**
- **`ref_cell.borrow()`** : Returns a shared reference to the value in the `RefCell`.
- **`ref_cell.borrow_mut()`** : Returns a mutable reference to the value in the `RefCell`. 
- **`ref_cell.try_borrow()`, `ref_cell.try_borrow_mut()`** 
	- Works like `borrow` and `borrow_mut` but returns a [[Result]]. Instead of panicking if a mutable reference is already borrowed, it returns an `Err`.

You can see a lot of similarities between `RefCell` and normal borrows. The only difference is that *borrows are checked at compile time and `RefCell` provide the same guarantees at run time*.

```rust
pub struct SpiderRobot {
	...
	log_file: RefCell<File>,
	...
}

// Now even non-mutable methods can access this.
impl SpiderRobot {
	// Note the refrence is &self and not &mut self.
	pub fn log(&self, message: &str) {
		let mut file = self.log_file.borrow_mut();
		writeln!(file, "{}", message).unwrap();
	}
}
```