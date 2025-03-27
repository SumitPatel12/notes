_____
**Created**: 25-01-2025 07:53 pm
**Status**: In Progress
**Tags**: #Interpreters [[Interpreters]]
**References**: 
______

### What it's doing at the moment, deep dive with grammar and understanding.
 In main file we've just:
- Instantiated the `Lox`class and got the file if any from the command line arguments.
- After that depending on if the arguments length was correct, run the file, or open a prompt session, or give an error.

#### Lox Class
We've got a `run_file` method, which handles running a file given in the arguments. It in turn calls the `runz` method.
`run` instantiates the `scanner` struct with the source file and then calls `scanner.scan_tokens()`. 
Scan tokens just returns a vector of tokens, which we can further use for parsing.

After a successful scan, we pass the tokes to a `parser` which gets us the statements. And these statements are then passed to an `interpreter`.


### Ramblings
- Has numeric, boolean, and string data-type.
- Has following arithmetic operations:
	- +
	- -
	- *
	- /
- Comparison and equality operators:
	- <
	- >
	- <=
	- >=
	- ==
	- !=
- Logical Operators
	- !
	- or *(maybe I'll try ||)*
	- and  *(maybe I'll try &&)*
- **Maybe implement bitwise, shift, and modulo operators in my implementation.**
- Control flow:
	- if, else
	- while
	- for
- Functions: I'm gonna use `fn`
	- Use a return statement for returning a value, otherwise it'll return `nil`.
- Classes:
	- Have methods but they are not followed by a `fn` or `fun` keyword.
	- Fields can be declared as well.
	- Use this to access fields of the class.
	- Inheritance is a thing here.
	- Super is used for calling methods from the parent class. 