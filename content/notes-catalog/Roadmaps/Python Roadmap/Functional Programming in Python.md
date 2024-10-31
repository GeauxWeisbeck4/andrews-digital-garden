---
id: 01JBFR0YDPK34T2DHD0DG605FK
title: Functional Programming in Python
modified: 2024-10-30T18:20:13-04:00
tags:
  - functional-programming
  - python
  - roadmaps
  - programming
---
# Functional Programming in Python
- It doesn't play a huge role in Python, but it is still useful to know.
- A **pure function** is a function whose output value follows solely from its input values without any observable [side effects](https://realpython.com/defining-your-own-python-function/#side-effects). In **functional programming**, a program consists primarily of the evaluation of pure functions. Computation proceeds by nested or [composed function calls](https://en.wikipedia.org/wiki/Function_composition_(computer_science)) without changes to state or mutable data.
- The functional paradigm is popular because it offers several advantages over other programming paradigms. Functional code is:
	- **High level:** You describe the result you want rather than explicitly specifying the steps required to get there. Single statements tend to be concise but pack a lot of punch.
	- **Transparent:** The behavior of a pure function can be described by its inputs and outputs, without intermediary values. This eliminates the possibility of side effects and facilitates [debugging](https://realpython.com/python-debugging-pdb/).
	- **Parallelizable:** Routines that don’t cause side effects can more easily [run in parallel](https://realpython.com/learning-paths/python-concurrency-parallel-programming/) with one another.
- To support functional programming, it’s beneficial if a [function](https://realpython.com/defining-your-own-python-function/) in a given programming language can do these two things:
	1. Take another function as an argument
	2. Return another function to its caller
- Python plays nicely in both respects. Everything in Python is an [object](https://realpython.com/python-variables/#object-references), and all objects in Python have more or less equal stature. Functions are no exception.
- In Python, functions are **first-class citizens**. This means that functions have the same characteristics as values like [strings](https://realpython.com/python-strings/) and [numbers](https://realpython.com/python-numbers/). Anything you would expect to be able to do with a string or number, you can also do with a function.
	- For example, you can assign a function to a variable. You can then use that variable the same way you would use the function itself:
```python

```
## Resources
- [Functional Programming in Python: When and How to Use It – Real Python](https://realpython.com/python-functional-programming/)
- 