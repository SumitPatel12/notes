_____
**Created**: 31-08-2024 02:06 pm
**Status**: Completed
**Tags**: #JavaScript [[JavaScript]]
**References**: [onClick={someFunction} VS onClick={()=>someFunction](https://dev.to/itric/onclicksomefunction-vs-onclicksomefunction-5d1i)
______

### Calling a function inside an arrow function vs Directly calling the function
There are couple of ways to use the desired function inside the onClick method. The most common ones are:
```js
// 1. Inside arrow ficntion with the parenthesis.
onClick={() => increaseCount()}

// 2. Directly call the function.
onClick={increaseCount()}

// 3. Inside arrow function but without the parenthesis.
onClick={() => increaseCount}
```

1. Inside arrow function we bind the function call to the user click event, meaning that the function is only called when the user clicks the bound element.
2. The function is not bound to anything but is directly in place, meaning that the function is encountered by the js engine *when it goes over the code **calling it directly***. This in many cases leads to *infinite calling and infinite re-renders*.
3. This is incorrect you cannot bind the function without the parentheses. This is likely to result in a compilation error or the code not working all together.