---
title: "Rust"
date: 2020-05-06T22:40:27+01:00
draft: true
---

## Rust concepts

* functions have types:
    - fn(u64, u64) -> u64 is function type
    - functions can also be stored in variables and passed to other functions.
    - The last expression is returned automatically (and must not be terminated with ;)
    - Functions are basically expressions that return a value, which is a () (Unit) type by default, akin to the void return type in C/C++
    - Functions can also be declared within other functions

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
   - We can even assign the value of if else blocks to a variable
