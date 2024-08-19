---
id: 01J5KP889E91933M73TG2WFY0A
title: Pure Functions
modified: 2024-08-18T20:27:13-04:00
description: How to use pure functions in Python
tags:
  - functional-programming
  - python
  - boot-dev
  - pure-functions
---
# Pure Functions
If you take nothing else away from this course, _please_ take this: **[Pure functions](https://en.wikipedia.org/wiki/Pure_function) are fantastic.** They have two properties:
- They _always_ return the same value given the same arguments.
- Running them causes no side effects
In short: **pure functions don't do anything with anything that exists outside of their scope.**

![pure function diagram](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/UnrcH6K.png)

## Example of a pure function

```py
def find_max(nums):
    max_val = float("-inf")
    for num in nums:
        if max_val < num:
            max_val = num
    return max_val
```
## Example of an impure function
```py
# instead of returning a value
# this function modifies a global variable
global_max = float("-inf")

def find_max(nums):
    global global_max
    for num in nums:
        if global_max < num:
            global_max = num
```
## Assignment
There's a bug in the `convert_file_format` function! As it stands, it depends on some data that isn't scoped locally within the function. Those global values are mutated by external functions and are not guaranteed to be the same every time `convert_file_format` is called.
Fix the bug by making `convert_file_format` a pure function. It should only depend on data that is scoped inside of the function.
## Answer
```python
def convert_file_format(filename, target_format):
    valid_extensions = ["docx", "pdf", "txt", "pptx", "ppt", "md"]
    valid_conversions = {
        "docx": ["pdf", "txt", "md"],
        "pdf": ["docx", "txt", "md"],
        "txt": ["docx", "pdf", "md"],
        "pptx": ["ppt", "pdf"],
        "ppt": ["pptx", "pdf"],
        "md": ["docx", "pdf", "txt"],
    }
    current_format = filename.split(".")[-1]
    if (
        current_format in valid_extensions
        and target_format in valid_conversions[current_format]
    ):
        return filename.replace(current_format, target_format)
    return None
```
# Pure Function Review
Pure functions have a _lot_ of benefits. Whenever possible, good developers try to use pure functions instead of impure functions. Remember, pure functions:
- Return the same result if given the same arguments. They are [deterministic](https://en.wikipedia.org/wiki/Deterministic_system).
- Do not change the external state of the program. For example, they do not change any variables outside of their scope.
- Do not perform any [I/O operations](https://en.wikipedia.org/wiki/Input/output) (like reading from disk, accessing the internet, or writing from the console).
These properties result in pure functions being easier to test, debug, and think about.
Refer to the following examples and answer the questions.
## Example 1
```py
def multiply_by2(nums):
    products = []
    for num in nums:
        products.append(num*2)
    return products
```
## Example 2
```py
balance = 1000
cars = []

def buy_car(new_car):
    cars.append(new_car)
    balance -= 69
```
## Example 3
```py
import random

def roll_die(num_sides):
    return random.randint(1, num_sides)
```
# Reference vs. Value
When you pass a value into a function as an argument, one of two things can happen:
- It's passed by **reference**: The function has access to the original value and can change it
- It's passed by **value**: The function only has access to a copy. Changes to the copy within the function don't affect the original
_There is a bit more nuance, but this explanation mostly works._
These types are passed by **reference**:
- Lists
- Dictionaries
- Sets
These types are passed by **value**:
- Integers
- Floats
- Strings
- Booleans
- Tuples
Most collection types are passed by reference (except for tuples) and most primitive types are passed by value.
## Example of Pass by Reference (mutable)
```py
def modify_list(inner_lst):
    inner_lst.append(4)
    # the original "outer_lst" is updated
    # because inner_lst is a reference to the original

outer_lst = [1, 2, 3]
modify_list(outer_lst)
# outer_lst = [1, 2, 3, 4]
```
## Example of Pass by Value (immutable)
```py
def attempt_to_modify(inner_num):
    inner_num += 1
    # the original "outer_num" is not updated
    # because inner_num is a copy of the original

outer_num = 1
attempt_to_modify(outer_num)
# outer_num = 1
```
## Assignment
We have a way for Doc2Doc users to set their supported formats in their settings. In memory, we store those settings as a simple dictionary:
```py
settings = {
    "docx": True,
    "pdf": True,
    "txt": False
}
```
Unfortunately, there is a bug in our code! When a new format is added or removed, it not only updates the new dictionary, but it changes the defaults themselves! That's not good. We want to create a _new_ dictionary with the updates, not change the original.
Fix the bug by making `add_format` and `remove_format` pure functions that don't mutate their inputs.
## Tip
The [.copy()](https://docs.python.org/3/library/copy.html) method can be used to create a new copy of a dictionary.
```python
def add_format(default_formats, new_format):
    updated_formats = default_formats.copy()
    updated_formats[new_format] = True
    return updated_formats


def remove_format(default_formats, old_format):
    updated_formats = default_formats.copy()
    updated_formats[old_format] = False
    return updated_formats
```
# Pass by Reference Impurity
Because certain types in Python are passed by reference, we can mutate values that we didn't intend to. This is a form of function impurity.
Remember, a pure function should have _no side effects_. It shouldn't modify anything outside of its scope, _including its inputs_. It should return new copies of inputs instead of changing them.
## Pure function
```py
def remove_format(default_formats, old_format):
    new_formats = default_formats.copy()
    new_formats[old_format] = False
    return new_formats
```
## Impure function
```py
def remove_format(default_formats, old_format):
    default_formats[old_format] = False
    return default_formats
```
## Why do we care?
One of the biggest differences between good and great developers is how often they incorporate pure functions into their code. Pure functions are easier to read, easier to reason about, easier to test, and easier to combine. Even if you're working in an imperative language like Python, you can (and should) write pure functions whenever reasonable.
There's nothing worse than trying to debug a program where the order functions are called needs to be _juuuuust right_ because they all read and modify the same global variable.
# Input and Output

![xkcd "side effects"](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/SVpmfNV.png)

Comic by [xkcd](https://xkcd.com/1790/).

The term "i/o" stands for input/output. In the context of writing programs, i/o refers to anything in our code that interacts with the "outside world". "Outside world" just means anything that's not stored in our application's memory (like variables).

## Examples of i/o

- Reading from or writing to a file on the hard drive
- Accessing the internet
- Reading from or writing to a database
- Even simply _printing to the console_ is considered i/o!

_All i/o is a form of "side effect"._

## Assignment

In Doc2Doc, we frequently need to change the casing of some text. For example:

### TitleCase

> Every Day Once A Day Give Yourself A Present

### LowerCase

> every day once a day give yourself a present

### UpperCase

> EVERY DAY ONCE A DAY GIVE YOURSELF A PRESENT

There is an issue in the `convert_case` function, our test suite can't test its behavior because it's printing to the console (eww... a _side-effect_) instead of returning a value. Fix the function so that it returns the correct value instead of printing it.

# Should I i/o?
A program that doesn't do _any_ i/o is pretty useless. What's the point of computing something if you can't see the results?
![fp meme](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/LUg4s1f.jpeg)
In functional programming, i/o is viewed as _dirty_ but _necessary_. We know we can't _eliminate_ i/o from our code, so we just _contain_ it as much as possible. There should be a clear place in your project that does nasty i/o stuff, and the rest of your code can be pure.
For example, a Python program might:
1. Read a file from the hard drive as the program starts
2. Run a bunch of pure functions to analyze the data
3. Write the results of the analysis to a file on the hard drive at the end
![io](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/45emq7q.png)

