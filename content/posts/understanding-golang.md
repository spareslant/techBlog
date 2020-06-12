---
title: "Understanding Golang"
date: 2020-02-29T22:13:36Z
draft: true
tags: ['golang']
---

- arrays are passed to functions as `passed by value`
- slices are passed as `references`
- assign a structure to an array of structures, the structure is copied into the array, so changing the value of the original structure will have no effect on the objects of the array
- rder in which you put the fields in the definition of a structure type is significant for the `type identity` of the defined structure. Put simply, two structures with the same fields will not be considered identical in Go if their fields are not in exactly the same order.
- it is perfectly legal for a Go function to return the memory address of a local variable.
- If you have a function call that you want to ensure is run, no matter what, you can use a defer statement. You can place the defer keyword before any ordinary function or method call, and Go will defer (that is, delay) making the function call until after the current function exits.
- strings in Go are value types not pointers, as is the case with C strings
- in `import` statement like `import ("some/other")`, `other` is the name of directory. Each go src file in this directory has a line like `package dosomething`. All the files inside `"some/other"` directory must have same package declaration, in this case `package dosomething`. Now this `dosomething` will be available to be used in the src file which is importing `"some/other"`. Therefore to make things easier to read you can write import statement like this `import ( dosomething "some/other") `. Normally package declaration name concides with package directory name.
- If an array’s element type is comparable then the array type is comparable too, so we may directly compare two arrays of that type using the == operator, which reports whether all corresponding elements are equal. The != operator is its negation.
- Strings can be converted to byte slices and back again:
```
s := "abc"
b := []byte(s)
s2 := string(b)
```
- The only legal slice comparison is against nil, as in `if summer == nil`
- The nil slice has length and capacity zero, but there are also non-nil slices of length and capacity zero, such as `[]int{}`
- the nil value of a particular slice type can be written using a conversion expression such as `[]int(nil)`
- to create a slice: `make([]T, len, cap)` // same as `make([]T, cap)[:len]`
- new empty map is `map[string]int{}`
- a map lookup using a key that isn’t present returns the zero value for its type. e.g 
```
x := make(map[string]int)
value of x["bob"] is 0 even though "bob" key was never defined.
```
- The order of map iteration is unspecified (in range)
- existence of a key in map is checked by following expression
```
age, ok := ages["bob"]
or
if age, ok := ages["bob"]; !ok  
``` 
- As with slices, maps cannot be compared to each other; the only legal comparison is with nil
- The main difference between new and make is that variables created with make are properly initialized without just zeroing the allocated memory space. Additionally, make can only be applied to maps, channels, and slices, and it does not return a memory address, which means that make does not return a pointer.
- A rune is an int32 value, and therefore it is a Go type that is used for representing a Unicode code point.
- type assertion example `v, isValid := str.(int)`
- `switch` statement can be used for type assertions as well.
- If all of a struct's fields are comparable types, then the struct as a whole is also comparable
- anonymous struct are possible
```
point2 := struct {
  x int
  y int
}{}
point2.x = 10
point2.y = 5
```
- Methods with a Pointer Receiver
```
func (p *Point) ScaleBy(factor float64) {
  p.X *= factor
  p.Y *= factor
}
```
The name of this method is `(*Point).ScaleBy`. The parentheses are necessary; without them, the expression would be parsed as `*(Point.ScaleBy)`. i.e `*Point.ScaleBy == *(Point.ScaleBy)`
- GOPATH is nothing but the current appointed workspace on your machine. It is an environment variable that tells the Go compiler about where your source code, binaries, and packages are placed. 


