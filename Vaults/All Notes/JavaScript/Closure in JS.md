_____
**Created**: 31-08-2024 05:37 pm
**Status**: In Progress
**Tags**: #JavaScript #Programming_Concepts [[JavaScript]] [[Programming Concepts]]
**References**: [Closure in JS](https://blog.hubspot.com/website/javascript-closure) [Fireship Closures](https://youtu.be/vKJpN5FAeF4)
______


### What is Closure?
To understand *closures* you need to understand [[Scope#^50264a |Lexical Scope]] first.

#### Pure/Self-contained Function
Before getting to closure lets understand what a self contained function would do.

```ts
// Assume this is a self contained function.
function fn(val: string, val2: string) {
	// ... do something.
}
```

If you look at the above function it is self contained. So, when we make a call to that function it will be pushed to call stack and the internal data of that function `val` and `val2` will be put in the stack memory. As soon as the function is done executing the memory will be cleared.


#### Impure function (Closure)
*A closure is not just a function but the function combined with its outer state/scope.*
Closures are generally used for encapsulation. Consider the following function

```ts
function buildGreeting(message) {
  return function (audience) {
    return message + " " + audience;
  };
}
let greeting1 = buildGreeting("Hi");
let greeting2 = buildGreeting("Hello");
console.log(greeting1("User")); // Hi User console.log(greeting2('World')); // Hello World
```

This function accesses variables from it's outer scope `message` variable. As opposed to the prior pure function this one accesses it's outer scope thus the when the call to this function is made, we will need to have preserved the state of the outer scope. To accomplish this the interpreter creates a closure, now the variables of the outer scope would be stored on the **Heap** instead of the **Stack** to preserve the variables for a longer duration. The garbage collector will reclaim these variables at the time when they are no longer needed.

#### Questions
##### First Question `var` keyword with closure 
```ts
for(var i = 0; i < 3; ++i) {
	// Creates a closure because log depends on the value of i.
	const log = () => {
		console.log(i);
	}

	// Queues a task to execute log after 100 miliseconds.	
	setTimeout(log, 100);
}

// This is equivalent to the above for loop.
var i;
for(i = 0; i < 3; ++i) {
	// Creates a closure because log depends on the value of i.
	const log = () => {
		console.log(i);
	}

	// Queues a task to execute log after 100 miliseconds.	
	setTimeout(log, 100);
}
```

**Here `i` will be in the global scope**
The output of this function will be 3, 3, 3 because how var works is that it hoists the `var i` inside the for to the outer scope, and since closure combines the function with the outer scope (of which `var i` is not a part of) we get 3, 3, 3 because after the *for loop* is executed the value of *i* will be 3. To make it more clear it can be understood as us mutating a *global variable i* over and over and the *log* function then printing it out. The closure does not bind the value of `i` as it is not a part of the *for loop*.

##### Second Question `let` keyword with closure
```ts
for(let i = 0; i < 3; ++i) {
	// Creates a closure because log depends on the value of i.
	const log = () => {
		console.log(i);
	}

	// Queues a task to execute log after 100 miliseconds.	
	setTimeout(log, 100);
}
```

**Here `i` will be in the block scope**
The output of the following will be 0, 1, 2 as expected. This is because unlike var let is scoped to the for loop thus the closure will bind the value of the variable `i` to the *log* function.