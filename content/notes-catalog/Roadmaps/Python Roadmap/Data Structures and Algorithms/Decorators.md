---
id: 01JBDGCYD1GK9S5FT9G33YM1SG
title: Decorators
tags:
  - python
  - programming
  - roadmaps
  - decorators
modified: 2024-10-30T18:14:38-04:00
---
# Decorators

Decorator is a design pattern in Python that allows a user to add new functionality to an existing object without modifying its structure. Decorators are usually called before the definition of a function you want to decorate.

Visit the following resources to learn more:

Free Resources

---

- [ArticleLearn Decorators in Python](https://pythonbasics.org/decorators/)
- [ArticlePython Decorators](https://www.datacamp.com/tutorial/decorators-python)
- [VideoDecorators in Python](https://www.youtube.com/watch?v=FXUUSfJO_J4)
- [VideoPython Decorators in 1 Minute](https://www.youtube.com/watch?v=BE-L7xu8pO4)
- [Primer on Python Decorators – Real Python](https://realpython.com/primer-on-python-decorators/)

## Primer on Python Decorators
- To understand decorators, we must first understand how functions work. Basically functions return a value based on a given set of arguments:
```python
>>> def add_one(number):
...     return number + 1
...

>>> add_one(2)
3
```
- To understand decorators, think of funcions **as tools that return values given a set of arguments**.
- ### First Class Objects
	- 