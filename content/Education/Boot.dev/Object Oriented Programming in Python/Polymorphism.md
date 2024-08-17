---
id: 01J5GPD84R1AQPHVFSRG90W9KJ
modified: 2024-08-17T14:17:52-04:00
title: Polymorphism
description: Using Polymorphism in Python
tags:
  - polymorphism
  - python
  - boot-dev
---
# Polymorphism
- # Polymorphism Review
	- Take a look at the Greek roots of the word "polymorphism".
	- "poly" means "many".
	- "morph" means "to change" or "form".
- Polymorphism in programming is the ability to present the same interface (function or method signatures) for many different underlying forms (data types).
- A classic example is a `Shape` class that `Rectangle`, `Circle`, and `Triangle` can inherit from. With polymorphism, each of these classes will have different underlying data. The circle needs its center point coordinates and radius. The rectangle needs two coordinates for the top left and bottom right corners. The triangle needs coordinates for the corners.
- By making each class responsible for its data **and** its code, you can achieve polymorphism. In the shapes example, each class would have its own `draw_shape()` method. This allows the code that uses the different shapes to be simple and easy, and more importantly, it can treat shapes as the **same** even though they are **different**. It hides the complexities of the difference behind a clean abstraction.

```python
shapes = [Circle(5, 5, 10), Rectangle(1, 3, 5, 6)]
for shape in shapes:
    print(shape.draw_shape())
```
- This is in contrast to the functional way of doing things where you would have had separate functions that have **different** function signatures, like `draw_rectangle(x1, y1, x2, y2)` and `draw_circle(x, y, radius)`.
## Wait, what is a "function signature"?
- A function signature (or method signature) includes the name, inputs, and outputs of a function or method. For example, `hit_by_fire` in the `Human` and `Archer` classes have identical signatures.
```python
class Human:
    def hit_by_fire(self):
        self.health -= 5
        return self.health

class Archer:
    def hit_by_fire(self):
        self.health -= 10
        return self.health
```
- Both methods have the same name, take `0` inputs, and return integers. If _any_ of those things were different, they would have different function signatures.
- Here is an example of different signatures:
```python
class Human:
    def hit_by_fire(self):
        self.health -= 5
        return self.health

class Archer:
    def hit_by_fire(self, dmg):
        self.health -= dmg
        return self.health
```
- The `Archer` class's `hit_by_fire` method takes an input, `dmg`, which is used to calculate the new health. This is a different signature than the `Human` class's `hit_by_fire` method, which takes no inputs. Calling two methods with the same signature should look the same to the caller.

```python
# same inputs (none)
# same name
# same outputs (a single integer)
health = sam.hit_by_fire()
health = archer.hit_by_fire()
```
## When overriding methods, use the same function signature
- If you change the function signature of a parent class when overriding a method, it could be a disaster. The whole point of overriding a method is so that the caller of your code _doesn't have to worry_ about what different things are going on inside the methods of different object types.

# Operator Overloading
- Another kind of built-in polymorphism in Python is the ability to override how an operator works. For example, the `+` operator works for built-in types like integers and strings.
```python
print(3 + 4)
# prints "7"

print("three " + "four")
# prints "three four"
```
- Custom classes on the other hand don't have any built-in support for those operators:
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y


p1 = Point(4, 5)
p2 = Point(2, 3)
p3 = p1 + p2
# TypeError: unsupported operand type(s) for +: 'Point' and 'Point'
```
- However, we can add our own support! If we create an `__add__(self, other)` method on our class, the Python interpreter will use it when instances of the class are being added with the `+` operator. Here's an example:
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, point):
        x = self.x + point.x
        y = self.y + point.y
        return Point(x, y)

p1 = Point(4, 5)
p2 = Point(2, 3)
p3 = p1 + p2
# p3 is (6, 8)
```
- Now, when `p1 + p2` is executed, under the hood the Python interpreter just calls `p1.__add__(p2)`.
## Assignment
- In Age of Dragons, players can upgrade their weaponry. To make "crafting" simple for other developers, we'll use operator overloading on the `Sword` class. Note how the test suite attempts to use the `+` operator to craft the swords. Overload the `+` operator to craft the swords.
- Create an `__add__(self, other)` method on the `Sword` class. It will be used to "craft" two swords together to create a new sword. `sword_type` is just a string, one of:
	- `bronze`
	- `iron`
	- `steel`
		- If two "bronze" swords are crafted together, return a new "iron" sword.
		- If two "iron" swords are crafted together, return a new "steel" sword.
		- If a player tries to craft anything other than 2 bronze swords or 2 iron swords, just `raise` an `Exception` with the message `"can not craft"`.
#### Answer
```python
class Sword:
    def __init__(self, sword_type):
        self.sword_type = sword_type

    def __add__(self, other):
        if self.sword_type == "bronze" and other.sword_type == "bronze":
            return Sword("iron")
        elif self.sword_type == "iron" and other.sword_type == "iron":
            return Sword("steel")
        else:
            raise Exception("can not craft")
        
```
#### Test Suite
```python
from main import *

run_cases = [
    (Sword("bronze"), Sword("bronze"), "iron", None),
    (Sword("bronze"), Sword("iron"), None, "can not craft"),
]

submit_cases = run_cases + [
    (Sword("steel"), Sword("steel"), None, "can not craft"),
    (Sword("iron"), Sword("iron"), "steel", None),
    (Sword("bronze"), Sword("steel"), None, "can not craft"),
]


def test(sword1, sword2, expected_result, expected_err):
    try:
        print("---------------------------------")
        print(
            f"Crafting a {sword1.sword_type} sword with a {sword2.sword_type} sword..."
        )
        result = sword1 + sword2

        if expected_err:
            print(f"Expected Exception: {expected_err}")
            print("Actual: no exception raised")
            print("Fail")
            return False

        print(f"Expected: {expected_result}")
        print(f"Actual: {result.sword_type}")
        print(f"A new {result.sword_type} sword was crafted!")
        if result.sword_type != expected_result:
            print("Fail")
            return False

    except Exception as e:
        print(f"Expected Exception: {expected_err}")
        print(f"Actual Exception: {e}")
        if expected_err != str(e):
            print("Fail")
            return False

    print("Pass")
    return True


def main():
    passed = 0
    failed = 0
    for test_case in test_cases:
        correct = test(*test_case)
        if correct:
            passed += 1
        else:
            failed += 1
    if failed == 0:
        print("============= PASS ==============")
    else:
        print("============= FAIL ==============")
    print(f"{passed} passed, {failed} failed")


test_cases = submit_cases
if "__RUN__" in globals():
    test_cases = run_cases

main()

```
# Operator Overload Review

As we discussed in the last assignment, operator overloading is the practice of defining custom behavior for standard Python operators. Here's a list of how the operators translate into method names.

| Operation           | Operator | Method       |
| ------------------- | -------- | ------------ |
| Addition            | +        | __add__      |
| Subtraction         | -        | __sub__      |
| Multiplication      | *        | __mul__      |
| Power               | **       | __pow__      |
| Division            | /        | __truediv__  |
| Floor Division      | //       | __floordiv__ |
| Remainder (modulo)  | %        | __mod__      |
| Bitwise Left Shift  | <<       | __lshift__   |
| Bitwise Right Shift | >>       | __rshift__   |
| Bitwise AND         | &        | __and__      |
| Bitwise OR          | \|       | __or__       |
| Bitwise XOR         | ^        | __xor__      |
| Bitwise NOT         | ~        | __invert__   |
# Overloading Built-in Methods
- Last but not least, let's take a look at some of the built-in methods we can overload in Python. While there isn't a default behavior for the arithmetic operators like we just saw, there _is_ a default behavior for **printing** a class.
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y


p1 = Point(4, 5)
print(p1)
# prints "<Point object at 0xa0acf8>"
```
- That's not super useful! Let's teach instances of our `Point` object to print themselves. The [`__str__`](https://docs.python.org/3/reference/datamodel.html#object.__str__) method (short for "string") lets us do just that. It takes no inputs but returns a string that will be printed to the console when someone passes an instance of the class to Python's `print()` function.
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"({self.x},{self.y})"

p1 = Point(4, 5)
print(p1)
# prints "(4,5)"
```
_Note: the [`__repr__`](https://docs.python.org/3/reference/datamodel.html#object.__repr__) method works in a similar way, you'll see it from time to time._
## Assignment
- Dragons are egotistical creatures, let's give them a great format for announcing their presence in "Age of Dragons". When `print()` is called on an instance of a `Dragon`, the string `I am {0}, the {1} dragon` should be printed.
	- `{0}` is the name of the dragon.
	- `{1}` is the color of the dragon.
### Answer
```python
class Dragon:
    def __init__(self, name, color):
        self.name = name
        self.color = color

    def __str__(self):
        return f"I am {self.name}, the {self.color} dragon"

```