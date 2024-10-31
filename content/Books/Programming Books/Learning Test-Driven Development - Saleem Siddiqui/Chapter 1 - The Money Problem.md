---
id: 01JBCGRBBDYCB2ST91N76RRPR6
title: Chapter 1 - The Money Problem
modified: 2024-10-29T12:23:47-04:00
tags:
  - test-driven-development
  - programming
  - books
  - javascript
  - go-lang
  - python
---
# Chapter 1. The Money Problem

> I would not give a fig for the simplicity this side of complexity, but I would give my life for the simplicity on the other side of complexity.
> 
> Oliver Wendell Holmes Jr.

Our development environment is ready. In this chapter, we’ll learn the three phases that support test-driven development. We’ll then write our first code feature using test-driven development.

# Red-Green-Refactor: The Building Blocks of TDD

Test-driven development follows a three-phase process:

1. _Red_. We write a failing test (including possible compilation failures). We run the test suite to verify the failing tests.
    
2. _Green_. We write just enough production code to make the test green. We run the test suite to verify this.
    
3. _Refactor_. We remove any code smells. These may be due to duplication, hardcoded values, or improper use of language idioms (e.g., using a verbose loop instead of a built-in iterator). If we break any tests during refactoring, we prioritize getting them back to green before exiting this phase.
    

This is the red-green-refactor (RGR) cycle, shown in [Figure 1-1](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#the_red_green_refactor_cycle_is). The three phases of this cycle are the essential building blocks of test-driven development. All the code we’ll develop in this book will follow this cycle.

![The Red Green Refactor (RGR) cycle. Red: write a failing test. Green: write just enough production code to pass the test. Refactor: remove code smells, e.g. duplication or poor design, without adding new features.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_0101.png)

###### Figure 1-1. The red-green-refactor cycle is the foundation on which test-driven development rests

###### Important

The three phases of the _red_-_green_-_refactor_ cycle are the essential building blocks of TDD.

##### RGR in Action

Throughout this book, we’ll use the three phases of the RGR cycle. We’ll stick to this regimen rather meticulously, so it’s important to start slowly and then speed up. In this chapter, the three phases are clearly marked in the relevant sections. In subsequent chapters, we’ll move briskly, and often seamlessly, from the red to the green phase. We’ll then redirect our attention to identify what to refactor. The transitions will get smoother as the pace of our development gets quicker. However, the three phases will always be there, and in that order.

# What’s the Problem?

We have a money problem. No, not the kind that almost _everyone_ has: not having enough of it! It’s more of a “we want to keep track of our money” problem.

Say we have to build a spreadsheet to manage money in more than one currency, perhaps to manage a stock portfolio.

|Stock|Stock exchange|Shares|Share price|Total|
|---|---|---|---|---|
|IBM|NASDAQ|100|124 USD|12400 USD|
|BMW|DAX|400|75 EUR|30000 EUR|
|Samsung|KSE|300|68000 KRW|20400000 KRW|

To build this spreadsheet, we’d need to do simple arithmetic operations on numbers in any one currency:

|   |
|---|
|5 USD × 2 = 10 USD|
|10 EUR × 2 = 20 EUR|
|4002 KRW / 4 = 1000.5 KRW|

We’d also like to convert between currencies. For example, if exchanging 1 EUR gets us 1.2 USD, and exchanging 1 USD gets us 1100 KRW:

|   |
|---|
|5 USD + 10 EUR = 17 USD|
|1 USD + 1100 KRW = 2200 KRW|

Each of the aforementioned line items will be one (teeny tiny) feature that we’ll implement using TDD. We already have several features to implement. In order to help us focus on one thing at a time, we’ll highlight the feature we’re working on **in bold**. When we’re done with a feature, we’ll signal our satisfaction by crossing it out.

So where should we start? In case the title of this book isn’t an obvious giveaway, we’ll start by writing a test.

# Our First Failing Test

Let’s start by implementing the very first feature in our list:

|   |
|---|
|**5 USD × 2 = 10 USD**|
|10 EUR × 2 = 20 EUR|
|4002 KRW / 4 = 1000.5 KRW|
|5 USD + 10 EUR = 17 USD|
|1 USD + 1100 KRW = 2200 KRW|

We’ll start by writing a failing test, corresponding to the _red_ phase of the RGR cycle.

## Go

In a new file called `money_test.go` in the `go` folder, let’s write our first test:

```
package
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-1)

Package declaration

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-2)

Imported “testing” package, used in `t.Errorf` later

[![3](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/3.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-3)

Our test method, which must start with `Test` and have one `*testing.T` argument

[![4](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/4.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-4)

Struct representing “USD 5.” `Dollar` does not exist yet

[![5](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/5.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-5)

Method under test: `Times`—which also does not exist yet

[![6](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/6.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-6)

Comparing actual value with expected value

[![7](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/7.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO1-7)

Ensuring test fails if expected value is not equal to actual value

This test function includes a bit of boilerplate code.

The `package main` declares that all ensuing code is part of the `main` package. This is a requirement for standalone executable Go programs. [Package management](https://oreil.ly/yvh3S) is a sophisticated feature in Go. It’s discussed in more detail in [Chapter 5](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch05.html#chapter_05).

Next, we import the `testing` package using the `import` statement. This package will be used in the unit test.

The unit test `func`tion is the bulk of the code. We declare an entity representing “5 USD.” This is the variable named `fiver`, which we initialize to a struct holding 5 in its `amount` field. Then, we multiply `fiver` by 2. And we expect the result to be 10 dollars, i.e., a variable `tenner` whose `amount` field must equal 10. If this isn’t the case, we print a nicely formatted error message with the actual value (whatever that may be).

When we run this test using “`go test -v .`” from the TDD Project Root folder, we should get an error:

```
...
```

We get the message loud and clear: that’s our first failing test!

###### Tip

“`go test -v .`” runs the tests in the current folder, and “`go test -v ./...`”[1](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#idm44839568418384) runs tests in the current folder and any subfolders. The `-v` switch produces verbose output.

## JavaScript

In a new file called `test_money.js` in the `js` folder, let’s write our first test:

```
const
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO2-1)

Importing the `assert` package, needed for the assertion later

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO2-2)

Object representing “USD 5.” `Dollar` does not exist yet

[![3](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/3.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO2-3)

Method under test: `times`—which also does not exist yet

[![4](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/4.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO2-4)

Comparing actual value with expected value in a `strictEqual` assert statement

JavaScript has minimal boilerplate code—the only line in addition to the test code is the `require` statement. This gives us access to the `assert` NPM package.

After that line are the three lines of code that form our test. We create an object representing 5 USD, we multiply it by 2, and we expect the result to be 10.

###### Important

ES2015 introduced the [`let`](https://oreil.ly/jBMPk) keyword for declaring variables and the [`const`](https://oreil.ly/GfYQ5) keyword to declare constants.

When we run this code from the TDD Project Root folder using `node js/test_money.js`, we should get an error that starts like this:

```
ReferenceError
```

That’s our first failing test. Hooray!

###### Tip

`node file.js` runs the JavaScript code in `file.js` and produces output. We use this command to run our tests.

## Python

In a new file called `test_money.py` in the `py` folder, let’s write our first test:

```
import
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-1)

Importing the `unittest` package, needed for the `TestCase` superclass.

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-2)

Our test class, which must subclass the `unittest.TestCase` class.

[![3](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/3.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-3)

Our method name must start with `test` to qualify as a test method.

[![4](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/4.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-4)

Object representing “USD 5.” `Dollar` does not exist yet.

[![5](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/5.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-5)

Method under test: `times`—which also does not exist yet.

[![6](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/6.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-6)

Comparing actual value with expected value in an `assertEqual` statement.

[![7](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/7.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO3-7)

The `main` idiom ensures this class can be run as a script.

Python requires `import`ing the `unittest` package, creating a class that subclasses `TestCase`, and `def`ining a function whose name starts with `test`. To be able to run the class as a standalone program, we need the [common Python idiom](https://docs.python.org/3/library/__main__.html) that runs the `unittest.main()` function when `test_money.py` is run directly.

The test function describes how we expect our code to work. We define a variable named `fiver` and initialize it to a desired (but yet-to-be-created) class `Dollar` with `5` as a constructor argument. We then multiply `fiver` by `2` and store the result in a variable `tenner`. Finally, we expect the `amount` in `tenner` to be `10`.

When we run this code from the `TDD_PROJECT_ROOT` folder using `python3 py/test_money.py -v`, we get an error:

```
NameError
```

That’s our first failing test. Hooray!

###### Tip

`python3 file.py -v` runs the Python code in `file.py` and produces verbose output. We use this command to run our tests.

# Going for Green

We wrote our tests as we would expect them to work, blithely ignoring all syntax errors for the moment. Is this smart?

In the _very_ beginning—which is where we are—it is smart to start with the smallest bit of code that sets us on the path to progress. Of course our tests fail because we haven’t defined what `Dollar` is. This _may_ seem the perfect time to say “Duh!” However, a wee bit of patience is warranted for these two reasons:

1. We have just finished the _first_ step—getting to _red_—of our _first_ test. Not only is this the beginning, it’s the very beginning of the beginning.
    
2. We can (and will) speed up the increments as we go along. However, it’s important to know that we can slow down when we need to.
    

The next phase in the RGR cycle is to get to _green_.

It’s clear we need to introduce an abstraction `Dollar`. This section defines how to introduce this, and other abstractions, to get our test to pass.

## Go

Add an empty `Dollar struct` to the end of `money_test.go`.

```
type
```

When we run the test now, we get a new error:

```
...
```

Progress!

The error message is directing us to introduce a field named `amount` in our `Dollar` struct. So let’s do this, using an `int` data type for now (which is sufficient for our goal):

```
type
```

Adding the `Dollar struct`, rather predictably, gets us to the next error:

```
...
```

We see a pattern here: when there is something (a field or method) that’s undefined, we get this `undefined` error from the Go runtime. We will use this information to speed up our TDD cycles in the future. For now, let’s add a `func`tion named `Times`. We know, from how we wrote our test, that this function needs to take a number (the multiplier) and return another number (the result).

But how should we calculate the result? We know basic arithmetic: how to multiply two numbers. But if we were to write the simplest code that works, we’d be justified in _always_ returning the result that our test expects, that is, a struct representing 10 dollars:

```
func
```

When we run our code now, we should get a short and sweet response on our terminal:

```
==
```

That’s the magic word: we made our test `PASS`!

## JavaScript

In `test_money.js`, right after the `const assert = require('assert');` line, define an empty class named `Dollar`:

```
class
```

When we run the `test_money.js` file now, we get an error:

```
TypeError
```

Progress! The error clearly states that there is no function named `times` defined for the object named `fiver`. So let’s introduce it inside the `Dollar` class:

```
class
```

Running the test now produces a new error:

```
TypeError
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO4-1)

This message is from Node.js v16; v14 produces a slightly different error message

Our test expects an object with a property `amount`. Since we’re not returning anything from our `times` method, the return value is `undefined`, which does not have an `amount` property (or any other property, for that matter).

###### Tip

In the JavaScript language, functions and methods do not explicitly declare any return types. If we examine the result of a function that returns nothing, we’ll find the return value is `undefined`.

So how should we make our test go green? What’s the simplest thing that could work? How about if we _always_ create an object representing 10 USD and return it?

Let’s try it out. We add a `constructor` that initializes objects to a given amount and a `times` method that obstinately creates and returns “10 USD” objects:

```
class
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO5-1)

The `constructor` function is called whenever a `Dollar` object is created.

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO5-2)

Initialize the `this.amount` variable to the given parameter.

[![3](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/3.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO5-3)

The `times` method takes a parameter.

[![4](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/4.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO5-4)

Simple implementation: always return 10 dollars.

When we run our code now, we should get no errors. This is our first green test!

###### Important

Because `strictEqual` and other methods in the `assert` package only produce output when the assertions _fail_, a successful test run will be quite silent with no output. We’ll improve this behavior in [Chapter 6](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch06.html#chapter_06).

## Python

Since `'Dollar' is not defined`, let’s define it in `test_money.py` before our `TestMoney` class:

```
class
```

When we run our code now, we get an error:

```
TypeError
```

Progress! The error is clearly telling us that there is currently no way to initialize `Dollar` objects with any arguments, such as the `5` and `10` we have in our code. So let’s fix this by providing the briefest possible initializer:

```
class
```

Now the error message from our test changes:

```
AttributeError
```

We see a pattern here: our test is still failing, but for _slightly different_ reasons each time. As we define our abstractions—first `Dollar` and then an `amount` field—the error messages “improve” to the next stage. This is a hallmark of TDD: steady progress at a pace we control.

Let’s speed things up a bit by defining a `times` function _and_ giving it the minimum behavior to get to green. What’s the minimum behavior necessary? Always returning a “ten dollar” object that’s required by our test, of course!

```
class
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO6-1)

The `__init__` function is called whenever a `Dollar` object is created.

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO6-2)

Initialize the `self.amount` variable to the given parameter.

[![3](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/3.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO6-3)

The `times` method takes a parameter.

[![4](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/4.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO6-4)

Simple implementation entails always returning 10 dollars.

When we run our test now, we get a short and sweet response:

```
Ran
```

It’s possible that the test may not run in `0.000s`, but let’s not lose sight of the magic word `OK`. This is our first green test!

# Cleaning Up

Do you feel bemused that we got to green by hardcoding “10 USD” in our tests? Fret not: the refactoring stage allows us to address this discomfort by teasing out _how_ we can remove the hardcoded and duplicated value of “10 USD.”

_Refactor_ is the third and final stage of the RGR cycle. We may not have many lines of code at this point; however, it’s still important to keep things tidy and compact. If we have any formatting clutter or commented-out lines of code, now is the time to clean it up.

More significant is the need to remove duplication and make code readable. At first blush, it may seem that in the 20 or so lines of code we’ve written, there can’t be any duplication. However, there is already a subtle yet significant bit of duplication.

We can find this duplication by noticing a couple of quirks within our code:

1. We have written just enough code to verify that “doubling 5 dollars should give us 10 dollars.” If we decide to change our existing test to say “doubling 10 dollars should give us 20 dollars”—an equally sensible statement—we will have to change _both_ our test and our `Dollar` code. There is a dependency, a _logical coupling_, between the two segments of code. In general, coupling of this kind should be avoided.
    
2. In both our test and our code, we had the magic number `10`. Where did we come up with that? We obviously did the math in our heads. We realize that doubling 5 dollars should give us 10 dollars. So we wrote `10` in both our test and in our `Dollar` code. We should realize that the `10` in the `Dollar` entity is really `5 * 2`. This realization would allow us to remove this duplication.
    

Duplicated code is often the symptom of some underlying problem: a missing code abstraction or bad coupling between different parts of the code.[2](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#idm44839565740576)

Let’s remove the duplication and thereby get rid of the coupling as well.

## Go

Replace the `10` in the `Times` function by its equivalent `5 * 2`:

```
func
```

The test should still be green.

Writing it this way makes us realize the missing abstraction. The hardcoded `5` is really `d.amount`, and the `2` is the `multiplier`. Replacing these hardcoded numbers with the correct variables gives us the non-trivial implementation:

```
func
```

Yay! The test still passes, and we have removed the duplication and the coupling.

There is one final bit of cleanup.

In our test, we explicitly used the field name `amount` when initializing a `Dollar` struct. It’s also possible to omit field names when initializing a struct, as we did in our `Times` method.[3](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#idm44839565668576) Either style—using explicit names or not using them—works. However, it’s important to be consistent. Let’s change the `Times` function to specify the field name:

```
func
```

###### Tip

Remember to run `go fmt ./...` periodically to fix any formatting issues in code.

## JavaScript

Let’s replace the `10` in the `times` method by its equivalent `5 * 2`:

    `times``(``multiplier``)` `{`
        `return` `new` `Dollar``(``5` `*` `2``);`
    `}`

The test should still be green.

The missing abstraction is now clear. We can replace `5` with `this.amount` and `2` with `multiplier`:

    `times``(``multiplier``)` `{`
        `return` `new` `Dollar``(``this``.``amount` `*` `multiplier``);`
    `}`

Yay! The test is still green, and we have eliminated both the duplicated `10` and the coupling.

## Python

Let’s replace the `10` in the `times` method by its equivalent `5 * 2`:

  `def` `times``(``self``,` `multiplier``):`
    `return` `Dollar``(``5` `*` `2``)`

The test stays green, as expected.

This reveals the underlying abstraction. The `5` is really `self.amount` and the `2` is the `multiplier`:

  `def` `times``(``self``,` `multiplier``):`
    `return` `Dollar``(``self``.``amount` `*` `multiplier``)`

Hooray! The test remains green, and the duplication and the coupling are gone.

# Committing Our Changes

We have finished our first feature using TDD. Lest we forget, it’s important to commit our code to our version control repository at frequent intervals.

A green test is an excellent place to commit code.

In a shell window, let’s type these two commands:

```
git
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO7-1)

Add all files, including all changes in them, to the Git index.

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO7-2)

Commit the Git index to the repository with the given message.

Assuming code for all three languages exists in the correct folders, we should get a message like this.

```
[
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO8-1)

The hex number, `bb31b94`, represents the first several digits of the unique “SHA hash” associated with the commit. It will be different for every person (and every commit).

This indicates that all our files are safely in our Git version control repository. We can verify this by executing the `git log` command on our shell, which should produce output similar to the following:

```
commit
```

[![1](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/1.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO9-1)

This is the first commit, with its full SHA hash.

[![2](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/2.png)](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#co_the_money_problem_CO9-2)

This is the message we typed for our first commit.

It’s important to realize that the Git repository to which we have committed our code also resides on our local filesystem. (It’s inside the `.git` folder under our `TDD_PROJECT_ROOT`). While this doesn’t save us from accidental coffee spills on our computer (always use lids), it does provide assurance that we can go back to a previous known good version if we get tangled up somewhere. In [Chapter 13](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch13.html#chapter_13), we’ll push all our code to a GitHub repository.

We’ll use this strategy of committing our code to our local Git repository in each chapter, using the same set of commands.

###### Important

We’ll use the two commands `git add .` and `git commit -m _commit message_` to frequently commit our code in each chapter.

The only thing that’ll vary is the commit message, which will follow the semantic commit style and include a short, one-line description of the changes.

###### Tip

The `git commit` messages in this book follow the [semantic commit style](https://oreil.ly/MhE1b).

# Where We Are

This chapter introduced test-driven development by showing the _very first_ red-green-refactor cycle. With our first tiny feature successfully implemented, let’s cross it off. Here’s where we are in our feature list:

|   |
|---|
|5 USD × 2 = 10 USD|
|10 EUR × 2 = 20 EUR|
|4002 KRW / 4 = 1000.5 KRW|
|5 USD + 10 EUR = 17 USD|
|1 USD + 1100 KRW = 2200 KRW|

Let’s take a moment to review and savor our code before we move on to the next challenge. The source code for all three languages is reproduced below. It’s also accessible in the GitHub repository. For the sake of brevity, future chapters will list the corresponding branch name only.

## Go

Here’s how the file `money_test.go` looks right now:

```
package
```

## JavaScript

Here’s how the `test_money.js` file looks at this point:

```
const
```

## Python

Here’s how the `test_money.py` file looks right now:

```
import
```

###### Tip

The code for this chapter is in a [branch named “chap01” in the GitHub repository](https://github.com/saleem/tdd-book-code/tree/chap01). There is a branch for each chapter in which code is developed.

In [Chapter 2](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch02.html#chapter_02), we’ll speed things up by building out a couple more features.

[1](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#idm44839568418384-marker) The three dots in “`go test -v ./...`” and “`go fmt ./...`” are to be typed literally; these are the only instances in this book where they do _not_ stand for omitted code!

[2](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#idm44839565740576-marker) Kent Beck’s opinion is worth quoting here: “If dependency is the problem, duplication is the symptom.”

[3](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#idm44839565668576-marker) If there are multiple fields in the struct—which currently there are not—then _either_ the order of the fields must be the same in both struct definition and initialization _or_ field names must be specified during struct initialization. See [_https://gobyexample.com/structs_](https://gobyexample.com/structs).