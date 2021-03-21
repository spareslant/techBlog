---
title: "Understanding Ruby"
date: 2021-03-06T21:55:33Z
draft: true
---

* class methods—which are, essentially, singleton methods attached to class objects
* a class variable is that it provides a storage mechanism that’s shared between a class and instances of that class, and that’s not visible to any other objects. Also class variables aren’t class-scoped variables. They’re class-hierarchy-scoped variables. It appears that instances can access class variables but not class methods because class methods are in class' singleton class
* subclasses (not the object created from subclass) get to call the class methods of their super-classes.
* if the receiver of the message is self, you can omit the receiver and the dot. Ruby will use self as the default receiver
* The rule for protected methods is as follows: you can call a protected method on an object x, as long as the default object (self) is an instance of the same class as x or of an ancestor or descendant class of x’s class.
* in the context of the top-level default object, main, which is an instance of Object brought into being automatically for the sole reason that something has to be self, even at the top level
* A method that you define at the top level is stored as a private instance method of the Object class
* First, Top level methods not only can but must be called in bareword style. Why? Because they’re private. You can only call them on self, and only without an explicit receiver (with the usual exemption of private setter methods, which must be called with self as the receiver).
* Second, private instance methods of Object can be called from anywhere in your code, because Object lies in the method-lookup path of every class
* Internally, Ruby uses symbols to keep track of all the names it’s created for variables, methods, and constants
* There is a top-level method called Array in addition to Array class.
* There is a top-level method called Hash in addition to Hash classh
* Any class that aspires to be enumerable must have an each method whose job is to yield items to a supplied code block, one at a time.
* By definition, an iterator is a method that yields one or more values to a block
* Classes that use Enumerable enter into a kind of contract: the class has to define an instance method called each, and in return, Enumerable endows the objects of the class with all sorts of collection-related behaviors. The methods behind these behaviors are defined in terms of each. In some respects, you might say the whole concept of a “collection” in Ruby is pegged to the Enumerable module and the methods it defines on top of each.
* `[1,2,3,4].inject(:+)` is equal to `[1, 2, 3, 4].inject(0) { |acc, n| acc + n}`
* you can use a symbol such as :upcase with an & in front of it in method-argument position, and the result will be the same as if you used a code block that called the method with the same name as the symbol on each element.
* `["google", "hotmail", "yahoo"].map {|name| name.upcase}` is equal to `["google", "hotmail", "yahoo"].map(&:upcase)`
* you call Enumerator.new with a code block, so that the code block contains the each logic you want the enumerator to follow; or you create an enumerator based on an existing enumerable object (an array, a hash, and so forth) in such a way that the enumerator’s each method draws its elements, for iteration, from a specific method of that enumerable object.
* This point also sheds light on the difference between an enumerator and an iterator. An enumerator is an object, and can therefore maintain state. It remembers where it is in the enumeration. An iterator is a method. When you call it, the call is atomic; the entire call happens, and then it’s over. Thanks to code blocks, there is of course a certain useful complexity to Ruby method calls: the method can call back to the block, and decisions can be made that affect the outcome. But it’s still a method. An iterator doesn’t have state. An enumerator is an enumerable object.
* an enumerator is an enumerable object whose each method operates as a kind of siphon, pulling values from an iterator defined on a different object.
* In addition to the three constants STDIN, STDOUT, STDERR, Ruby also gives you three global variables: $stdin, $stdout, and $stderr.
* The << object notation means the anonymous, singleton class of object.
```
str = "I am a string"
class << str
  def twice
    self + " " + self
  end
end
puts str.twice
```
* is there any difference between defining a method directly on an object (using the def obj.some_method notation) and adding a method to an object’s singleton class explicitly (by doing class << obj; def some_method)? The answer is that there’s one difference: constants are resolved differently.
* class methods also exhibit special behavior. Normally, when you define a singleton method on an object, no other object can serve as the receiver in a call to that method. (That’s what makes singleton methods singleton, or per-object.) Class methods are slightly different: a method defined as a singleton method of a class object can also be called on subclasses of that class
* In our example, the singleton class of C (where the method a_class_method lives) is considered the superclass of the singleton class of D.
```
class C
end
def C.a_class_method
  puts "Singleton method defined on C"
end
C.a_class_method

class D < C
end
D.a_class_method
```
* The main callable objects in Ruby are Proc objects, lambdas, and method objects
* Proc objects are self-contained code sequences that you can create, store, pass around as method arguments, and, when you wish, execute with the call method
*  When you provide a code block, you’re not sending the block to the method as an argument; you’re providing a code block, and that’s a thing unto itself
* it’s also possible for a proc to serve in place of the code block in a method call, using a similar special syntax:
```
p = Proc.new {|x| puts x.upcase }
%w{ David Black }.each(&p)
```
*  A method call can include both an argument list and a code block. Without a special flag like &, a Ruby method has no way of knowing that you want to stop binding parameters to regular arguments and instead perform a block-to-proc conversion and save the results.
```
def capture_block(&block)
  puts "Got block as proc"
  block.call
end
capture_block { puts "Inside the block" }

p = Proc.new { puts "This proc argument will serve as a code block." }
capture_block(&p)
```
* the & in capture_block(&p) does two things: it triggers a call to p’s to_proc method, and it tells Ruby that the resulting Proc object is serving as a code block stand-in
* The Proc object carries its context around with it.
* Procs differ from methods, with respect to argument handling, in that they don’t care whether they get the right number of arguments
* Like Proc.new, the lambda method returns a Proc object, using the provided code block as the function body:
* lambda-flavored procs don’t like being called with the wrong number of arguments.
* lambdas require explicit creation.
* lambdas differ from procs in how they treat the return keyword. return inside a lambda triggers an exit from the body of the lambda to the code context immediately containing the lambda. return inside a proc triggers a return from the method in which the proc is being executed.
* In addition to the lambda method, there’s a lambda literal constructor
* You can also unbind the method from its object and then bind it to another object, as long as that other object is of the same class as the original object (or a subclass):
* So far we’ve exclusively used the call method to call callable objects. You do, however, have a couple of other options.

One is the square-brackets method/operator, which is a synonym for call. You place any arguments inside the brackets:
```
mult = lambda {|x,y| x * y }
twelve = mult[3,4]
```
* You can also call callable objects using the () method:
```
twelve = mult.(3,4)
```
* instance_eval
```
class Person
  def initialize(&block)
    instance_eval(&block)
  end
  def name(name=nil)
    @name ||= name
  end
  def age(age=nil)
    @age ||= age
  end
end

joe = Person.new do
  name "Joe"
  age 37
end
```