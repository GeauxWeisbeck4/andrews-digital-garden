---
id: 01J5HTANG6MS6BC5GW7T06HNKP
title: First Class Functions
modified: 2024-08-18T17:27:24-04:00
description: Learning about First Class Functions in Python
tags:
  - python
  - first-class-functions
  - functional-programming
  - boot-dev
---
# First Class Functions
# Functions as Values
In Python, functions are just values, like strings, integers, or objects. For example, we can assign an existing function to a variable:

```py
def add(x, y):
    return x + y

# assign the function to a new variable
# called "addition". It behaves the same
# as the original "add" function
addition = add
print(addition(2, 5))
# 7
```
## Assignment

With the popularity of generative AI (like ChatGPT), we need to be able to convert files into pure text to be injected into prompts.

**Complete the `file_to_prompt` function.** It should take a `file` dictionary and a `to_string` function as inputs and return a formatted string. The provided `to_string` function is responsible for converting the file dictionary into a string: you don't need to implement it, it's a value passed to your function.

However, your function should wrap the result of the `to_string` function in triple backticks (` ``` `) to format it as a code block in Markdown. For example:

```
an example string
```
should become:
````md
```
an example string
```
````
## Tips

Notice the two newlines in the example above! You don't need a trailing newline, but you do need the 2 newlines between the triple backticks. You can achieve this by using the newline [`\n`](https://en.wikipedia.org/wiki/Newline) escape character. Here's an example:

```py
print("I wish the ring had never come to me.\nI wish none of this had happened.")
```
becomes:
```
I wish the ring had never come to me.
I wish none of this had happened.
```
## Answer
```python
def file_to_prompt(file, to_string):
    new_string = to_string(file)
    return f"```\n{new_string}\n```"
```
# Anonymous Functions

Anonymous functions have _no name_, and in Python, they're called [lambda functions](https://docs.python.org/3/reference/expressions.html#lambda) after [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus). Here's a lambda function that takes a single argument `x` and returns the result of `x + 1`:

```py
lambda x: x + 1
```
Notice that the [expression](https://docs.python.org/3/reference/expressions.html#expressions) `x + 1` is returned _automatically_, no need for a `return` statement. And because functions are just values, we can assign the function to a variable named `add_one`:

```py
add_one = lambda x: x + 1
print(add_one(2))
# 3
```
Lambda functions might _look_ scary, but they're still just functions. Because they simply return the result of an expression, they're often used for small, simple evaluations. Here's an example that uses a lambda to get a value from a dictionary:
```py
get_age = lambda name: {
    'lane': 29,
    'hunter': 69,
    'allan': 17
}.get(name, 'not found')
print(get_age('lane'))
# 29
```
## Assignment

Complete the `file_type_getter` function. This function accepts a list of tuples, where each tuple contains:

1. A "file type" (e.g. "code", "document", "image", etc)
2. A list of associated file extensions (e.g. `['.py', '.js']` or `['.docx', '.doc']`)

First, use loops to create a dictionary that maps each file extension to its corresponding file type, based on the input tuples. For example, the resulting dictionary might be:

```py
{
    '.doc': 'text',
    '.docx': 'document',
    '.py': 'code',
    '.jpg': 'image'
}
```
Next, return a lambda function that accepts a string (a file extension) and returns the corresponding file type. If the extension is not found in the dictionary, the lambda function should return `"Unknown"`. I used the [.get](https://docs.python.org/3/library/stdtypes.html#dict.get) dictionary method to handle this.
## Answer
```python
def file_type_getter(file_extension_tuples):
    extension_map = {}
    for file_type, extensions in file_extension_tuples:
        for extension in extensions:
            extension_map[extension] = file_type
            get_extension = lambda extension: extension_map.get(extension, "Unknown")
    return get_extension
```
# First Class and Higher Order Functions

A programming language "supports first-class functions" when functions are treated like any other variable. That means functions can be passed as arguments to other functions, can be returned by other functions, and can be assigned to variables.

- **First-class function:** A function that is treated like any other value
- **Higher-order function:** A function that accepts another function as an argument or returns a function

Python supports first-class and higher-order functions.

## First-class example

```py
def square(x):
    return x * x

# Assign function to a variable
f = square

print(f(5))
# 25
```
## Higher-order example

```py
def square(x):
    return x * x

def my_map(func, arg_list):
    result = []
    for i in arg_list:
        result.append(func(i))
    return result

squares = my_map(square, [1, 2, 3, 4, 5])
print(squares)
# [1, 4, 9, 16, 25]
```
# Map

"Map", "filter", and "reduce" are three commonly used [higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function) in functional programming.

In Python, the built-in [map](https://docs.python.org/3/library/functions.html#map) function takes a function and an [iterable](https://docs.python.org/3/glossary.html#term-iterable) (in this case a list) as inputs. It applies the function to each element in the iterable and returns a new iterable with all the results.

![map function](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/J5hKgxZ.png)

With `map`, we can operate on lists without using loops and nasty stateful variables. For example:

```py
def square(x):
    return x * x

nums = [1, 2, 3, 4, 5]
squared_nums = map(square, nums)
print(list(squared_nums))
# [1, 4, 9, 16, 25]
```

_The [list type constructor](https://docs.python.org/3/library/stdtypes.html#list), `list()` converts the `map` object back into a standard list._

## Assignment

[Markdown](https://www.markdownguide.org/cheat-sheet/) supports two different styles of bullet points, `-` and `*`. We prefer `*`, so, we need a function to convert any `-` bullet points to `*` bullet points.

Complete the `change_bullet_style` function. It takes a document (a string) as input, and returns a single string as output. The returned string should have any lines that start with a `-` character replaced with a `*` character.

For example, this:

```
- This is a bullet
- This is a bullet
```
Becomes:
```
* This is a bullet
* This is a bullet
```
Use the built-in [map](https://docs.python.org/3/library/functions.html#map) function to apply the provided `convert_line` function to each line of the input string. Use [.split()](https://docs.python.org/3/library/stdtypes.html#str.split) and [.join()](https://docs.python.org/3/library/stdtypes.html#str.join) to split the document into a list of lines, and then join the lines back together. This should preserve the original line breaks. Don't use the `.replace()` string method.

Examples of split and join:

```py
# my_document is a string with newlines
lines_list = my_document.split("\n")

rejoined_doc = "\n".join(lines_list)
```
## Answer
```python
def change_bullet_style(document):
    line_list = document.split("\n")
    converted_lines = list(map(convert_line, line_list))
    return "\n".join(converted_lines)


# Don't edit below this line


def convert_line(line):
    old_bullet = "-"
    new_bullet = "*"
    if len(line) > 0 and line[0] == old_bullet:
        return new_bullet + line[1:]
    return line

```
# Filter

The built-in [filter](https://docs.python.org/3/library/functions.html#filter) function takes a function and an iterable (in this case a list) and returns a _new_ iterable that only contains elements from the original iterable where the result of the function on that item returned `True`.

![filter function](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/FfVxD7d.png)

In Python:

```py
def is_even(x):
    return x % 2 == 0

numbers = [1, 2, 3, 4, 5, 6]
evens = list(filter(is_even, numbers))
print(evens)
# [2, 4, 6]
```
## Assignment
Complete the `remove_invalid_lines` function. It accepts a document as input. It should use the built-in `filter` function and a lambda to return a copy of the input document but _remove_ any lines that start with a `-` character. Keep _all_ other lines and **preserve trailing newlines**. This:
```
* Star Wars episode 1 is underrated
- Star Wars episode 9 is fine
* Star Wars episode 3 is the best

```
Should become:
```
* Star Wars episode 1 is underrated
* Star Wars episode 3 is the best

```
## Tips
[.join](https://docs.python.org/3/library/stdtypes.html#str.join)
```py
"\n".join(["a", "b", "c"])
# a
# b
# c
```
[.startswith](https://docs.python.org/3/library/stdtypes.html#str.startswith)

```py
s = "hello"
s.startswith("h")
# True
s.startswith("o")
# False
```
[.split](https://docs.python.org/3/library/stdtypes.html#str.split)

```py
s = """hello
world"""
lines = s.split("\n")
# ['hello', 'world']
```
## Answer
```python
def remove_invalid_lines(document):
    lines = document.split("\n")
    filtered_lines = list(filter(lambda line: not line.startswith("-"), lines))
    return "\n".join(filtered_lines)
    
```
# Reduce

The built-in [functools.reduce()](https://docs.python.org/3/library/functools.html#functools.reduce) function takes a function and a list of values, and applies the function to each value in the list, _accumulating a single result_ as it goes.

![reduce function](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/HxX0hTx.png)

```py
# import functools from the standard library
import functools

def add(sum_so_far, x):
    print(f"sum_so_far: {sum_so_far}, x: {x}")
    return sum_so_far + x

numbers = [1, 2, 3, 4]
sum = functools.reduce(add, numbers)
# sum_so_far: 1, x: 2
# sum_so_far: 3, x: 3
# sum_so_far: 6, x: 4
# 10 doesn't print, it's just the final result
print(sum)
# 10
```
## Assignment

Complete the `join` and the `join_first_sentences` functions.

### join()

This is a helper function we'll use in `join_first_sentences`. It takes two inputs:

1. A "doc_so_far" accumulator string. It's similar to the `sum_so_far` variable in the example above.
2. A "sentence" string. This is the next string we want to add to the accumulator.

It returns the result of concatenating the "doc" and "sentence" strings together, with a period and a space in between. For example:

```py
doc = "This is the first sentence"
sentence = "This is the second sentence"
print(join(doc, sentence))
# This is the first sentence. This is the second sentence
```
### join_first_sentences()

It accepts two arguments:

1. A list of sentence strings
2. An integer `n`

It should use the built-in [functools.reduce()](https://docs.python.org/3/library/functools.html#functools.reduce) function alongside your `join` function to return a single string: the result of joining the _first_ `n` sentences in the list. It should also add a final period (but no trailing space) to the end of the final "reduced" string.

If `n` is zero, just return an empty string.

Use [list slicing](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations) to get the first `n` sentences. For example:

```py
fruits = ["apple", "banana", "cherry", "date"]
print(fruits[:2])
# ["apple", "banana"]
```

Here's an example of the expected behavior:
```py
joined = join_first_sentences(
    ["This is the first sentence", "This is the second sentence", "This is the third sentence"],
    2
)
print(joined)
# This is the first sentence. This is the second sentence.
```
```python
import functools


def join(doc_so_far, sentence):
    return doc_so_far + ". " + sentence


def join_first_sentences(sentences, n):
    if n == 0:
        return ""
    else:
        first_n_sentences = sentences[:n]
        joined_string = functools.reduce(join, first_n_sentences)
        return joined_string + "."
```
# Map, Filter, and Reduce Review
Higher-order functions like `map`, `filter`, and `reduce`, allow us to avoid stateful iteration and mutations of variables.
Take a look at this [imperative](https://en.wikipedia.org/wiki/Imperative_programming) code that calculates the [factorial](https://en.wikipedia.org/wiki/Factorial) of a number:
```py
def factorial(n):
    # a procedure that continuously multiplies
    # the current result by the next number
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result
```
Here's the same factorial function using `reduce`:
```py
import functools

def factorial(n):
    return functools.reduce(lambda x, y: x * y, range(1, n + 1))
```
In the functional example, we're just combining functions to get the result we want. There's no need to reassign variables or keep track of the program's state in a loop.
A loop is inherently stateful. Depending on which iteration you're on, the `i` variable has a different value.

