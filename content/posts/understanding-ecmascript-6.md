---
title: "Ecmascript 6"
date: 2019-08-18T18:45:13+01:00
draft: true
tags: ["javascript", "ecmascript-6"]
---

## Function
Some `functions` have two internal-only methods `[[Call]]` and `[[Construct]]`.

* `[[Construct]]` method is called when function is called via `new` (to create object), otherwise `[[Call]]` is called for normal function call.
* Arrow functions do not have `[[Construct]]` method.
* `new.target` is meta property
* block level functions are allowed and are `hoisted`.
* function expressions using `let` are not hoisted.

### Arrow functions

* no `this`, `super` and `new.target`
* cannot be called with `new`
* no prototype
* returning just an object in arrow function syntax

```javascript
var getTempItem = id => ({ id: id, name: "Temp" });
```
* Arrow functions can access `arguments` of its contaning function.
* `typeof <function>` returns `function`
* `<function> instanceof Function` returns `true`

## Object

* Computed properties
```
var lastName = "last name";
var person = {
    "first name": "John",
    [lastName]: "doe"
}
console.log(person[lastname]);
console.log(person["first name"]);
```
* object's prototype is stored in an internal property `[[Prototype]]`
* methods in object defines an internal property called `[[HomeObject]]`. 
* The prototype of plain JavaScript objects (like person above) is `Object.prototype`.

## Object destructuring

```
let node = {
    type: "Identifier",
    name: "foo"
};

let {type, name} = node;
```

```
let node = {
    type: "Identifier",
    name: "foo"
};
let type = "literal", name = 5;

({type, name}) = node;
```

```
let node = {
    type: "Identifier",
    name: "foo"
};
let {type: localType, name: localName} = node;
```

### Array destructuring
```
let colors = ["red", "blue", "green"];
let firstColor = "black";
let secondColor = "white";
[firstColor, secondColor] = colors;

let [first, ...rest] = colors;
```
A powerful feature of array destructuring is that it does not actually require an array! You can use any iterable object on the righthand side of the assignment; any object that can be used with a for/of loop  can also be destructured:

```
let [first, ...rest] = "Hello"; // first == "H"; rest == ["e","l","l","o"]
```

## Symbol

```
let firstName = Symbol("first name");
let person = {};
person[firstName] = "NIck";
```
* A symbol’s description is stored internally in the `[[Description]]` property.
* ECMAScript 6 provides a global symbol registry that you can access at any point in time.

```
let uid = Symbol.for("uid");
```
* symbols are created using `Symbol()` function. However there is `Symbol` class as well. But `new Symbol()` is not allowed and raises error.

## Events/Asynchronous
```
let button = document.getElementById("my-btn");
button.onclick = function(event) {
    console.log("clicked");
}

Above function might be called like this:

button.onlick(eobject);
```

## Promises
* A promise takes a function as its parameter. This function is called `executor`. This executor function further takes two parameters (provided by the compiler). This params are genereally written as `resolve` and `reject`.
```
const p = new Promise(function(reslove, reject) {
  AnAsyncFunction((err, value) => {
    if err reject(err)
    else resolve(value)
  })
})
``` 
```
const p = new Promise((resolve, reject) => {
  AnAsyncFunction((err, value) => {
    if err reject(err)
    else resolve(value)
  })
})
```
* `await` is applicable to promise not function.

## Inheritence
The class syntax sugar does reduce boilerplate when creating a prototype chain:

```
class Wolf {
  constructor (name) {
    this.name = name
  }
  howl () { console.log(this.name + ': awoooooooo') }
}

class Dog extends Wolf {
  constructor(name) {
    super(name + ' the dog')
  }
  woof () { console.log(this.name + ': woof') }
}

const rufus = new Dog('Rufus')

rufus.woof() // prints "Rufus the dog: woof"
rufus.howl() // prints "Rufus the dog: awoooooooo"
```

This will setup the same prototype chain as in the Functional Prototypal Inheritance and the Function Constructors Prototypal Inheritance examples:

```
console.log(Object.getPrototypeOf(rufus) === Dog.prototype) //true
console.log(Object.getPrototypeOf(Dog.prototype) === Wolf.prototype) //true
```

To describe the full prototype chain:

    the prototype of rufus is Dog.prototype
    the prototype of Dog.prototype is Wolf.prototype
    the prototype of Wolf.prototype is Object.prototype.

The extends keyword makes prototypal inheritance a lot simpler. In the example code, class Dog extends Wolf will ensure that the prototype of Dog.prototype will be Wolf.prototype.

The constructor method in each class is the equivalent to the function body of a Constructor Function. So for instance `function Wolf (name) { this.name = name }` is the same as `class Wolf { constructor (name) { this.name = name } }`.

The super keyword in the Dog class constructor method is a generic way to call the parent class constructor while setting the this keyword to the current instance. In the Constructor Function example `Wolf.call(this, name + ' the dog')` is equivalent to `super(name + ' the dog')` here.

Any methods other than constructor that are defined in the class are added to the prototype object of the function that the class syntax creates.

Let's take a look at the Wolf class again:

```
class Wolf {
  constructor (name) {
    this.name = name
  }
  howl () { console.log(this.name + ': awoooooooo') }
}
```

This is desugared to:

```
function Wolf (name) {
  this.name = name
}

Wolf.prototype.howl = function () {
 console.log(this.name + ': awoooooooo')
}
```
The class syntax based approach is the most recent addition to JavaScript when it comes to creating prototype chains, but is already widely used.

## `Null` and `undefined`
The null primitive is typically used to describe the absence of an object, whereas undefined is the absence of a defined value. Any variable initialized without a value will be undefined. Any expression that attempts access of a non-existent property on object will result in undefined. A function without a return statement will return undefined.

## `this` 
It's crucial to understand that `this` refers to the object on which the function was called, not the object which the function was assigned to:

```
const obj = { id: 999, fn: function () { console.log(this.id) } }
const obj2 = { id: 2, fn: obj.fn }
obj2.fn() // prints 2
obj.fn() // prints 999
```
Both obj and obj2 to reference the same function but on each invocation the this context changes to the object on which that function was called.

Functions have a call method that can be used to set their this context:

function fn() { console.log(this.id) }
const obj = { id: 999 }
const obj2 = { id: 2 }
fn.call(obj2) // prints 2
fn.call(obj) // prints 999
In this case the fn function wasn't assigned to any of the objects, this was set dynamically via the call function.

Lambda functions do not have their own this context, when this is referenced inside a function, it refers to the this of the nearest parent non-lambda function.

```
function func1() {
  return function(num) {
    console.log(this.id + num)
  }
}

function func2() {
  return (num) => {
    console.log(this.id + num)
  }
}

let obj1 = { id: 99}

let retFunc1 = func1.call(obj1)
retFunc1(1) // prints NaN
console.log('============')
let retFunc2 = func2.call(obj1)
retFunc2(1) // prints 100
```
While normal functions have a prototype property, fat arrow functions do not.

The `this` keyword is not scoped the way variables are, and, except for arrow functions, nested functions do not inherit the this value of the containing function. If a nested function is invoked as a method, its this value is the object it was invoked on. If a nested function (that is not an arrow function) is invoked as a function, then its this value will be either the global object (non-strict mode) or undefined (strict mode). It is a common mistake to assume that a nested function defined within a method and invoked as a function can use this to obtain the invocation context of the method.

Arrow functions inherit the this value of the context where they are defined.


## Symbol
To obtain a Symbol value, you call the Symbol() function. This function never returns the same value twice, even when called with the same argument.

The Symbol() function takes an optional string argument and returns a unique Symbol value. If you supply a string argument, that string will be included in the output of the Symbol’s toString() method. Note, however, that calling Symbol() twice with the same string produces two completely different Symbol values

```
console.log(Symbol('yahoo') === Symbol('yahoo'))    // prints false
console.log(Symbol() === Symbol())   // prints false
```

the Symbol.for() function is completely different than the Symbol() function: Symbol() never returns the same value twice, but Symbol.for() always returns the same value when called with the same string

## global
ES2020 finally defines `globalThis` as the standard way to refer to the global object in any context. As of early 2020, this feature has been implemented by all modern browsers and by Node.

## conditional property access
```
let a = { b: {} };
a.b?.c?.d  // => undefined

let a = { b: null };
a.b?.c.d   // => undefined
```
## conditional functional call
```
function square(x, log) { // The second argument is an optional function
    log?.(x);             // Call the function if there is one
    return x * x;         // Return the square of the argument
}
```
Note, however, that ?.() only checks whether the lefthand side is null or undefined. It does not verify that the value is actually a function

```
o.m()     // Regular property access, regular invocation
o?.m()    // Conditional property access, regular invocation
o.m?.()   // Regular property access, conditional invocation
```

## for
`for/of` works with iterables. `for/in` works with object properties. 

## spread operator
The spread operator works on any iterable object.
```
let digits = [..."0123456789ABCDEF"];
digits // => ["0","1","2","3","4","5","6","7","8","9","A","B","C","D","E","F"]
```

## super
super can only be used in a derived class `constructor` or `static method` (not even in derived class instance method).

If you decline to define a constructor function, super() will be invoked and all arguments passed to the derived class constructor.


