---
id: 01J5HTANG6MS6BC5GW7T06HNKP
title: First Class Functions
modified: 2024-08-18T00:03:03-04:00
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