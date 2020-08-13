---
title: "Understanding Javascript"
date: 2019-07-28T14:27:07+01:00
draft: true
tags: ["javascript"]
---

* using the var operator to define a variable makes it local to the scope in which it was defined.
* It is, however, possible to define a variable globally by simply omitting the var operator.
* `let` is block scoped, but `var` is function scoped.
* declaring variables using `let` in the global context, variables will not attach to the `window` object as they do with `var`
* Following are the primitive data types:
  * Null
  * Undefined
  * Boolean
  * Number
  * String
* Following is the refernece type data
  * Object
* `typeof` is an operator and may return one of the following strings:
  * "undefined"
  * "boolean"
  * "string"
  * "number"
  * "object"
  * "function"
* `"null"` is considered to be an empty `object`.
* when a variable is declared with `var` but un-initialized, it is assigned the value of `undefined`.
* `undefined` is derivative of `null`. `undefined == null` returns `true`.
* `null` is used in conjunction with object.
* never explicitly set the value of a variable to undefined, but the same does not hold true for null. Any time an object is expected but is not available, null should be used in its place.
* When defining a variable that is meant to later hold an object, it is advisable to initialize the variable to null as opposed to anything else.
* boolean values are `true` and `false`.
* All types of values have boolean equivalent. To convert a value to its boolean equivalent, `Boolean()` function can be used.
* Flow control statements like `if` automatically performs boolean conversion.
* `NaN == NaN` returns `false`. `isNaN()` function exists.
* Number conversion functions
  * `Number()`
  * `parseInt()`
  * `parseFloat()`
* strings are sequence of 16-bit Unicode characters.
* single quoted `'` and double quoted `"` strings are same.
* Unicode chars in string can be written as `\unnnn`
* Strings are immutable.
* string conversions.
  * almost every type of value has `toString()` method.
  * `String()` casting function.
* Object Type 
  * An object can be created using following syntax
    * `var o = new Object()`
      * `Object()` is a constructor function in above call.
      * `Object` is also the base of all objects.
    * Each `Object` instance has following properties and methods
      * `constructor`
      * `hasOwnProperty("propertyName")`
      * `isPrototypeOf(object)`
      * `propertyIsEnumerable("propertyName")`
      * `toLocaleString()`
      * `toString()`
      * `valueOf()`
* `with` statement sets the scope of the code within a particular object
* paramters passed in function can be accessed via `arguments` object.
* All arguments in ECMAScript are passed by value. It is not possible to pass arguments by reference.
* functions cannot be overloaded.
* use `typeof` with primitive types and `instanceof` with reference types.
* All reference values, by definition, are instances of `Object`, so the `instanceof` operator always returns `true` when used with a reference value and the `Object constructor`
* if `instanceof` is used with a `primitive value`, it will always return `false`, because primitives aren’t objects.
* The `typeof` operator also returns `function` when used on a function.
* The `for-in` statement is a strict iterative statement. It is used to enumerate the non-symbol keyed properties of an `object`.
* The `for-of` statement is a strict iterative statement. It is used to loop through elements in an iterable `object:w`
* Built-in reference types
  * `Object`
  * `Array`
    * `push()`
    * `pop()`
    * `Array.isArray(value)`
    * Arrays can contain different `types` of data in their cells.
    * `length` property of Array is NOT read-only.
    * `shift()`
    * `unshift()`
    * `reverse()`
    * `sort()`
    * `concat()`
    * `slice()`
    * `splice()`
    * `indexOf`
    * `lastIndexOf()`
    * `every()`
    * `filter()`
    * `forEach()`
    * `map()`
    * `some()`
    * `reduce()`
    * `reduceRight()`
  * `Date`
    * `parse()`
    * `UTC()`
    * `now()`
    * `Regexp`
      * `exec()`
  * `Function`
    * Each function is an instance of `function` type.
    * function names are simply pointers to functions.
    * Function declarations are read and available in an execution context before any code is executed, whereas function expressions aren’t complete until the execution reaches that line of code.
    * function declarations are read and added to the execution context before, the code begins running, through a process called function declaration hoisting.
    * Two special objects exist inside a function: `arguments` and `this`
    * the `arguments` object also has a property named `callee`, which is a pointer to the function that owns the arguments object.
    * `this` is a reference to the context object that the function is operating on.
    * when a function is called in the global scope of a web page, the this object points to `window`
    * `caller` is another property which contains a reference to the function that called this function or null if the function was called from the global scope
    * For looser coupling, you can also access the same information via `arguments.callee.caller`
    * As functions are objects, Each function has two properties: `length` and `prototype`.
    * the prototype property is not enumerable and so will not be found using `for-in`.
    * There are two additional `methods` for functions: `apply()` and `call()`
    * The `bind()` method creates a new function instance whose this value is bound to the value that was passed into `bind()`
    * For functions, the inherited methods `toLocaleString()` and `toString()` always return the function’s code
* Every time a primitive value is read, an object of the corresponding primitive wrapper type is created behind the scenes, allowing access to any number of methods for manipulating the data.
* Three special reference types are designed to ease interaction with primitive values: the `Boolean type`, the `Number type`, and the `String type`
* It is possible to create the primitive wrapper objects explicitly using the Boolean, Number, and String constructors. This should be done only when absolutely necessary
* Calling typeof on an instance of a primitive wrapper type returns "object"
* There are two other singleton built-in objects defined by ECMA-262: `Global` and `Math`.
* Functions covered earlier in this book, such as isNaN(), isFinite(), parseInt(), and parseFloat() are actually methods of the Global object
* `Object` consists of `properties` and `methods`.
  * `data` properties
    * has following 4 `attributes`
      * `[[Configurable]]`
      * `[[Enumerable]]`
        * property will be returned in `for-in` loop.
      * `[[Writable]]`
      * `[[Value]]`
  * `accessor` properties
    * has following 4 `attributes`
      * `[[Configurable]]`
      * `[[Enumerable]]`
      * `[[Get]]`
      * `[[Set]]`
* Any function that is called with the new operator acts as a constructor, whereas any function called without it acts just as you would expect a normal function call to act
* The constructor property mentioned earlier exists only on the prototype and so is accessible from object instances.
* Interface inheritance is not possible in ECMAScript, because, as mentioned previously, functions do not have signatures
* Implementation inheritance is the only type of inheritance supported by ECMAScript, and this is done primarily through the use of prototype chaining.
* you cannot pass arguments into the supertype constructor when the subtype instance is being created
* You can create `named` function expressions.

```javascript
var factorial = (function f(num){
    if (num <= 1){
        return 1;
    } else {
        return num * f(num-1);
    }
});
```
* The window object serves a dual purpose in browsers, acting as the JavaScript interface to the browser window and the ECMAScript Global object. 
* a global variable and defining a property directly on window: global variables cannot be removed using the delete operator, while properties defined directly on window can.
* attempting to access an undeclared variable throws an error, but it is possible to check for the existence of a potentially undeclared variable by looking on the window object.
* If a page contains frames, each frame has its own window object and is stored in the frames collection.
* Each window object has a name property containing the name of the frame












