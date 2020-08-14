# Factory Functions and the Module Pattern

## What's wrong with constructors? 

A lot of people are against constructors because if you aren't careful, they can introduce bugs to your code. They *look* like regular functions, but they do not act like them. Forgetting the `new` keyword when using a constructor will result in errors. 

## Learning Outcomes
Answer the following: 
* Describe common bugs using constructors
    + Common bugs:
        - Forgetting the `new` keyword
        - Breaks `instanceof`
* Write a factory method that returns an object
    + See factory function introduction
* Explain how scope works in JavaScript (what has ES6 changed?)
    + Previously all variables were scoped within their function or 
    globally if not declared within a function
    + `let` and `const` allow for blockscoped variables, so if declared within an `if` statement they will only reside within those brackets
* Explain what Closure is and how it impacts private functions & variables
* Describe how private functions & variables are useful
* Use inheritance in objects using factory pattern
* Explain module pattern
* Describe IIFE, what does it stand for?
* Explain namespacing and its uses

## Factory function introduction

Factory functions are similar to constructors, but they do not use `new` to create an object. They set up and return the new object when the factory function is called. For example: 

```javascript
function myObject(data) {
  var obj = Object.create(myObject.proto);
  obj.data = data;
  return obj;
}
 
myObject.proto = {
  getData: function() {
    return this.data;
  }
}
 
var o = myObject("data");
```
or 
```javascript
const personFactory = (name, age) => {
  const sayHello = () => console.log('hello!');
  return { name, age, sayHello };
};

const jeff = personFactory('jeff', 27);

console.log(jeff.name); // 'jeff'

jeff.sayHello(); // calls the function and logs 'hello!'
```

## Scope and Closure
* Global Scope: variables defined outside of functions, accessible from anywhere
* Local Scope: Defined within brackets (within functions, `if` statements for `let` and `const`)
* Lexical Scope / Closure / Static Scope: Inner functions will have accsess to outer function variables

Closure allows you to do things like this: 
```javascript
 var updateClickCount=(function(){
    var counter=0;

    return function(){
     ++counter;
     // do something with counter
    }
})();
```
The code above is protecting the `counter` variable from the global scope by assigning the function that updates the counter to `updateClickCount`. Whenever `updateClickCount` is called, it will run `++counter` since that function was returned by the outer function. The `counter` variable is only intialized once, at the time of assignment to `updateClickCount`.

## Scope and `this`

By default, `this` refers to the outer most global object (the `window`). 
```javascript
var myFunction = function () {
  console.log(this); // this = global, [object Window]
};
myFunction();

var myObject = {};
myObject.myMethod = function () {
  console.log(this); // this = Object { myObject }
};

var nav = document.querySelector('.nav'); // <nav class="nav">
var toggleNav = function () {
  console.log(this); // this = <nav> element
};
nav.addEventListener('click', toggleNav, false);
```
However, scope can be changed like below since the `setTimeout()` function refers to the `Window` object: 
```javascript 
var nav = document.querySelector('.nav'); // <nav class="nav">
var toggleNav = function () {
  console.log(this); // <nav> element
  setTimeout(function () {
    console.log(this); // [object Window]
  }, 1000);
};
nav.addEventListener('click', toggleNav, false);
```
The second `this` refers to the `Window` instead of the `<nav>` element. In order to get the second `this` to refer to the `<nav>` element, we can assign it to a local variable: 
```javascript 
var nav = document.querySelector('.nav'); // <nav class="nav">
var toggleNav = function () {
  console.log(this); // <nav> element
  setTimeout(function () {
    console.log(this); // [object Window]
  }, 1000);
};
nav.addEventListener('click', toggleNav, false);
```