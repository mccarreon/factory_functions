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

## Changing scope with `.call()`, `.apply()`, and `.bind()`
The following codeblock iterates over the links list and tries to log each element, but fails to do so because `this` is currently referring to the object Window. 
```javascript
var links = document.querySelectorAll('nav li');
for (var i = 0; i < links.length; i++) {
  console.log(this); // [object Window]
}
```
In order to change this, we must use either the `.call()` or `.apply()` methods on a function to pass in a scope. 
```javascript
var links = document.querySelectorAll('nav li');
for (var i = 0; i < links.length; i++) {
  (function () {
    console.log(this);
  }).call(links[i]);
}
```
The `links[i]` is the current element in the array iteration and changes the scope of the function so that `this` is now the iterated element. `.call(scope, arg1, arg2, arg3)` takes individual arguments while `.apply(scope, [arg1, arg2, arg3])` takes in a list of arguments afterwards. These both invoke the function they are called on. 

`.bind()` does not invoke a function, it binds the values before the function is invoked. This is useful if we're trying to pass in arguments without calling the function such as with event listeners. 
```javascript
nav.addEventListener('click', toggleNav.bind(scope, arg1, arg2), false);
```

## Private and Public Scope
Since functions create scope, we can create a private scope by creating a function inside of a function.
```javascript
(function () {
  var myFunction = function () {
    // do some stuff here
  };
})();

myFunction(); // Uncaught ReferenceError: myFunction is not defined
```
If we want the function to be public, we can use the Module Pattern (and Revealing Module Pattern). It allows us to scope our functions correctly using private and public scope and an Object. 
```javascript
// define module
var Module = (function () {
  return {
    myMethod: function () {
      console.log('myMethod has been called.');
    }
  };
})();

// call module + methods
Module.myMethod();
```
The Module acts at the global namespace and returns the public methods which can be accessed in the global scope. These methods are `namespaced` and we can extend the Module with more methods. 

This allows us to create private and public functions. Private functions should be used for things like helpers, addClass, removeClass, Ajax/XHR calls, Arrays, Objects, etc. Public methods should only be public if necessary for the API calls. The public methods also have access to the private methods since they are in the same scope. 
```javascript
var Module = (function () {
  var _privateMethod = function () {

  };
  var publicMethod = function () {
    // can call _privateMethod
  };
  return {
    publicMethod: publicMethod,
    anotherPublicMethod: anotherPublicMethod
  }
})();
```

## Back to Factory Functions
Factories are just functions that return objects to use in code. It helps with organization. For example, in a game we would want objects to describe our players and encapsulate all thet hings a player can do. 
```javascript
const Player = (name, level) => {
  let health = level * 2;
  const getLevel = () => level;
  const getName  = () => name;
  const die = () => {
    // uh oh
  };
  const damage = x => {
    health -= x;
    if (health <= 0) {
      die();
    }
  };
  const attack = enemy => {
    if (level < enemy.getLevel()) {
      damage(1);
      console.log(`${enemy.getName()} has damaged ${name}`);
    }
    if (level >= enemy.getLevel()) {
      enemy.damage(1);
      console.log(`${name} has damaged ${enemy.getName()}`);
    }
  };
  return {attack, damage, getLevel, getName}
};

const jimmie = Player('jim', 10);
const badGuy = Player('jeff', 5);
jimmie.attack(badGuy);
```
In this example, we would not be able to edit player health or cause the player to die by calling the function because they're both private.

### Inheritance with factories 

#### Delegation
Method delegation is easy: you can define one method in the prototype that all subsequent objects that use the prototype will use. 
```javascript
const proto = {
  hello () {
    return `Hello, my name is ${ this.name }`;
  }
};

const greeter = (name) => Object.assign(Object.create(proto), {
  name
});

const george = greeter('george');

const msg = george.hello();

console.log(msg);
```
The greeter object inherits the `hello()` function from proto. This lets you create a greeter object named george and have george use the `hello()` function. Delegation is not good for storing state, mutating any member of the object or array will mutate the member for every instance that shares the prototype. You'd need to make a copy of the state for each object to preserve instance safety. 
#### Concatenative inheritance / Cloning / Mixins
Concatenative inheritance is copying properties from one object to another without retaining a reference between the objects. 

Cloning is good for storing default state for objects and is usually achieved using `Object.assign()`. 
```javascript
const proto = {
  hello: function hello() {
    return `Hello, my name is ${ this.name }`;
  }
};

const george = Object.assign({}, proto, {name: 'George'});

const msg = george.hello();

console.log(msg); // Hello, my name is George
```
