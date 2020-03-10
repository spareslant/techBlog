---
title: "Understanding Golang"
date: 2020-02-29T22:13:36Z
draft: true
tags: ['golang']
---

* arrays are passed to functions as `passed by value`
* slices are passed as `references`
* assign a structure to an array of structures, the structure is copied into the array, so changing the value of the original structure will have no effect on the objects of the array
* rder in which you put the fields in the definition of a structure type is significant for the `type identity` of the defined structure. Put simply, two structures with the same fields will not be considered identical in Go if their fields are not in exactly the same order.
* it is perfectly legal for a Go function to return the memory address of a local variable.
* If you have a function call that you want to ensure is run, no matter what, you can use a defer statement. You can place the defer keyword before any ordinary function or method call, and Go will defer (that is, delay) making the function call until after the current function exits.
* strings in Go are value types not pointers, as is the case with C strings
* 