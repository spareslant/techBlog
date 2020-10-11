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