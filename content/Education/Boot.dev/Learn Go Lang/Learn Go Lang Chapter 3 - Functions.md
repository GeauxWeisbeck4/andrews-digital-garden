---
id: 01J6DGCY5N9TFS7ZPX40YJRZ22
title: Learn Go Lang  Chapter 3 - Functions
modified: 2024-08-28T19:39:38-04:00
description: Learning how to write functions in Go Lang
tags:
  - go-lang
  - programming
  - boot-dev
---
# Functions

Functions in Go can take zero or more arguments.

To make code easier to read, the variable type comes _after_ the variable name.

For example, the following function:

```go
func sub(x int, y int) int {
  return x-y
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Accepts two integer parameters and returns another integer.

Here, `func sub(x int, y int) int` is known as the "function signature".

## Assignment

We often will need to manipulate strings in our messaging app. For example, adding some personalization by using a customer's name within a template. The `concat` function should take two strings and smash them together.

- `hello` + `world` = `helloworld`

Fix the [function signature](https://www.devx.com/open-source-zone/programming-basics-the-function-signature/#:~:text=A%20function%20signature%20includes%20the%20function%20name%2C%20its%20arguments%2C%20and%20in%20some%20languages%2C%20the%20return%20type.) of `concat` to reflect its behavior.

# Multiple Parameters

When multiple arguments are of the same type, and are next to each other in the function signature, the type only needs to be declared after the last argument.

Here are some examples:

```go
func addToDatabase(hp, damage int) {
  // ...
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

```go
func addToDatabase(hp, damage int, name string) {
  // ?
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

```go
func addToDatabase(hp, damage int, name string, level int) {
  // ?
}
```

# Unit Test Lessons

Up until now, all the coding lessons in this course have been testing you based on your code's _console output_ (what's printed). For example, a lesson might expect your code (in conjunction with the code we provide) to `print` something like:

```bash
Price: 0.2
NumMessages: 18
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

If your code prints that _exact_ output, you pass. If it doesn't, you fail.

## A new type of lesson

Going forward, you'll also encounter a new type of lesson: [unit tests](https://en.wikipedia.org/wiki/Unit_testing) If this isn't your first course with us, you'll know what we are referring to, but incase you haven't. A unit test is just an automated program that tests a small "unit" of code. Usually just a function or two. The editor will have tabs: the "main.go" file containing your code, and the "main_test.go" file containing the unit tests.

These new unit-test-style lessons will test your code's _functionality_ rather than its output. Our tests will call functions in your code with different arguments, and expect specific `return` values. If your code returns the correct values, you pass. If it doesn't, you fail.

There are two reasons for this change:

1. It's more realistic. In the real world, you'll be writing unit tests and running them against your code to make sure it works as expected.
2. You can run and debug your code with `fmt.Println` statements, and leave those print statements in when you submit. Unlike the output-based lessons, you won't have to remove your `fmt.Println` statements to pass.

## Assignment

Complete the `getMonthlyPrice` function. It accepts a `tier` (string) as input and returns the monthly price for that tier in pennies. Here are the prices in dollars:

- "basic" - $100.00
- "premium" - $150.00
- "enterprise" - $500.00

Convert the prices from dollars to pennies. If the given tier doesn't match any of the above, return 0 pennies.

_Note: a dollar consists of 100 pennies._

## Answer

```go
package main

func getMonthlyPrice(tier string) int {
	if tier == "basic" {
		return 10000
	} else if tier == "premium" {
		return 15000
	} else if tier == "enterprise" {
		return 50000
	} else {
		return 0
	}
}

```
# Declaration Syntax

Developers often wonder why the declaration syntax in Go is different from the tradition established in the C family of languages.

## C-Style syntax

The C language describes types with an expression including the name to be declared, and states what type that expression will have.

```c
int y;
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

The code above declares `y` as an `int`. In general, the type goes on the left and the expression on the right.

Interestingly, the creators of the Go language agreed that the C-style of declaring types in signatures gets confusing really fast - take a look at this nightmare.

```c
int (*fp)(int (*ff)(int x, int y), int b)
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Go-style syntax

Go's declarations are clear, you just read them left to right, just like you would in English.

```go
x int
p *int
a [3]int
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

It's nice for more complex signatures, it makes them easier to read.

```go
f func(func(int,int) int, int) int
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Reference

The [following post on the Go blog](https://blog.golang.org/declaration-syntax) is a great resource for further reading on declaration syntax.