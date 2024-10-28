_____
**Created**: 27-10-2024 10:36 am
**Status**: Completed
**Tags**: [[The Pragmatic Programmer]]
**References**: 
______

### TLDR
- *Concurrency* is when two or more things act as if they are running at the same time. *Parallelism* is when they actually run at the same time.
- Decoupled code is easier to change.
- Tell, Don't Ask.
```java
// This could be written better with not so much method chaining.
customers.orders
		 .find(order_id)
		 .getTotals()
		 .applyDiscount(amt);

// Better code
customer.findOrder(order_id)
		.applyDiscount(amt);
```
- If it is possible to avoid method chaining avoid it. There are exceptions though, for example the language methods are unlikely to change and chaining them is perfectly valid.
- The true problem with *global variables and data* is that when changing them we're never sure we managed to change all of the instances.
- *If it is important enough to be global, have it wrapped up in an API.* And for the love of god, have only one team managing that API.
- Prefer interfaces to express polymorphism.
- Parameterise your app using external configurations. The thing is to keep environment and customer specific things out of the app logic in a config, then for each different instance we can read from this config and the application will run according to this config.