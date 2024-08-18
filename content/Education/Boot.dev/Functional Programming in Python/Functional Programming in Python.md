---
id: 01J5H7ZT5ZM5ZVEMJAKH0HDBQE
modified: 2024-08-18T00:00:01-04:00
---
# Functional Programming
- ### Part 1: What is functional programming?
	- Functional programming is a style (or "paradigm" if you're pretentious) of programming where we compose functions instead of mutating state (updating the value of variables).
		- [Functional programming](https://en.wikipedia.org/wiki/Functional_programming) is more about declaring _what_ you want to happen, rather than _how_ you want it to happen.
		- [Imperative](https://en.wikipedia.org/wiki/Imperative_programming) (or procedural) programming declares both the _what_ and the _how_.
**Example of imperative code:**
```py
car = create_car()
car.add_gas(10)
car.clean_windows()
```
**Example of functional code:**

```py
return clean_windows(add_gas(create_car()))
```
- The important distinction is that in the functional example, we never change the value of the `car` variable, we just compose functions that return new values, with the outermost function, `clean_windows` in this case, returning the final result.
- ## Doc2Doc
	- In this course, we're working on "Doc2Doc", a command line tool for converting documents from one format to another. If you're familiar with [Pandoc](https://pandoc.org/), the idea is similar.
## Assignment
- Complete the `stylize_title` function. It should take a single string as input, and return a single string as output. The returned string should have both the title centered and a border added.
	- Use the provided functions `center_title` and `add_border`.
	- Center the title _before_ adding the border.
	- Do not create any variables.
	- Use only 1 line of code in the function body.
# It's Math

Functional programming tends to be popular amongst developers with a strong mathematical background. After all, a math equation isn't procedural: it's declarative. Take the following math equation:

```
avg = Σx/N
```
To put this calculation in plain English:

1. `Σ` is just the Greek letter [Sigma](https://en.wikipedia.org/wiki/Sigma), and it represents "the [sum](https://en.wikipedia.org/wiki/Summation) of a collection".
2. `x` is the collection of numbers we're averaging.
3. `N` is the number of elements in the collection.
4. `avg` is equal to the sum of all the numbers in collection "x" divided by the number of elements in collection "x".
So, the equation really just says that `avg` is the average of all the numbers in collection "x". This math equation is a _declarative_ way of writing "calculate the average of a list of numbers". Here's some _imperative Python code_ that does the same thing:

```py
def get_average(nums):
    total = 0
    for num in nums:
        total += num
    return total / len(nums)
```
However, with functional programming, we would write code that's _a bit more_ declarative:
```py
def get_average(nums):
    return sum(nums) / len(nums)
```
Here we're not keeping track of state (the `total` variable in the first example is ["stateful"](https://en.wikipedia.org/wiki/State_(computer_science)#:~:text=In%20information%20technology%20and%20computer,known%20as%20its%20state%20space.)). We're simply composing functions together to get the result we want.
## Assignment
In the world of document conversion, we sometimes need to handle fonts and font sizes.
Complete the `get_median_font_size` function. Given a list of numbers representing font sizes, return the [median](https://en.wikipedia.org/wiki/Median) of the list.
For example:
```
[1, 2, 3] => 2
[10, 8, 7, 5] => 7
```
Notice the second list is out of order. Order the list, then find the middle index, and return the middle number. If there is an even amount of numbers, return the smaller of the two middle numbers (I know it's not a _true_ median, but good for our purposes). If the list is empty, just return `None`.

Here are some helpful docs:
- [sorted](https://docs.python.org/3/library/functions.html#sorted)
- [len](https://docs.python.org/3/library/functions.html#len)
- [//](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex) (floor division)
To be a good little functional programmer, your code for this lesson should **not**:
1. Use loops
2. Mutate any variables (it's okay to create new ones)
### Answer:
```python
def get_median_font_size(font_sizes):
    x = len(font_sizes) // 2
    new_list = sorted(font_sizes)
    if font_sizes == []:
        return None
    elif len(font_sizes) % 2 == 0:
        return new_list[int(x) - 1]
    else:
        return new_list[int(x)]

```
#### Test Suite
```python
from main import *

run_cases = [
    ([4, 3, 2, 1, 5], 3),
    ([20, 14, 16], 16),
    ([9, 11, 16, 20], 11),
]

submit_cases = run_cases + [
    ([8, 8, 8], 8),
    ([30, 18, 14, 22], 18),
    ([6, 24, 6, 6, 24, 24, 2, 1, 3], 6),
    ([], None),
]


def test(input, expected_output):
    print("---------------------------------")
    print(f"Input: {input}")
    print(f"Expected: {expected_output}")
    input_copy = input.copy()
    result = get_median_font_size(input)
    print(f"Actual: {result}")
    if result != expected_output:
        print("Fail")
        return False
    if input != input_copy:
        print(f"Expected font_sizes: {input_copy}")
        print(f"Actual font_sizes: {input}")
        print("font_sizes was modified")
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
# Classes vs Functions

I run into new developers who, after learning about classes, want to use them _everywhere_. They assume that because they learned about functions first, functions are somehow inferior.

**Nope.** They're just different.

## Should I use functions or classes?

Here's my rule of thumb:

**If you're unsure, default to functions.** I find myself reaching for classes when I need something long-lived and stateful that would be easier to model if I could share behavior _and data structure_ via inheritance. This is often the case for:

- Video games
- Simulations
- GUIs

The difference is:

> **Classes** encourage you to think about the world as a hierarchical collection of objects. Objects bundle behavior, data, and state together in a way that draws boundaries between instances of things, like chess pieces on a board.

> **Functions** encourage you to think about the world as a series of data transformations. Functions take data as input and return a transformed output. For example, a function might take the entire state of a chess board and a move as inputs, and return the new state of the board as output.

_Use what feels right to you in your projects, and adjust and refactor as you improve your skills._

# Debugging FP

It's _nearly impossible_, even for tenured senior developers, to write perfect code the first time. That's why debugging is such an important skill. The trouble is, sometimes you have these "elegant" (sarcasm intended) one-liners that are tricky to debug:

```py
def get_player_position(position, velocity, friction, gravity):
    return calc_gravity(calc_friction(calc_move(position, velocity), friction), gravity)
```
If the output of `get_player_position` is incorrect, it's hard to know what's going on inside that black box. _Break it up!_ Then you can inspect the `moved`, `slowed`, and `final` variables more easily:
```py
def get_player_position(position, velocity, friction, gravity):
    moved = calc_move(position, velocity)
    slowed = calc_friction(moved, friction)
    final = calc_gravity(slowed, gravity)
    print("Given:")
    print(f"position: {position}, velocity: {velocity}, friction: {friction}, gravity: {gravity}")
    print("Results:")
    print(f"moved: {moved}, slowed: {slowed}, final: {final}")
    return final
```
Once you've run it, found the issue, and solved it, you can remove the `print` statements.
## Assignment

Fix the `format_line` function. It should apply the following transformations in order:

1. Strip whitespace from the beginning and end of the line.
2. Capitalize every character in the line.
3. Remove any periods from the line.
4. Suffix the line with an ellipsis: `words go here...`

_Run the code._ You should see that some subtle bugs are present.

Break up the function to make it easier to debug. Use `print()` statements to see what's going on at each step.

## Tips

Be careful about whitespace! It's easy to miss in console output. I sometimes add a character, like a `|` to the beginning and end of a string to make whitespace more obvious when print debugging.

- [.replace(old, new)](https://docs.python.org/3/library/stdtypes.html#str.replace) can be used to replace all instances of a character in a string.
- [.upper()](https://docs.python.org/3/library/stdtypes.html#str.upper) capitalizes an entire string, [.capitalize()](https://docs.python.org/3/library/stdtypes.html#str.capitalize) capitalizes the first letter.
- [.strip()](https://docs.python.org/3/library/stdtypes.html#str.strip) removes whitespace from the beginning and end of a string, [.lstrip()](https://docs.python.org/3/library/stdtypes.html#str.lstrip) and [.rstrip()](https://docs.python.org/3/library/stdtypes.html#str.rstrip) remove whitespace from the left and right respectively.
### Answer:
```python
def format_line(line):
    return f"{line.strip().upper().replace('.', '')}..."
```
# Functional vs OOP

Functional programming and object-oriented programming are **styles for writing code**. One isn't inherently superior to the other, but to be a well-rounded developer you should understand both well and use ideas from each when appropriate.

You'll encounter developers who love functional programming and others who love object-oriented programming. However, contrary to popular opinion, FP and OOP are _not_ always at odds with one another. They aren't opposites. Of the four pillars of OOP, [inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)) is the only one that doesn't fit with functional programming.

![oop vs fp](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/8ZrwpU2.png)

Inheritance isn't seen in functional code due to the mutable classes that come along with it. Encapsulation, polymorphism and abstraction are still used all the time in functional programming.

When working in a language that supports ideas from both FP and OOP (like Python, JavaScript, or Go) the best developers are the ones who can use the best ideas from both paradigms effectively and appropriately.