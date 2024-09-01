---
id: 01J6DGCY5N9TFS7ZPX40YJRZ22
title: Learn Go Lang  Chapter 3 - Functions
modified: 2024-08-30T17:31:56-04:00
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

# Passing Variables by Value

Variables in Go are passed by value (except for a few data types we haven't covered yet). "Pass by value" means that when a variable is passed into a function, that function receives a _copy_ of the variable. The function is unable to mutate the caller's original data.

```go
func main() {
    x := 5
    increment(x)

    fmt.Println(x)
    // still prints 5,
    // because the increment function
    // received a copy of x
}

func increment(x int) {
    x++
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

Fix the bugs in the `monthlyBillIncrease` and `getBillForMonth` functions.

- `monthlyBillIncrease`: Returns the increase in the bill from the previous to the current month. If the bill decreased, it should return a negative number.
- `getBillForMonth`: Returns the bill for the given month.

It looks like whoever wrote the `getBillForMonth` function thought that they could pass in the `bill` parameter, update it inside the function, and that update would apply in the parent function (`monthlyBillIncrease`). _They were wrong_.

Change the `getBillForMonth` function to explicitly _return_ the bill for the given month, and be sure to capture that return value properly in the `monthlyBillIncrease` function.

The function signature for `getBillForMonth` should only take 2 parameters once you're done.

## Answer

```go
package main

func monthlyBillIncrease(costPerSend, numLastMonth, numThisMonth int) int {
	lastMonthBill := getBillForMonth(costPerSend, numLastMonth)
	thisMonthBill := getBillForMonth(costPerSend, numThisMonth)
	return thisMonthBill - lastMonthBill
}

func getBillForMonth(costPerSend, messagesSent int) int {
	return costPerSend * messagesSent
}
```
# Ignoring Return Values

A function can return a value that the caller doesn't care about. We can explicitly ignore variables by using an underscore, or more precisely, the [blank identifier `_`](https://go.dev/doc/effective_go#blank).

For example:

```go
func getPoint() (x int, y int) {
  return 3, 4
}

// ignore y value
x, _ := getPoint()
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Even though `getPoint()` returns two values, we can capture the first one and ignore the second. In Go, the blank identifier isn't just a convention; it's a real language feature that completely discards the value.

## Why might you ignore a return value?

Maybe a function called `getCircle` returns the center point and the radius, but you only need the radius for your calculation. In that case, you can ignore the center point variable.

The Go compiler will **throw an error** if you have any unused variable declarations in your code, so you _need_ to ignore anything you don't intend to use.

## Assignment

Run the code as-is. You should get a compiler error. Fix the `getProductMessage` to ignore the unused return value.

## Answer
```go
package main

func getProductMessage(tier string) string {
	quantityMsg, priceMsg := getProductInfo(tier)
	return "You get " + quantityMsg + " for " + priceMsg + "."
}

// don't touch below this line

func getProductInfo(tier string) (string, string) {
	if tier == "basic" {
		return "1,000 texts per month", "$30 per month" 
	} else if tier == "premium" {
		return "50,000 texts per month", "$60 per month"
	} else if tier == "enterprise" {
		return "unlimited texts per month", "$100 per month"
	} else {
		return "", ""
	}
}
```
# Named Return Values

Return values may be given names, and if they are, then they are treated the same as if they were new variables defined at the top of the function.

Named return values are best thought of as a way to document the purpose of the returned values.

According to the [tour of go](https://tour.golang.org/):

> A return statement without arguments returns the named return values. This is known as a "naked" return. Naked return statements should be used only in short functions. They can harm readability in longer functions.

Use naked returns if it's obvious what the purpose of the returned values is. Otherwise, use named returns for clarity.

```go
func getCoords() (x, y int){
  // x and y are initialized with zero values

  return // automatically returns x and y
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Is the same as:

```go
func getCoords() (int, int){
  var x int
  var y int
  return x, y
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

In the first example, `x` and `y` are the return values. At the end of the function, we could simply write `return` to return the values of those two variables, rather than writing `return x,y`.

## Assignment

One of our clients likes us to send text messages reminding users of life events coming up.

Fix the bug by using named return values in the function signature. The variables need to be automatically initialized. Order them as they appear in the code. _Do not alter the body of the function_.

## Answer

```go
package main

func yearsUntilEvents(age int) (yearsUntilAdult int, yearsUntilDrinking int, yearsUntilCarRental int) {
	yearsUntilAdult = 18 - age
	if yearsUntilAdult < 0 {
		yearsUntilAdult = 0
	}
	yearsUntilDrinking = 21 - age
	if yearsUntilDrinking < 0 {
		yearsUntilDrinking = 0
	}
	yearsUntilCarRental = 25 - age
	if yearsUntilCarRental < 0 {
		yearsUntilCarRental = 0
	}
	return
}
```
# The Benefits of Named Returns

## Good For Documentation (Understanding)

Named return parameters are great for documenting a function. We know what the function is returning directly from its signature, no need for a comment.

Named return parameters are particularly important in longer functions with many return values.

```go
func calculator(a, b int) (mul, div int, err error) {
    if b == 0 {
      return 0, 0, errors.New("Can't divide by zero")
    }
    mul = a * b
    div = a / b
    return mul, div, nil
}
```

Which is easier to understand than:

```go
func calculator(a, b int) (int, int, error) {
    if b == 0 {
      return 0, 0, errors.New("Can't divide by zero")
    }
    mul := a * b
    div := a / b
    return mul, div, nil
}
```

We know _the meaning_ of each return value just by looking at the function signature: `func calculator(a, b int) (mul, div int, err error)`

_Note: `nil` is the zero value of an error. More on this later._

## Less Code (Sometimes)

If there are multiple return statements in a function, you don’t need to write all the return values each time, though you _probably_ should.

When you choose to omit return values, it's called a _naked_ return. Naked returns should only be used in short and simple functions.

## Answer

```go
package main

func yearsUntilEvents(age int) (yearsUntilAdult, yearsUntilDrinking, yearsUntilCarRental int) {
	yearsUntilAdult = 18 - age
	if yearsUntilAdult < 0 {
		yearsUntilAdult = 0
	}
	yearsUntilDrinking = 21 - age
	if yearsUntilDrinking < 0 {
		yearsUntilDrinking = 0
	}
	yearsUntilCarRental = 25 - age
	if yearsUntilCarRental < 0 {
		yearsUntilCarRental = 0
	}
	return yearsUntilAdult, yearsUntilDrinking, yearsUntilCarRental
}

```
# Early Returns

Go supports the ability to return early from a function. This is a powerful feature that can clean up code, especially when used as guard clauses.

Guard Clauses leverage the ability to `return` early from a function (or `continue` through a loop) to make nested conditionals one-dimensional. Instead of using if/else chains, we just return early from the function at the end of each conditional block.

```go
func divide(dividend, divisor int) (int, error) {
	if divisor == 0 {
		return 0, errors.New("Can't divide by zero")
	}
	return dividend/divisor, nil
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Error handling in Go naturally encourages developers to make use of guard clauses. When I started writing more JavaScript, I was disappointed to see how many nested conditionals existed in the code I was working on.

Let’s take a look at an exaggerated example of nested conditional logic:

```go
func getInsuranceAmount(status insuranceStatus) int {
  amount := 0
  if !status.hasInsurance(){
    amount = 1
  } else {
    if status.isTotaled(){
      amount = 10000
    } else {
      if status.isDented(){
        amount = 160
        if status.isBigDent(){
          amount = 270
        }
      } else {
        amount = 0
      }
    }
  }
  return amount
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

This could be written with guard clauses instead:

```go
func getInsuranceAmount(status insuranceStatus) int {
  if !status.hasInsurance(){
    return 1
  }
  if status.isTotaled(){
    return 10000
  }
  if !status.isDented(){
    return 0
  }
  if status.isBigDent(){
    return 270
  }
  return 160
}
```

The example above is _much_ easier to read and understand. When writing code, it’s important to try to reduce the cognitive load on the reader by reducing the number of entities they need to think about at any given time.

In the first example, if the developer is trying to figure out when `270` is returned, they need to think about each branch in the logic tree and try to remember which cases matter and which cases don’t. With the one-dimensional structure offered by guard clauses, it’s as simple as stepping through each case in order.

# Functions as values

Go supports [first-class](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function) and higher-order functions, which are just fancy ways of saying "functions as values". Functions are just another type -- like `int`s and `string`s and `bool`s.

To dynamically accept a function as a parameter to another function:

```go
func add(x, y int) int {
	return x + y
}

func mul(x, y int) int {
	return x * y
}

// aggregate applies the given math function
// called "arithmetic" to all three integers
// instead of just the two it was capable of before
func aggregate(a, b, c int, arithmetic func(int, int) int) int {
  firstSum := arithmetic(a, b)
  secondSum := arithmetic(firstSum, c)
  return secondSum
}

func main() {
	sum := aggregate(2, 3, 4, add)
	// sum is 9
	product := aggregate(2, 3, 4, mul)
	// product is 24
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

Complete the `reformat` function. It takes a `message` string and a `formatter` function as input:

1. Apply the given `formatter` _three times_ to the `message`
2. Add a prefix of `TEXTIO:` to the result
3. Return the final string

For example, if the `message` is "General Kenobi" and the `formatter` adds a period to the end of the string, the final result should be

```
TEXTIO: General Kenobi...
```

## Answer
```go
package main

func reformat(message string, formatter func(string) string) string {
	once := formatter(message)
	twice := formatter(once)
	thrice := formatter(twice)
	prefix := "TEXTIO: "
	return prefix + thrice
}

```