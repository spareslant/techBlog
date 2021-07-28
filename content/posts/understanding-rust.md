---
title: "Rust"
date: 2020-05-06T22:40:27+01:00
draft: true
---

## Rust concepts

* functions have types:
  - fn(u64, u64) -> u64 is function type
  - In function signatures, you must declare the type of each parameter
  - functions can also be stored in variables and passed to other functions.
  - The last expression is returned automatically (and must not be terminated with ;)
  - Functions are basically expressions that return a value, which is a () (Unit) type by default, akin to the void return type in C/C++
  - Functions can also be declared within other functions
  - Rust doesn’t care where you define your functions, only that they’re defined somewhere.
  - Statements are instructions that perform some action and do not return a value.
  - Expressions evaluate to a resulting value. Hence expressions returns a value.
  - Rust is an expression-based language
  - let y = 6; is a statement
  - Function definitions are also statements
  - C and Ruby, where the assignment returns the value of the assignment. In those languages, you can write x = y = 6 and have both x and y have the value 6; that is not the case in Rust.
  - Calling a function is an expression
  - Calling a macro is an expression
  - The block that we use to create new scopes, {}, is an expression, hence {} block returns a value. i.e if, loop, while all returns a value
    ```rust
    fn main() {
      let x = 5;
      let y = {
        let x = 3;
        x + 1
      };
    }
    Note: Note the x + 1 line without a semicolon at the end, which is unlike most of the lines you’ve seen so far. Expressions do not include ending semicolons
    ```
  - It’s possible to return multiple values using a tuple.
	- Rust doesn't care where in the file you define your functions, as long as they're defined somewhere in the file.
  - reading (&self), mutating (&mut self), or consuming (self).
  - the :: syntax is used for both associated functions and namespaces created by modules
	- One valid return type for function `main` is (), and conveniently, another valid return type is Result<T, E>



* Strings
    - two forms of string: &str and String
    - strings are not null terminated.
    - strings of type `String` are allocated on heap
    - &str types are usually pointers to an existing string, which could either be on stack, the heap, or a string in the data segment of the compiled object code



* if-then-else
   - if construct is not a statement. Its an expression
   - expression returns a value, therefore if construct returns a value as well.
   - value may be an empty value called `empty unit` `empty () unit type` or may be an actual value.
   - Whatever remains in the last line inside the braces becomes the return value of the if else expression
   - It is important to note that both if and else branches should have the same return type
   - we don't need parentheses around the if condition expression
   - if is an expression, we can use it on the right side of a let statement. We can even assign the value of if else blocks to a variable
   ```rust
   let number = if condition { 5 } else { 6 };
   ```


* constants
  - You declare constants using the const keyword instead of the let keyword, and the type of the value must be annotated.
  - can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.
  - constants may be set only to a constant expression, not the result of a function call or any other value that could only be computed at runtime
  - Rust’s naming convention for constants is to use all uppercase with underscores between words



* variable shadowing
  - shodowing is done by re-declaring the same variable using `let` key.
  ```
  let x = 5;
  let x = "yahoo";
  ```
  - shadowing also allows to change the variable type



* scalar types
  - integers
  - floating points
  - numbers
  - characters => char type is four bytes in size and represents a Unicode Scalar Value



* compound types
  - Tuple
    - Tuples have a fixed length: once declared, they cannot grow or shrink in size
    - we can access a tuple element directly by using a period (.) followed by the index of the value we want to access
    - destructuring is also possible
      ```rust
      let tup = (500, 6.4, "yahoo");
      let (x, y, z) = tup;
      ```
    - After a tuple is declared, it can't grow or shrink in size. Elements can't be added or removed. The data type of a tuple is defined by the sequence of the data types of the elements.
  - Array
    - All elements must be of same type.
    - Arrays are useful when you want your data allocated on the stack rather than the heap.
    - always have a fixed number of elements
    ```rust
    let a: [i32; 5] = [1, 2, 3, 4, 5];
    let a = [3; 5];
    let a = [3, 3, 3, 3, 3];
    ```


* Generate local documentation
  - `cargo doc --open`


* Stack and Heap
  - All data stored on the stack must have a known, fixed size. Data with an unknown size at compile time or a size that might change must be stored on the heap instead.
  - For heap, memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. Because the pointer is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer.
  - Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; that location is always at the top of the stack.
  - Comparatively, allocating space on the heap requires more work, because the allocator must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.
  - Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there.
  - Keeping track of what parts of code are using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap so you don’t run out of space are all problems that ownership addresses



* Ownerships
  - Rust won’t let us annotate a type with the Copy trait if the type, or any of its parts, has implemented the Drop trait.
  - you can’t have mutable and immutable references in the same scope
	- https://blog.thoughtram.io/ownership-in-rust/
	- In Rust, only one thing can ever own a piece of data at a time.
	- Ownership can't be transferred for types that implement the Copy trait, such as for simple values like numbers.
	- & borrows known as "immutable borrows". &mut borrows known as mutable borrows



* References
  - Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used.
	- https://blog.thoughtram.io/references-in-rust/



* Pointers
  - References are indicated by the & symbol and borrow the value they point to. They don’t have any special capabilities other than referring to data.
  - reference counting smart pointer:  This pointer enables you to have multiple owners of data by keeping track of the number of owners and, when no owners remain, cleaning up the data.
  - an additional difference between references and smart pointers is that references are pointers that only borrow data; in contrast, in many cases, smart pointers `own` the data they point to.
  - String and Vec<T> are smart pointers
  - Smart pointers are usually implemented using structs. The characteristic that distinguishes a smart pointer from an ordinary struct is that smart pointers implement the Deref and Drop traits.
  - The Deref trait allows an instance of the smart pointer struct to behave like a reference so you can write code that works with either references or smart pointers.
  - The Drop trait allows you to customize the code that is run when an instance of the smart pointer goes out of scope
  - Deref coercion works only on types that implement the Deref trait



* Structs
  - Rust supports three struct types: classic structs, tuple structs, and unit structs (source: https://docs.microsoft.com/en-us/learn/modules/rust-understand-common-concepts/4-structs-enums)
    ```
    // Classic struct with named fields
    struct Student { name: String, level: u8, pass: bool  }

    // Tuple struct with data types only
    struct Grades(char, char, char, char, f32);

    // Unit struct
    struct Unit;
    ```
  - Both tuple and struct use dot notation to access their elements
  - Rust doesn’t allow us to mark only certain fields as mutable
  - As with any expression, we can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance
  - the name of a struct type is capitalized
  - Struct types are often defined outside of the main function and other functions in the Rust program
  - A classic struct definition doesn't end with a semicolon.
  - Like a function, the body of a classic struct is defined inside curly brackets {}. `struct Student { name: String, level: u8, pass: bool }`
  - the body of a tuple struct is defined inside parentheses (), like a tuple.
  - (source: https://docs.microsoft.com/en-us/learn/modules/rust-understand-common-concepts/4-structs-enums)



* Enums
  - Enums are types that can be any one of several variants.
  - Like structs, enum variants can have named fields, fields without names, or no fields at all. Like struct types, enum types are also capitalized.
  ```
	enum WebEvent {
    // An enum variant can be like a unit struct without fields or data types
    WELoad,
    // An enum variant can be like a tuple struct with data types but no named fields
    WEKeys(String, char),
    // An enum variant can be like a classic struct with named fields and their data types
    WEClick { x: i64, y: i64 }
	}
  ```
	- We define an enum with variants similar to how we define different kinds of struct types
	- All the variants are grouped together in the same `WebEvent` enum type. Each variant in the enum isn't its own type
	- Any function that uses a variant of the WebEvent enum must accept all the variants in the enum
	- define an enum with structs
	```
	// Define a tuple struct
	struct KeyPress(String, char);

	// Define a classic struct
	struct MouseClick { x: i64, y: i64 }

	// Redefine the enum variants to use the data from the new structs
	// Update the page Load variant to have the boolean type
	enum WebEvent { WELoad(bool), WEClick(MouseClick), WEKeys(KeyPress) }
	```
	- To access the specific variant in the enum definition, we use the syntax <enum>::<variant> with double colons ::
  - The Option<T> enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. in addition, so are its variants: you can use Some and None directly without the Option:: prefix.
  - methods can be defined on enums also.
  - Options Enum
    ```rust
    enum Option<T> {
      Some(T_type_value),
      None
    }
    ```


* if-let
  - https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#conditional-if-let-expressions
  - https://doc.rust-lang.org/book/ch18-02-refutability.html
  - https://nickymeuleman.netlify.app/garden/rust-if-let-while-let



* match
  - The code associated with each arm is an expression, and the resulting value of the expression in the matching arm is the value that gets returned for the entire match expression.



* modules
  - The way privacy works in Rust is that all items (functions, methods, structs, enums, modules, and constants) are private by default.
  - items in a parent module can’t use/see the private items inside child modules, but items in child modules can use the items in their ancestor modules
  - If we use pub before a struct definition, we make the struct public, but the struct’s fields will still be private
  - if we make an enum public, all of its variants are then public. We only need the pub before the enum keyword
  - Paths brought into scope with use also check privacy
  - with `use` we bring the function's parent module in scope (not function itself) so that we call function like `mymod::myfunc()`
  - However with `structs, enums` and other items, we specify full path with `use`.
  - standard library is shipped with the Rust language, we don’t need to change Cargo.toml to include std. But we do need to refer to it with use to bring items from there into our package’s scope
  - Using a semicolon after mod front_of_house rather than using a block tells Rust to load the contents of the module from another file with the same name as the module
  - The mod keyword declares modules, and Rust looks in a file with the same name as the module for the code that goes into that module.
	- When we bring a name into scope with the use keyword, the name available in the new scope is private
	- `mod front_of_house;` Using a semicolon after mod front_of_house rather than using a block tells Rust to load the contents of the module from another file with the same name as the module
  - src/main.rs and src/lib.rs are called crate roots. The reason for their name is that the contents of either of these two files form a module named crate at the root of the crate’s module structure, known as the module tree


* Vectors
  - the two ways to get the third element are by using & and [], which gives us a reference, or by using the get method with the index passed as an argument, which gives us an Option<&T>
  ```rust
  let v = vec![1, 2, 3, 4, 5];
  let third: &i32 = &v[2]; // first method
  match v.get(2) {
    Some(third) => println!("Third element is {}", third),
    None => println!("There is no third element"),
  }
  ```
  - Vectors store data in Heap
	- vec::get(index) returns Option<&T>. So in case of `vec!["yahoo", "google", "hotmail"]` it will return Option<&&str> because "yahoo" etc are actually `&str`.



* Strings
  - Rust has only one string type in the `core language`, which is the string slice str that is usually seen in its borrowed form `&str`
  - The `String` type, which is provided by `Rust’s standard library` rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type
  - both `String` and `string slices (&str)` are UTF-8 encoded.
  - "ALiteralDoubleQuotedString" is actually a string slice. &str as a pointer to an immutable string data. String literals are all of type &str
  - compiler can coerce the `&String` argument into a `&str`
  - Rust strings don’t support indexing. 
  - The char type in Rust contains unicode code points, but they don't use utf-8 encoding. A char in Rust is a 21-bit integer that's padded to be 32 bits wide. The char contains the plain code point value directly. [source: https://docs.microsoft.com/en-us/learn/modules/rust-understand-common-concepts/3-data-types]
  - `String` type is allocated on heap.
	- https://blog.thoughtram.io/string-vs-str-in-rust/
  - The type that signifies “string slice” is written as &str
  - &str is an immutable reference
  - string literals *are* string slices already
  ```rust
  let a = [1, 2, 3, 4, 5];
  let a_slice = &a[1..3];
  type of a_slice is &[i32]
  ```
	- `str` is an immutable sequence of UTF-8 bytes of dynamic length somewhere in memory. Since the size is unknown, one can only handle it behind a pointer. This means that str most commonly appears as `&str`, a reference to some UTF-8 data, normally called a "string slice" or just a "slice"
	- https://stackoverflow.com/questions/24158114/what-are-the-differences-between-rusts-string-and-str#:~:text=A%20Rust%20String%20is%20like,contents%20of%20std%3A%3Astring%20.
	- https://stackoverflow.com/questions/41413336/do-all-primitive-types-implement-the-copy-trait


* Hash Maps
  - Just like vectors, hash maps store their data on the heap
  - all of the keys must have the same type, and all of the values must have the same type.
	- access to the hash map items are checked at run time.



* Errors
  - A backtrace is a list of all the functions that have been called to get to this point.
  - the key to reading the backtrace is to start from the top and read until you see files you wrote
  ```
  enum Result<T, E> {
    Ok(T),
    Err(E),
  }
  ```


* Loops
	- Rust offers three loop expressions to make a program repeat a block of code: loop, while, for
	- By using the break keyword, you can both stop repeating the actions in the expression body and also return a value at the break point.
	- The loop expression body can have more than one break point. When the expression has multiple break points, every break point must return a value of the same type
	- When a break point doesn't explicitly return a value, the program interprets the expression result as an empty tuple, ()
	- The loop expression creates an infinite loop
	- The while loop uses a conditional expression
	- The for loop uses an iterator to process a collection of items



* Lifetimes
	- https://blog.thoughtram.io/lifetimes-in-rust/
	- As with types, lifetime durations are inferred by the Rust compiler.
	- The reference's lifetime that the function returns matches the smaller of the references' lifetimes that are passed in.



* Generics and Traits
	- A trait is a common interface that a group of types can implement
	- Traits can be defined on structs and enums both.
	- the Debug and PartialEq traits can be automatically implemented for us by the Rust compiler by using the #[derive(Trait)] attribute, if each of its fields implements the trait
	- some generic parameters are declared with impl and some are declared with the method definition (source: https://doc.rust-lang.org/book/ch10-01-syntax.html)
  - trait implementations is that we can implement a trait on a type only if either the trait or the type is local to our crate.
  - it isn’t possible to call the default implementation from an overriding implementation of that same method.
  - Default implementations can call other methods in the same trait, even if those other methods don’t have a default implementation



* Tests
  - Tests fail when something in the test function panics
  - Each test is run in a new thread, and when the main thread sees that a test thread has died, the test is marked as failed.




* Iterators
	- all iterators implement a trait named Iterator that's defined in the standard library and is used to implement iterators over collections such as ranges, arrays, vectors, and hash maps
  - The iter method produces an iterator over immutable references
  - In Rust, iterators are lazy, meaning they have no effect until you call methods that consume the iterator to use it up
  - Methods that call next are called `consuming adaptors`, because calling them uses up the iterator.
  - Other methods defined on the Iterator trait, known as `iterator adaptors`, allow you to change iterators into different kinds of iterators
  - The filter method on an iterator takes a closure that takes each item from the iterator and returns a Boolean. If the closure returns true, the value will be included in the iterator produced by filter. If the closure returns false, the value won’t be included in the resulting iterator



* Closures
  - Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions.
  - Unlike functions, closures can capture values from the scope in which they’re defined
  - Closures don’t require you to annotate the types of the parameters or the return value like fn functions do
  - Each closure instance has its own unique anonymous type: that is, even if two closures have the same signature, their types are still considered different
  - To define structs, enums, or function parameters that use closures, we use generics and trait bounds
