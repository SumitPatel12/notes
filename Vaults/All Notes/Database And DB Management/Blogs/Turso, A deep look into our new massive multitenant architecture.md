_____
**Created**: 19-10-2024 08:36 pm
**Status**: Completed
**Tags**: #DatabaseArchitecture #Rust [[Database Architecture]] [[Rust]]
**References**: [A Deep Look into Our New Massive Multitenant Architecture](https://turso.tech/blog/a-deep-look-into-our-new-massive-multitenant-architecture)
______

### Turso High Level Overview
Once a person signs up they are assigned/spun up a new VM. Inside that VM are the [[SQLite]] files which can then be interacted with over HTTP or be replicated to local devices.
The VM scales to 0 after certain period of inactivity.


### Example Bug
One of their customers created ~5000 DBs in a couple of hours. Sometime after creating a couple of them in quick succession the database creation would [[Deadlock|deadlock]]. 
This was a special case though. Their [[Deadlock|deadlock]] occurred in a single [[Mutex|mutex]], the offending code is shown below.

```rust
let blocking_task = tokio::task::spawn_blocking ({
	let mutex = mutex.clone();
	move || loop {
		erprintln!("Blocking thread started.");
		let guard = mutex.lock().unwrap();
		some_sync_function();
		drop(guard);
		erprintln!("Blocking thread ended.");
	}
})
```

The problem with this is not so apparent. Before we see what is wrong with this piece of code, you must know that in Rust, you are not supposed to hold a *synchronous* `mutex` across an `.await`.
Now lets look at the problem, the code directly doesn't have anything wrong with it. The offending part is what happens in the **`some_sync_funciton()`**. It looks like this:
```rust
fn some_sync_function() {
	// block_on implicitly calls .await making us break the rule stated above.
	tokio::runtime::Handle::current().block_on(someAsyncFunction);
}
```

The code of `some_async_function()` calls `block_on` which is an implicit `await`. This makes us break the rule we just talked about.
Let's see why it failed.
- Say all [[Rust Futures|futures]] resolved immediately and no control was yielded to the executor, we don't have a problem.
- The problem arises when we have stragglers. Say one of the futures did not resolve immediately, then we have a deadlock. Now it's really difficult to know when the future will resolve immediately, it depends on too many things.


### TigerStyle
At the core of this style we have [[Deterministic Simulation Testing (DST)]]. 

>The appeal of DST is that you can write the system in such a way that all I/O operations are abstracted. You can then plug a simulator into the I/O loop. The simulator has a seed, and given the same seed, you guarantee that **every single thing will always execute in the same way, in the same order, and with the same side-effects.**

>And then comes the beautiful part: you can generate random traces, and get the system to do things so wicked that not even the most degenerate chat member on ThePrimeagen stream could think of. And if anything breaks, the seed will allow you to replay exactly what happened, as many times as you want.

### The Cost of Using Async Rust
#### Direct Cost
Async rust is not without cost. There is a great benchmark that compares the cost of the async abstraction to manually rolling your own event loop. You can read the whole thing [here](https://github.com/jkarneges/rust-async-bench), but a great summary is found in those two lines:
```
manual                  time:   [26.262 µs 26.270 µs 26.279 µs]
...
box                     time:   [96.430 µs 96.447 µs 96.469 µs]
```

The first one is sync rust and the second one is async abstraction. Now for most of the use cases microseconds won't matter much but, for [[SQLite]] the operations cost in the order of microseconds so it is detrimental to the performance of Turso.

#### Indirect Cost
Async Rust forces everything to have a `static` [[Rust Lifetime|lifetime]]. This causes variables to hold onto memory for a lot longer than you might want. 
Static allocates things on the heap, which Turso is trying to control. As variables get passed on the memory holds. An example:
```rust
let a = Rc::new(something);
let inner = a.clone();

handle = task::spawn(async move { inner_usage(inner); });
other_usage(a);
```
The memory above is now very hard to account for, and will not be freed for as long as someone holds a reference.


#### Inherent Non-Determinism
The async executor can also be a source of non-determinism. 
Within a thread if we have work stealing scheduling enabled, the async executor also has its own task scheduler meaning that the code is now open to issues that may arise from. the timing of scheduling.