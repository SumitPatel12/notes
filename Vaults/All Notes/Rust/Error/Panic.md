_____
**Created**: 16-10-2024 09:55 pm
**Status**: Completed
**Tags**: #Rust #RustErrorHandling [[Rust]] [[Rust Error Handling]]
**References**: 
______

Some examples of when panic occurs:
- Out of bounds array access
- Integer division by zero
- Calling `.expect()` on a `Result` that is and `Err`
- Assertion failure

In case of panic Rust can either **unwind the stack** or **abort the process**. *Unwinding* is the default.

### Unwinding
```rust
fn pirate_share(total: u64, crew_size: usize) -> u64 {
	let half = total / 2;
	half / crew_size as u64
}
```
 The above code has one problem if `crew_size` turns out to be zero we will encounter a divide by zero error. As discussed earlier this will result in panic.
- An error message will be printed to the terminal. Optionally with the `RUST_BACKTRACE=1` set it will also dump the stack trace.
- The stack is unwound.
	Any temporary values, local variables, or arguments the function was using are dropped in the reverse order of creation. Once the current function call is cleaned up we move up to its caller rinse and repeat up the stack.
- Finally the thread exits. If the panicking thread was the main thread then the whole process exits.

**Panic is safe.** It does not violate any of Rusts safety rules; even if you manage to panic in the middle of the std library method, it will *never leave a dangling pointer or a half initialised value in memory.* Rust catches anything bad before it happens and unwinds, thus letting the program as a whole to continue running.

**Panicking is per thread.** A thread can be panicking while other threads run as if nothing happened. Sometimes you might want to catch the unwind and let the thread keep running `std::panic::catch_unwind()` lets you do this.


### Aborting
In the following two cases Rust does not try to `unwind`.
1. If a `.drop()` method triggers a second panic while Rust is cleaning up after the first.
2. If you compile with the `-C panic = abort` the first panic aborts the program. 