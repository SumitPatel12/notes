_____
**Created**: 02-10-2024 04:57 pm
**Status**: Completed
**Tags**: #JavaScript #JavaScriptArrayFunctions
**References**: [Stop Abusing .map()](https://dev.to/catchmareck/stop-abusing-map-51mj)
______
### Example of a wrong way to use .map()
```js
const fruitIds = ['apple', 'oragne', 'banana'];

// .map always returns an array.
// .forEach would have been a better fit here.
fruitIds.map((id) => {
   document.getElementById(`fruit-${id}`).classList.add('active');
});
```
On first glance it seems like a simple iteration to add a certain call to DOM elements. We will see what is wrong with this as we go on with the article.
### What is wrong with using .map()
Before going on to what is wrong with map lets define what a map is:
> Mapping is an operation which applies a given function to all the elements in the collection and ***return a new collection*** with elements changed by that function.

Note the `returns a new collection` part. It always returns a new collection irrespective of you using it or not. This is the reason why the first code snippet is wrong. The `.map()` method would unnecessarily return a new collection, we could have used `.forEach()` and it would have been a better fit as its designed for such cases.

### Implications of abusing .map()
First off it cannot be stressed more **.map() always returns a new collection**, meaning for an array of size *n* the function will return a new array of size *n*(not necessarily different array, in the above given case the array returned will be duplicate of the original one).
Also in JS all functions implicitly return a value, even if it does not use a return statement, in such cases it returns `undefined`. This also applies to callbacks - **they are functions too**.

The above given function would return:
```js
// This is because the callback passed to the .map does not have a return statement and as we saw earlier undefined is returned in those cases.
[undefined, undefined, undefined]
```
This takes memory and for larger arrays this could have serious performance issues.