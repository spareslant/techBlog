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