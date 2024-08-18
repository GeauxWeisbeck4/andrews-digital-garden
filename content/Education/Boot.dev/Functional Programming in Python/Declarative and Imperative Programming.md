---
id: 01J5HRR49809VXR1ZPM11GX6NP
modified: 2024-08-17T23:34:48-04:00
title: Declarative and Imperative Programming
description: What declarative and imperative programming are and how to do them in Python
tags:
  - declarative-programming
  - imperative-programming
  - functional-programming
  - python
---
# Declarative Programming

Functional programming aims to be _declarative_. We prefer to declare _what_ we want the computer to do, rather than muck around with the details of _how_ to do it.

Let's take an extreme example and pretend we wanted to style a webpage with [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) (Obviously a hypothetical because, well, why would anyone want to work on the frontend???)

## Declarative styling

The following CSS changes all [button](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button) elements to have red text:

```css
button {
    color: red;
}
```
It does _not_ execute line-by-line like an imperative language. Instead, it simply declares the desired style, and it's up to a web browser to figure out how to apply and display it.
## Imperative styling

Unlike functional programming (and CSS), a lot of code is _imperative_. We write out the exact step-by-step implementation details. This Python script draws a red button on a screen using the [Tkinter](https://docs.python.org/3/library/tkinter.html) library:

```py
from tkinter import * # first, import the library
master = Tk() # create a window
master.geometry("200x100") # set the window size
button = Button(master, text="Submit", fg="red").pack() # create a button
master.mainloop() # start the event loop
```
