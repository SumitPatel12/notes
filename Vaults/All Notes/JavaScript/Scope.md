_____
**Created**: 31-08-2024 06:11 pm
**Status**: In Progress
**Tags**: #Programming_Concepts [[Programming Concepts]]
**References**: [Lexical Scope](https://stackoverflow.com/questions/1047454/what-is-lexical-scope) [Scope](https://press.rebus.community/programmingfundamentals/chapter/scope/)
______

### What is scope?
Scope in programming language can mean a particular region inside of that program. We are generally more concerned with *variable/method scope*.
- **Variable scope** dictates whether a particular variable or method is available within a certain **scope/region**. The variable scope also gives an indication of the **lifetime** of a variable or function.
List of common scopes you will come across:
1. Global scope
2. Class scope
3. Method scope
4. Block scope
5. Lexical scope ^50264a
6. Dynamic scope

#### Global Scope
As the name suggests its the top level scope, everything defined within the top level of the program lives in this scope. 
*Not all variables and functions* live within this scope. Don't make the mistake of thinking that everything defined within the program will be accessible here. The telling property of something within the global scope is that it's accessible everywhere in the program. For e.g. in javascript file a variable defined outside of all functions and classes would be in the global scope.

In Java the *static main* method is the entry point of the program hence it is in the global scope. It's easy to confuse, since it is defined inside of a class should it not be class scoped and not global scoped? Normally it'd be but the ***static*** keyword lifts it to the global scope or more precisely to the scope of its parent. Think about it, how do you generally access static members? `Class.Static_Member` is the way to do it, so wherever the class i.e. its parent is accessible the static member will also be accessible. 

#### Class Scope
As the name suggests a variable or method accessible everywhere within the class will be called as class scoped. Note that not everything declared within the class is class scoped, there can be block or function scoped variables as well.

#### Method/Function Scope
Variable available everywhere within the function or method is method scoped.

#### Block Scope
Each language has its own syntax for defining blocks, `{...block code}` being the most common one among them. Anything defined within the enclosing blocks would be scoped to that block.

```ts
// a and b are method scoped
function fn(a: string, b: number) {
	console.log(a, b);
	{
		// block_variable is block scoped 
		var block_variable = something;	
	}
	// block_variable is not accessible here.
	// a, b are accessbile here.
}
```

#### Lexical Scope
First off *Lexical Scope* is also commonly referred to as *Static Scope*. Lexical scope tells us that an inner level can access its outer levels. The reason why it is called as static scope is because the scope can be determined at compile time.

```ts
void fun()
{
	// x is available to the inner fucntion.
    int x = 5;

    void fun2()
    {
		// f3val is not available to the outer function.
		var f2val = something;
        printf("%d", x);
    }
}
```


#### Dynamic Scope
Dynamic scope is something that cannot be determined at compile time. It will be more clear when we see it by an example.

```ts
// This function prints the value of the variable x, but we cannot determine which x it will print.
// Reason being the outer scope of this function is not fixed, it is dynamic.
void fun()
{
    printf("%d", x);
}

void dummy1()
{
    int x = 5;
	// Here it will print 5.
    fun();
}

void dummy2()
{
    int x = 10;
	// Here it will print 10.
    fun();
}
```