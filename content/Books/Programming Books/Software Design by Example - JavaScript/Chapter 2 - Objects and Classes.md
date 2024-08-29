---
id: 01J6CQY19Z145V6BN002CDD2G1
title: Chapter 2 - Objects and Classes
modified: 2024-08-28T11:26:42-04:00
description: Learn about Objects and classes
tags:
  - python
  - books
  - software-design
  - programming
---
# Chapter 2: Objects and Classes

- Objects are useful without classes, but classes make them easier to understand.
- A well-designed class defines a contract that code using its instances can rely on.
- Objects that respect the same contract are polymorphic, i.e., they can be used interchangeably even if they do different specific things.
- Objects and classes can be thought of as dictionaries with stereotyped behavior.
- Most languages allow functions and methods to take a variable number of arguments.
- Inheritance can be implemented in several ways that differ in the order in which objects and classes are searched for methods.

Terms defined: [alias](https://third-bit.com/sdxpy/glossary/#gl:alias), [argument](https://third-bit.com/sdxpy/glossary/#gl:argument), [cache](https://third-bit.com/sdxpy/glossary/#gl:cache), [class method](https://third-bit.com/sdxpy/glossary/#gl:class_method), [constructor](https://third-bit.com/sdxpy/glossary/#gl:constructor), [derived class](https://third-bit.com/sdxpy/glossary/#gl:derived_class), [design by contract](https://third-bit.com/sdxpy/glossary/#gl:design_by_contract), [monkey patching](https://third-bit.com/sdxpy/glossary/#gl:monkey_patching), [multiple inheritance](https://third-bit.com/sdxpy/glossary/#gl:multiple_inheritance), [object-oriented programming](https://third-bit.com/sdxpy/glossary/#gl:oop), [parameter](https://third-bit.com/sdxpy/glossary/#gl:parameter), [polymorphism](https://third-bit.com/sdxpy/glossary/#gl:polymorphism), [recursion](https://third-bit.com/sdxpy/glossary/#gl:recursion), [spread](https://third-bit.com/sdxpy/glossary/#gl:spread), [static method](https://third-bit.com/sdxpy/glossary/#gl:static_method), [upcall](https://third-bit.com/sdxpy/glossary/#gl:upcall), [varargs](https://third-bit.com/sdxpy/glossary/#gl:varargs)

1. [Objects](https://third-bit.com/sdxpy/oop/#oop-objects)
2. [Classes](https://third-bit.com/sdxpy/oop/#oop-classes)
3. [Arguments](https://third-bit.com/sdxpy/oop/#oop-args)
4. [Inheritance](https://third-bit.com/sdxpy/oop/#oop-inheritance)
5. [Summary](https://third-bit.com/sdxpy/oop/#oop-summary)
6. [Exercises](https://third-bit.com/sdxpy/oop/#oop-exercises)
7. [slides](https://third-bit.com/sdxpy/oop/slides/)

We are going to create a lot of objects and classes in these lessons, and they will be a lot easier to use if we understand how they are implemented. Historically, [object-oriented programming](https://third-bit.com/sdxpy/glossary/#gl:oop "A style of programming in which functions and data are bound together in objects that only interact with each other through well-defined interfaces.") (OOP) was invented to solve two problems:

1. What is a natural way to represent real-world “things” in code?
    
2. How can we organize code to make it easier to understand, test, and extend?

[

## Section 2.1: Objects

](https://github.com/gvwilson/sdxpy/issues/new?title=/oop%20-%20Section%202.1:%20Objects%20(2024-04-28))

As a motivating problem, let’s define some of the things a generic shape in a drawing package must be able to do:
```
class Shape:
    def __init__(self, name):
        self.name = name

    def perimeter(self):
        raise NotImplementedError("perimeter")

    def area(self):
        raise NotImplementedError("area")
```

[shapes_original.py](https://third-bit.com/sdxpy/oop/shapes_original.py)

A specification like this is sometimes called a [contract](https://third-bit.com/sdxpy/glossary/#gl:design_by_contract "A style of designing software in which functions specify the pre-conditions that must be true in order for them to run and the post-conditions they guarantee will be true when they return. A function can then be replaced by one with weaker pre-conditions (i.e., it accepts a wider set of input) and/or stronger post-conditions (i.e., it produces a smaller range of output) without breaking anything else.") because an object must satisfy it in order to be considered a shape, i.e., must provide methods with these names that do what those names suggest. For example, we can [derive](https://third-bit.com/sdxpy/glossary/#gl:derived_class "In object-oriented programming, a class that is a direct or indirect extension of a base class.") classes from `Shape` to represent squares and circles.

`class Square(Shape):     def __init__(self, name, side):         super().__init__(name)         self.side = side      def perimeter(self):         return 4 * self.side      def area(self):         return self.side ** 2  class Circle(Shape):     def __init__(self, name, radius):         super().__init__(name)         self.radius = radius      def perimeter(self):         return 2 * math.pi * self.radius      def area(self):         return math.pi * self.radius ** 2`

[shapes_original.py](https://third-bit.com/sdxpy/oop/shapes_original.py)

Since squares and circles have the same methods, we can use them interchangeably. This is called [polymorphism](https://third-bit.com/sdxpy/glossary/#gl:polymorphism "Having many different implementations of the same interface. If a set of functions or objects are polymorphic, they can be called interchangeably."), and it reduces cognitive load by allowing the people using related things to ignore their differences:

`examples = [Square("sq", 3), Circle("ci", 2)] for thing in examples:     n = thing.name     p = thing.perimeter()     a = thing.area()     print(f"{n} has perimeter {p:.2f} and area {a:.2f}")`

[shapes_original.py](https://third-bit.com/sdxpy/oop/shapes_original.py)

`sq has perimeter 12.00 and area 9.00 ci has perimeter 12.57 and area 12.57`

[shapes_original.out](https://third-bit.com/sdxpy/oop/shapes_original.out)

But how does polymorphism work? The first thing we need to understand is that a function is an object. While the bytes in a string represent characters and the bytes in an image represent pixels, the bytes in a function are instructions ([Figure 2.1](https://third-bit.com/sdxpy/oop/#oop-func-obj)). When Python executes the code below, it creates an object in memory that contains the instructions to print a string and assigns that object to the variable `example`:

`def example():     print("in example")`

[func_obj.py](https://third-bit.com/sdxpy/oop/func_obj.py)

![Bytes as characters, pixels, or instructions](https://third-bit.com/sdxpy/oop/func_obj.svg)

Figure 2.1: Bytes can be interpreted as text, images, instructions, and more.

We can create an [alias](https://third-bit.com/sdxpy/glossary/#gl:alias "A second or subsequent reference to the same object. Aliases are useful, but increase the cognitive load on readers who have to remember that all these names refer to the same thing.") for the function by assigning it to another variable and then call the function by referencing that second variable. Doing this doesn’t alter or erase the connection between the function and the original name:

`alias = example alias()`

[func_obj.py](https://third-bit.com/sdxpy/oop/func_obj.py)

`in example`

[func_obj.out](https://third-bit.com/sdxpy/oop/func_obj.out)

We can also store function objects in data structures like lists and dictionaries. Let’s write some functions that do the same things as the methods in our original Python and store them in a dictionary to represent a square ([Figure 2.2](https://third-bit.com/sdxpy/oop/#oop-shapes-dict)):

`def square_perimeter(thing):     return 4 * thing["side"]  def square_area(thing):     return thing["side"] ** 2  def square_new(name, side):     return {         "name": name,         "side": side,         "perimeter": square_perimeter,         "area": square_area     }`

[shapes_dict.py](https://third-bit.com/sdxpy/oop/shapes_dict.py)

![Storing shapes as dictionaries](https://third-bit.com/sdxpy/oop/shapes_dict.svg)

Figure 2.2: Using dictionaries to emulate objects.

If we want to use one of the “methods” in this dictionary, we call it like this:

`def call(thing, method_name):     return thing[method_name](thing)  examples = [square_new("sq", 3), circle_new("ci", 2)] for ex in examples:     n = ex["name"]     p = call(ex, "perimeter")     a = call(ex, "area")     print(f"{n} {p:.2f} {a:.2f}")`

[shapes_dict.py](https://third-bit.com/sdxpy/oop/shapes_dict.py)

The function `call` looks up the function stored in the dictionary, then calls that function with the dictionary as its first object; in other words, instead of using `obj.meth(arg)` we use `obj["meth"](obj, arg)`. Behind the scenes, this is (almost) how objects actually work. We can think of an object as a special kind of dictionary. A method is just a function that takes an object of the right kind as its first parameter (typically called `self` in Python).

[

## Section 2.2: Classes

](https://github.com/gvwilson/sdxpy/issues/new?title=/oop%20-%20Section%202.2:%20Classes%20(2024-04-28))

One problem with implementing objects as dictionaries is that it allows every single object to behave slightly differently. In practice, we want objects to store different values (e.g., different squares to have different sizes) but the same behaviors (e.g., all squares should have the same methods). We can implement this by storing the methods in a dictionary called `Square` that corresponds to a class and having each individual square contain a reference to that higher-level dictionary ([Figure 2.3](https://third-bit.com/sdxpy/oop/#oop-shapes-class)). In the code below, that special reference uses the key `"_class"`:

`def square_perimeter(thing):     return 4 * thing["side"]  def square_area(thing):     return thing["side"] ** 2  Square = {     "perimeter": square_perimeter,     "area": square_area,     "_classname": "Square" }  def square_new(name, side):     return {         "name": name,         "side": side,         "_class": Square     }`

[shapes_class.py](https://third-bit.com/sdxpy/oop/shapes_class.py)

![Separating properties from methods](https://third-bit.com/sdxpy/oop/shapes_class.svg)

Figure 2.3: Using dictionaries to emulate classes.

Calling a method now involves one more lookup because we have to go from the object to the class to the method, but once again we call the “method” with the object as the first argument:

`def call(thing, method_name):     return thing["_class"][method_name](thing)  examples = [square_new("sq", 3), circle_new("ci", 2)] for ex in examples:     n = ex["name"]     p = call(ex, "perimeter")     a = call(ex, "area")     c = ex["_class"]["_classname"]     print(f"{n} is a {c}: {p:.2f} {a:.2f}")`

[shapes_class.py](https://third-bit.com/sdxpy/oop/shapes_class.py)

As a bonus, we can now reliably identify objects’ classes and ask whether two objects are of the same class or not by checking what their `"_class"` keys refer to.

### Arguments vs. Parameters

Many programmers use the words [argument](https://third-bit.com/sdxpy/glossary/#gl:argument "A value passed into a function or method call.") and [parameter](https://third-bit.com/sdxpy/glossary/#gl:parameter "The name that a function gives to one of the values passed to it when it is called.") interchangeably, but to make our meaning clear, we call the values passed into a function its arguments and the names the function uses to refer to them as its parameters. Put another way, parameters are part of the definition, and arguments are given when the function is called.

[

## Section 2.3: Arguments

](https://github.com/gvwilson/sdxpy/issues/new?title=/oop%20-%20Section%202.3:%20Arguments%20(2024-04-28))

The methods we have defined so far operate on the values stored in the object’s dictionary, but none of them take any extra arguments as input. Implementing this is a little bit tricky because different methods might need different numbers of arguments. We could define functions `call_0`, `call_1`, `call_2` and so on to handle each case, but like most modern languages, Python gives us a better way. If we define a parameter in a function with a leading `*`, it captures any “extra” values passed to the function that don’t line up with named parameters. Similarly, if we define a parameter with two leading stars `**`, it captures any extra named parameters:

`def show_args(title, *args, **kwargs):     print(f"{title} args '{args}' and kwargs '{kwargs}'")  show_args("nothing") show_args("one unnamed argument", 1) show_args("one named argument", second="2") show_args("one of each", 3, fourth="4")`

[varargs.py](https://third-bit.com/sdxpy/oop/varargs.py)

`nothing args '()' and kwargs '{}' one unnamed argument args '(1,)' and kwargs '{}' one named argument args '()' and kwargs '{'second': '2'}' one of each args '(3,)' and kwargs '{'fourth': '4'}'`

[varargs.out](https://third-bit.com/sdxpy/oop/varargs.out)

This mechanism is sometimes referred to as [varargs](https://third-bit.com/sdxpy/glossary/#gl:varargs "Short for "variable arguments", a mechanism that captures any "extra" arguments to a function or method.") (short for “variable arguments”). A complementary mechanism called [spreading](https://third-bit.com/sdxpy/glossary/#gl:spread "To automatically match the values from a list or dictionary supplied by the caller to the parameters of a function.") allows us to take a list or dictionary full of arguments and spread them out in a call to match a function’s parameters:

`def show_spread(left, middle, right):     print(f"left {left} middle {middle} right {right}")  all_in_list = [1, 2, 3] show_spread(*all_in_list)  all_in_dict = {"right": 30, "left": 10, "middle": 20} show_spread(**all_in_dict)`

[spread.py](https://third-bit.com/sdxpy/oop/spread.py)

`left 1 middle 2 right 3 left 10 middle 20 right 30`

[spread.out](https://third-bit.com/sdxpy/oop/spread.out)

With these tools in hand, let’s add a method to our `Square` class to tell us whether a square is larger than a user-specified size:

`def square_larger(thing, size):     return call(thing, "area") > size  Square = {     "perimeter": square_perimeter,     "area": square_area,     "larger": square_larger,     "_classname": "Square" }`

[larger.py](https://third-bit.com/sdxpy/oop/larger.py)

The function that implements this check for circles looks exactly the same:

`def circle_larger(thing, size):     return call(thing, "area") > size`

[larger.py](https://third-bit.com/sdxpy/oop/larger.py)

We then modify `call` to capture extra arguments in `*args` and spread them into the function being called:

`def call(thing, method_name, *args):     return thing["_class"][method_name](thing, *args)`

[larger.py](https://third-bit.com/sdxpy/oop/larger.py)

Our tests show that this works:

`examples = [square_new("sq", 3), circle_new("ci", 2)] for ex in examples:     result = call(ex, "larger", 10)     print(f"is {ex['name']} larger? {result}")`

[larger.py](https://third-bit.com/sdxpy/oop/larger.py)

`is sq larger? False is ci larger? True`

[larger.out](https://third-bit.com/sdxpy/oop/larger.out)

However, we now have two functions that do exactly the same thing—the only difference between them is their names. Anything in a program that is duplicated in several places will eventually be wrong in at least one, so we need to find some way to share this code.

[

## Section 2.4: Inheritance

](https://github.com/gvwilson/sdxpy/issues/new?title=/oop%20-%20Section%202.4:%20Inheritance%20(2024-04-28))

The tool we want is inheritance. To see how this works in Python, let’s add a method called `density` to our original `Shape` class that uses other methods defined by the class

`class Shape:     def __init__(self, name):         self.name = name      def perimeter(self):         raise NotImplementedError("perimeter")      def area(self):         raise NotImplementedError("area")      def density(self, weight):         return weight / self.area()`

[inherit_original.py](https://third-bit.com/sdxpy/oop/inherit_original.py)

`examples = [Square("sq", 3), Circle("ci", 2)] for ex in examples:     n = ex.name     d = ex.density(5)     print(f"{n}: {d:.2f}")`

[inherit_original.py](https://third-bit.com/sdxpy/oop/inherit_original.py)

`sq: 0.56 ci: 0.40`

[inherit_original.out](https://third-bit.com/sdxpy/oop/inherit_original.out)

To enable our dictionary-based “classes” to do the same thing, we create a dictionary to represent a generic shape and give it a “method” to calculate density:

`def shape_density(thing, weight):     return weight / call(thing, "area")  Shape = {     "density": shape_density,     "_classname": "Shape",     "_parent": None }`

[inherit_class.py](https://third-bit.com/sdxpy/oop/inherit_class.py)

We then add another specially-named field to the dictionaries for “classes” like `Square` to keep track of their parents:

`Square = {     "perimeter": square_perimeter,     "area": square_area,     "_classname": "Square",     "_parent": Shape }`

[inherit_class.py](https://third-bit.com/sdxpy/oop/inherit_class.py)

and modify the `call` function to search for the requested method ([Figure 2.4](https://third-bit.com/sdxpy/oop/#oop-inherit-class)):

`def call(thing, method_name, *args):     method = find(thing["_class"], method_name)     return method(thing, *args)  def find(cls, method_name):     while cls is not None:         if method_name in cls:             return cls[method_name]         cls = cls["_parent"]     raise NotImplementedError("method_name")`

[inherit_class.py](https://third-bit.com/sdxpy/oop/inherit_class.py)

![Implementing inheritance](https://third-bit.com/sdxpy/oop/inherit_class.svg)

Figure 2.4: Using dictionary search to implement inheritance.

A simple test shows that this is working as intended:

`examples = [square_new("sq", 3), circle_new("ci", 2)] for ex in examples:     n = ex["name"]     d = call(ex, "density", 5)     print(f"{n}: {d:.2f}")`

[inherit_class.py](https://third-bit.com/sdxpy/oop/inherit_class.py)

`sq: 0.56 ci: 0.40`

[inherit_class.out](https://third-bit.com/sdxpy/oop/inherit_class.out)

We do have one task left, though: we need to make sure that when a square or circle is made, it is made correctly. In short, we need to implement [constructors](https://third-bit.com/sdxpy/glossary/#gl:constructor "A function that creates an object of a particular class."). We do this by giving the dictionaries that implement classes a special key `_new` whose value is the function that builds something of that type:

`def shape_new(name):     return {         "name": name,         "_class": Shape     }  Shape = {     "density": shape_density,     "_classname": "Shape",     "_parent": None,     "_new": shape_new }`

[inherit_constructor.py](https://third-bit.com/sdxpy/oop/inherit_constructor.py)

In order to make an object, we call the function associated with its `_new` key:

`def make(cls, *args):     return cls["_new"](*args)`

[inherit_constructor.py](https://third-bit.com/sdxpy/oop/inherit_constructor.py)

That function is responsible for [upcalling](https://third-bit.com/sdxpy/glossary/#gl:upcall "The act of explicitly invoking a method of a parent class from inside a child class. A method in a child class may upcall to the corresponding method in the parent class as part of extending that method.") the constructor of its parent. For example, the constructor for a square calls the constructor for a generic shape and adds square-specific values using `|` to combine two dictionaries:

`def square_new(name, side):     return make(Shape, name) | {         "side": side,         "_class": Square     }  Square = {     "perimeter": square_perimeter,     "area": square_area,     "_classname": "Square",     "_parent": Shape,     "_new": square_new }`

[inherit_constructor.py](https://third-bit.com/sdxpy/oop/inherit_constructor.py)

Of course, we’re not done until we test it:

`examples = [make(Square, "sq", 3), make(Circle, "ci", 2)] for ex in examples:     n = ex["name"]     d = call(ex, "density", 5)     print(f"{n}: {d:.2f}")`

[inherit_constructor.py](https://third-bit.com/sdxpy/oop/inherit_constructor.py)

`sq: 0.56 ci: 0.40`

[inherit_constructor.out](https://third-bit.com/sdxpy/oop/inherit_constructor.out)

[

## Section 2.5: Summary

](https://github.com/gvwilson/sdxpy/issues/new?title=/oop%20-%20Section%202.5:%20Summary%20(2024-04-28))

We have only scratched the surface of Python’s object system. [Multiple inheritance](https://third-bit.com/sdxpy/glossary/#gl:multiple_inheritance "Inheriting from two or more classes when creating a new class."), [class methods](https://third-bit.com/sdxpy/glossary/#gl:class_method "A function defined inside a class that takes the class object as an input rather than an instance of the class."), [static methods](https://third-bit.com/sdxpy/glossary/#gl:static_method "A function that is defined within a class but does not require either the class itself or an instance of the class as a parameter."), and [monkey patching](https://third-bit.com/sdxpy/glossary/#gl:monkey_patching "To replace methods in a class or object at run-time without modifying the original code.") are powerful tools, but they can all be understood in terms of dictionaries that contain references to properties, functions, and other dictionaries ([Figure 2.5](https://third-bit.com/sdxpy/oop/#oop-concept-map)).

![Concept map for objects and classes](https://third-bit.com/sdxpy/oop/concept_map.svg)

Figure 2.5: Concept map for implementing objects and classes.

[

## Section 2.6: Exercises

](https://github.com/gvwilson/sdxpy/issues/new?title=/oop%20-%20Section%202.6:%20Exercises%20(2024-04-28))

### Handling Named Arguments

The final version of `call` declares a parameter called `*args` to capture all the positional arguments of the method being called and then spreads them in the actual call. Modify it to capture and spread named arguments as well.

### Multiple Inheritance

Implement multiple inheritance using dictionaries. Does your implementation look methods up in the same order as Python would?

### Class Methods and Static Methods

1. Explain the differences between class methods and static methods.
    
2. Implement both using dictionaries.
    

### Reporting Type

Python `type` method reports the most specific type of an object, while `isinstance` determines whether an object inherits from a type either directly or indirectly. Add your own versions of both to dictionary-based objects and classes.

### Using Recursion

A [recursive function](https://third-bit.com/sdxpy/glossary/#gl:recursion "To define something in terms of itself, or the act of a function invoking itself (directly or indirectly).") is one that calls itself, either directly or indirectly. Modify the `find` function that finds a method to call so that it uses recursion instead of a loop. Which version is easier to understand? Which version is more efficient?

### Method Caching

Our implementation searches for the implementation of a method every time that method is called. An alternative is to add a [cache](https://third-bit.com/sdxpy/glossary/#gl:cache "Something that stores copies of data so that future requests for it can be satisfied more quickly. The CPU in a computer uses a hardware cache to hold recently-accessed values; many programs rely on a software cache to reduce network traffic and latency. Figuring out when something in a cache is out-of-date and should be replaced is one of the two hard problems in computer science.") to each object to save the methods that have been looked up before. For example, each object could have a special key called `_cache` whose value is a dictionary. The keys in that dictionary are the names of methods that have been called in the past, and the values are the functions that were found to implement those methods. Add this feature to our dictionary-based objects. How much more complex does it make the code? How much extra storage space does it need compared to repeated lookup?
