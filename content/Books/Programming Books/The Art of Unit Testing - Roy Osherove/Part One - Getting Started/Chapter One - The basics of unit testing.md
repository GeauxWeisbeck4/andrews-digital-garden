---
id: 01JH65D88VZHFH2EHSX4ZCVNYD
modified: 2025-01-09T13:32:23-05:00
---
# 1 The basics of unit testing

This chapter covers

- Identifying entry points and exit points
- The definitions of _unit test_ and _unit of work_
- The difference between unit testing and integration testing
- A simple example of unit testing
- Understanding test-driven development

Manual tests suck. You write your code, you run it in the debugger, you hit all the right keys in your app to get things just right, and then you repeat all this the next time you write new code. And you have to remember to check all the other code that might have been affected by the new code. More manual work. Great.

Doing tests and regression testing completely manually, repeating the same actions again and again like a monkey, is error prone and time consuming, and people seem to hate doing that as much as anything can be hated in software development. These problems are alleviated by tooling and our decision to use it for good, by writing automated tests that save us precious time and debugging pain. Integration and unit testing frameworks help developers write tests more quickly with a set of known APIs, execute those tests automatically, and review the results of those tests easily. And they never forget! I’m assuming you’re reading this book because either you feel the same way, or because someone forced you to read it, and that someone feels the same way. Or maybe that someone was forced to force you into reading this book. Doesn’t matter. If you believe repetitive manual testing is awesome, this book will be very difficult to read. The assumption is that you _want_ to learn how to write good unit tests.

This book also assumes that you know how to write code using JavaScript or TypeScript, using at least ECMAScript 6 (ES6) features, and that you are comfortable with node package manager (npm). Another assumption is that you are familiar with Git source control. If you’ve seen github.com before and you know how to clone a repository from there, you are good to go.

Although all the book’s code listings are in JavaScript and TypeScript, you don’t have to be a JavaScript programmer to read this book. The previous editions of this book were in C#, and I’ve found that about 80% of the patterns there have transferred over quite easily. You should be able to read this book even if you come from Java, .NET, Python, Ruby, or other languages. The patterns are just patterns. The language is used to demonstrate those patterns, but they are not language-specific.

JavaScript vs. TypeScript in this book

This book contains both vanilla JavaScript and TypeScript examples throughout. I take full responsibility for creating such a Tower of Babel (no pun intended), but I promise, there’s a good reason: this book is dealing with three programming paradigms in JavaScript: _procedural_, _functional,_ and _object-oriented_ design.

I use regular JavaScript for the samples dealing with procedural and functional designs. I use TypeScript for the object-oriented examples, because it provides the structure needed to express these ideas.

In previous editions of this book, when I was working in C#, this wasn’t an issue. When moving to JavaScript, which supports these multiple paradigms, using TypeScript makes sense.

Why not just use TypeScript for all the paradigms, you ask? Both to show that TypeScript is not needed to write unit tests and that the concepts of unit testing do not depend on one language or another, or on any type of compiler or linter, to work.

This means that if you’re into functional programming, some of the examples in this book will make sense, and others will seem like they are overcomplicated or needlessly verbose. Feel free to focus only on the functional examples.

If you’re into object-oriented programming or are coming from a C#/Java background, you’ll find that some of the non-object-oriented examples are simplistic and don’t represent your day-to-day work in your own projects. Fear not, there will be plenty of sections relating to the object-oriented style.

## 1.1 The first step

There’s always a first step: the first time you wrote a program, the first time you failed a project, and the first time you succeeded in what you were trying to accomplish. You never forget your first time, and I hope you won’t forget your first tests.

You may have come across tests in some form. Some of your favorite open source projects come with bundled “test” folders—you have them in your own projects at work. You might have already written a few tests yourself, and you may even remember them as being bad, awkward, slow, or unmaintainable. Even worse, you might have felt they were useless and a waste of time. (Many people sadly do.) Or you may have had a great first experience with unit tests, and you’re reading this book to see what more you might be missing.

This chapter will analyze the “classic” definition of a unit test and compare it to the concept of integration testing. This distinction is confusing to many, but it’s very important to learn, because, as you’ll learn later in the book, separating unit tests from other types of tests can be crucial to having high confidence in your tests when they fail or pass.

We’ll also discuss the pros and cons of unit testing versus integration testing, and we’ll develop a better definition of what might be a “good” unit test. We’ll finish with a look at test-driven development (TDD), because it’s often associated with unit testing but is a separate skill that I highly recommend giving a chance (it’s not the main topic of this book, though). Throughout this chapter, I’ll also touch on concepts that are explained more thoroughly elsewhere in the book.

First, let’s define what a unit test should be.

## 1.2 Defining unit testing, step by step

Unit testing isn’t a new concept in software development. It’s been floating around since the early days of the Smalltalk programming language in the 1970s, and it proves itself time and time again as one of the best ways a developer can improve code quality while gaining a deeper understanding of the functional requirements of a module, class, or function. Kent Beck introduced the concept of unit testing in Smalltalk, and it has carried on into many other programming languages, making unit testing an extremely useful practice.

To see what we _don’t_ want to use as our definition of unit testing, let’s look to Wikipedia as a starting point. I’ll use its definition with reservations, because, in my opinion, there are many important parts missing, but it is largely accepted by many for lack of other good definitions. Our definition will slowly evolve throughout this chapter, with the final definition appearing in section 1.9.

Unit tests are typically automated tests written and run by software developers to ensure that a section of an application (known as the “unit”) meets its design and behaves as intended. In procedural programming, a unit could be an entire module, but it is more commonly an individual function or procedure. In object-oriented programming, a unit is often an entire interface, such as a class, or an individual method ([https://en.wikipedia.org/wiki/Unit_testing](https://en.wikipedia.org/wiki/Unit_testing)).

The thing you’ll write tests for is the _subject_, _system, or suite under test_ (SUT).

DEFINITION SUT stands for _subject_, _system, or suite under test_, and some people like to use CUT (_component, class, or code under test_). When you test something, you refer to the thing you’re testing as the SUT.

Let’s talk about the word “unit” in unit testing. To me, _unit_ stands for a “unit of work” or a “use case” inside the system. A unit of work has a beginning and an end, which I call an _entry point_ and an _exit point_. A simple example of a unit of work is a function that calculates something and returns a value. However, a function could also use other functions, other modules, and other components in the calculation process, which means the unit of work (from entry point to exit point), could span more than just a function.

Unit of work

A _unit of work_ is all the actions that take place between the invocation of an _entry point_ up until a noticeable end result through one or more _exit points_. The _entry point_ is the thing we trigger. Given a publicly visible function, for example

1. The function’s body is all or part of the unit of work.
    
2. The function’s declaration and signature are the entry point into the body.
    
3. The resulting outputs or behaviors of the function are its exit points.
    

## 1.3 Entry points and exit points

A unit of work always has an entry point and one or more exit points. Figure 1.1 shows a simple diagram of a unit of work.

![01-01](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-01.png)  

Figure 1.1 A unit of work has entry points and exit points.

A unit of work can be a single function, multiple functions, or even multiple modules or components. But it always has an entry point that we can trigger from the outside (via tests or other production code), and it always ends up doing something useful. If it doesn’t do anything useful, we might as well remove it from our codebase.

What’s _useful_? Something publicly noticeable that happens in the code: a return value, a state change, or calling an external party, as shown in figure 1.2. Those noticeable behaviors are what I call _exit points._

![01-02](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-02.png)  

Figure 1.2 Types of exit points

Why “exit point”?

Why use the term “exit point” and not something like “behavior”? My thinking is that behaviors can be purely internal, whereas we’re looking for externally visible behaviors from the caller. That difference might be difficult to distinguish at a glance. Also, “exit point” nicely suggests we are leaving the context of a unit of work and going back to the test context, though behaviors might be a bit more fluid than that. There’s an extensive discussion about types of behavior, including observable behavior, in _Unit Testing Principles, Practices, and Patterns_ by Vladimir Khorikov (Manning, 2020). Refer to that book to learn more about this topic.

The following listing shows a quick code example of a simple unit of work.

Listing 1.1 A simple function that we’d like to test

const sum = (numbers) => {
  const [a, b] = numbers.split(',');
  const result = parseInt(a) + parseInt(b);
  return result;
};

About the JavaScript version used in this book

I’ve chosen to use Node.js 12.8 with plain ES6 JavaScript along with JSDoc-style comments. The module system I’ll use is CommonJS, to keep things simple. Perhaps in a future edition I’ll start using ES modules (.mjs files), but for now, and for the rest of this book, CommonJS will do. It doesn’t really matter for the patterns in this book anyway.

You should be able to easily extrapolate the techniques used here for whatever JavaScript stack you’re currently working with, whether you’re using TypeScript, Plain JS, ES modules, backend or frontend, Angular, or React. It shouldn’t matter.

Getting the code for this chapter

You can download all the code samples shown in this book from GitHub. You can find the repository at [https://github.com/royosherove/aout3-samples](https://github.com/royosherove/aout3-samples). Make sure you have Node 12.8 or higher installed, and run `npm` `install` followed by `npm` `run` `ch[chapter` `number]`. For this chapter, you would run `npm` `run` `ch1`. This will run all the tests for this chapter so you can see their outputs.

This unit of work is completely encompassed in a single function. The function is the entry point, and because its end result returns a value, it also acts as the exit point. We get the end result in the same place we trigger the unit of work, so the entry point is also the exit point.

If we drew this function as a unit of work, it would look something like figure 1.3. I used `sum(numbers)` as the entry point, not `numbers`, because the entry point is the function signature. The parameters are the context or input given through the entry point.

 

![01-03](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-03.png)  

Figure 1.3 A function that has the same entry point as exit point

The following listing shows a variation on this idea.

Listing 1.2 A unit of work with entry points and exit points

**let total = 0;**
 
**const totalSoFar = () => {**
  **return total;**
**};**
 
const sum = (numbers) => {
  const [a, b] = numbers.split(',');
  const result = parseInt(a) + parseInt(b);
  **total += result;**          ❶
  return result;
};

❶ New functionality: calculating a running total

This new version of `sum` has _two_ exit points. It does two things:

- It returns a value.
    
- It introduces new functionality: a running total of all the sums. It sets the state of the module in a way that is noticeable (via `totalSoFar`) from the caller of the entry point.
    

Figure 1.4 shows how I would draw this unit of work. You can think of these two exit points as two different paths, or requirements, from the same unit of work, because they indeed _are_ two different useful things the code is expected to do. This also means I’d be very likely to write two different unit tests here: one for each exit point. Very soon we’ll do exactly that.

![01-04](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-04.png)  

Figure 1.4 A unit of work with two exit points

What about `totalSoFar`? Is this also an entry point? Yes, it could be, _in a separate test_. I could write a test that proves that calling `totalSoFar` without triggering prior to that call returns `0`. That would make it its own little unit of work, which would be perfectly fine. Often one unit of work (such as `sum`) can be composed of smaller units.

As you can see, the scope of our tests can change and mutate, but we can still define them with entry points and exit points. Entry points are always where the test triggers the unit of work. You can have multiple entry points into a unit of work, each used by a different set of tests.

A note on design

There are two main types of actions: “query” actions and “command” actions. Query actions don’t change stuff; they just return values. Command actions change stuff but don’t return values.

We often combine the two, but there are many cases where separating them might be a better design choice. This book isn’t primarily about design, but I urge you to read more about the concept of _command query separation_ over on Martin Fowler’s website: [https://martinfowler.com/bliki/CommandQuerySeparation.html](https://martinfowler.com/bliki/CommandQuerySeparation.html).

Exit points signifying requirements and new tests, and vice versa

Exit points are end results of a unit of work. For unit tests, I usually write at least one test, with its own readable name, for each exit point. I may then add more tests with variations on the inputs, all using the same entry point, to gain more confidence.

Integration tests, which we’ll discuss later in this chapter and in the book, usually include multiple end results, since it can be impossible to separate code paths at those levels. That’s also one of the reasons integration tests are harder to debug, get up and running, and maintain: they do much more than unit tests, as you’ll soon see.

A third version of our example function is shown in the following listing.

Listing 1.3 Adding a logger call to the function

let total = 0;
 
const totalSoFar = () => {
  return total;
};
 
**const logger = makeLogger();**
 
const sum = (numbers) => {
  const [a, b] = numbers.split(',');
 **logger.info(**                               ❶
    **'this is a very important log output',**   ❶
    **{ firstNumWas: a, secondNumWas: b });**    ❶
 
  const result = parseInt(a) + parseInt(b);
  total += result;
  return result;
};

❶ CA new exit point

You can see that there’s a new exit point (or requirement, or end result) in the function. It logs something to an external entity—perhaps to a file, or the console, or a database. We don’t know, and we don’t care. This is the third type of exit point: _calling a third party_. I also like to refer to it as “calling a _dependency_.”

DEFINITION A _dependency_ is something we don’t have full control over during a unit test. Or it can be something that trying to control in a test would make our lives miserable. Some examples would include loggers that write to files, things that talk to the network, code that’s controlled by other teams, components that take a long time (calculations, threads, database access), and more. The rule of thumb is that if we can fully and easily control what it’s doing, and it runs in memory, and it’s fast, then it’s not a dependency. There are always exceptions to the rule, but this should get you through 80% of the cases, at least.

Figure 1.5 shows how I’d draw this unit of work with all three exit points. At this point we’re still discussing a function-sized unit of work. The entry point is the function call, but now we have three possible paths, or exit points, that do something useful and that the caller can verify publicly.

![01-05](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-05.png)  

Figure 1.5 Showing three exit points from a function

Here’s where it gets interesting: it’s a good idea to have a _separate test for each exit point_. This will make the tests more readable and simpler to debug or change without affecting other outcomes.

## 1.4 Exit point types

We’ve seen that we have three different types of end results:

- The invoked function returns a useful value (not undefined). If this was in a statically typed language such as Java or C#, we’d say it is a public, non-void function.
    
- There’s a _noticeable_ change to the state or behavior of the system before and after invocation that can be determined without interrogating private state.
    
- There’s a callout to a third-party system over which the test has no control. That third-party system doesn’t return any value, or that value is ignored. (Example: the code calls a third-party logging system that was not written by you, and you don’t control its source code.)
    

_XUnit Test Patterns’_ definition of entry and exit points

Gerard Meszaros’ book _XUnit Test Patterns_ (Addison-Wesley Professional, 2007) discusses the notion of _direct inputs and outputs,_ and _indirect inputs and outputs_. _Direct inputs_ are what I like to call entry points. Meszaros refers to it as “using the front door” of a component. _Indirect outputs_ in that book are the other two types of exit points I mentioned (state change and calling a third party).

Both versions of these ideas have evolved in parallel, but the idea of a “unit of work” only appears in this book. A unit of work, coupled with entry and exit points, makes much more sense to me than direct and indirect inputs and outputs, but you can consider this a stylistic choice about how to teach the concept of test scopes. You can find more about _XUnit Test Patterns_ at [xunitpatterns.com](http://xunitpatterns.com/).

Let’s see how the idea of entry and exit points affects the definition of a unit test: A _unit test_ is a piece of code that invokes a unit of work and checks one specific exit point as an end result of that unit of work. If the assumptions about the end result turn out to be wrong, the unit test has failed. A unit test’s scope can span as little as a function or as much as multiple modules or components, depending on how many functions and modules are used between the entry point and the exit point.

## 1.5 Different exit points, different techniques

Why am I spending so much time talking about types of exit points? Because not only is it a great idea to separate the tests for each exit point, but different types of exit points might require different techniques to test successfully:

- Return-value-based exit points (direct outputs in Meszaros’ _XUnit Test Patterns_) should be the easiest exit points to test. You trigger an entry point, you get something back, and you check the value you got back.
    
- State-based tests (indirect outputs) usually require a little more gymnastics. You call something, and then you do another call to check something else (or you call the previous thing again) to see if everything went according to plan.
    

In a third-party situation (indirect outputs), we have the most hoops to jump through. We haven’t discussed this yet, but that’s where we’re forced to use things like _mock objects_ to replace the external system with something we can control and interrogate in our tests. I’ll cover this idea deeply later in the book.

Which exit points make the most problems?

As a rule of thumb, I try to mostly use either return-value-based or state-based tests. I try to avoid mock-object-based tests if I can, and usually I can. As a result, I usually have no more than 5% of my tests using mock objects for verification. Those types of tests complicate things and make maintainability more difficult. Sometimes there’s no escape, though, and we’ll discuss them as we proceed in the next chapters.

## 1.6 A test from scratch

Let’s go back to the first, simplest version of the code (listing 1.1) and try to test it, shall we? If we were to try to write a test for this, what would it look like?

Let’s take the visual approach first with figure 1.6. Our entry point is `sum` with an input of a string called `numbers`. `sum` is also our exit point, since we will get a return value back from it and check its value.

![01-06](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-06.png)  

Figure 1.6 A visual view of our test

It’s possible to write an automated unit test without using a test framework. In fact, because developers have gotten more into the habit of automating their testing, I’ve seen plenty of them doing this before discovering test frameworks. In this section, we’ll write such a test without a framework, so that you can contrast this approach with using a framework in chapter 2.

So, let’s assume test frameworks don’t exist (or that we don’t know they do). We have decided to write our own little automated test from scratch. The following listing shows a very naive example of testing our own code with plain JavaScript.

Listing 1.4 A very naive test against `sum()`

const parserTest = () => {
  try {
    **const result = sum('1,2');**
    if (result === 3) {
      console.log('parserTest example 1 **PASSED'**);
    } else {
      throw new Error(`parserTest: **expected 3 but was ${result}**`);
    }
  } catch (e) {
    console.error(e.stack);
  }
};

No, this code is not lovely. But it’s good enough to explain how tests work. To run this code, we can do the following:

1. Open the command line and type an empty string.
    
2. Add an entry under package.json’s `"scripts"` entry under `"test"` to execute `"node` `mytest.js"` and then execute `npm` `test` on the command line.
    

The following listing shows this.

Listing 1.5 The beginning of our package.json file

{
  "name": "aout3-samples",
  "version": "1.0.0",
  "description": "Code Samples for Art of Unit Testing 3rd Edition",
  "main": "index.js",
  "scripts": {
    **"test": "node ./ch1-basics/custom-test-phase1.js",**
  }
}

The test method invokes the _production module_ (the SUT) and then checks the returned value. If it’s not what’s expected, the test method writes to the console an error and a stack trace. The test method also catches any exceptions that occur and writes them to the console, so that they don’t interfere with the running of subsequent methods. When we use a test framework, that’s usually handled for us automatically.

Obviously, this is an ad hoc way of writing such a test. If you were to write multiple tests like this, you might want to have a generic `test` or `check` method that all tests could use, and which would format the errors consistently. You could also add special helper methods that would check on things like null objects, empty strings, and so on, so that you don’t need to write the same long lines of code in many tests.

The following listing shows what this test would look like with a slightly more generic `check` and `assertEquals` functions.

Listing 1.6 Using a more generic implementation of the `Check` method

const assertEquals = (expected, actual) => {
  if (actual !== expected) {
    throw new Error(`Expected ${expected} but was ${actual}`);
  }
};
 
const check = (name, implementation) => {
  try {
    implementation();
    console.log(`${name} passed`);
  } catch (e) {
    console.error(`${name} FAILED`, e.stack);
  }
};
 
**check('sum with 2 numbers should sum them up', () => {**
  **const result = sum('1,2');**
  **assertEquals(3, result);**
**});**
 
**check('sum with multiple digit numbers should sum them up', () => {**
  **const result = sum('10,20');**
  **assertEquals(30, result);**
**});**

We’ve now created two helper methods: `assertEquals`, which removes boilerplate code for writing to the console or throwing errors, and `check`, which takes a string for the name of the test and a callback to the implementation. It then takes care of catching any test errors, writing them to the console, and reporting on the status of the test.

Built-in asserts

It’s important to note that we don’t need to write our own asserts. We could have easily used Node.js’s built-in assert functions, which were originally built for internal use in testing Node.js itself. We could do so by importing the functions with

const assert = require('assert'); 

However, I’m trying to demonstrate the underlying simplicity of the concept, so we’ll avoid that. You can find more info about Node.js’s `assert` module at [https://nodejs.org/api/assert.html](https://nodejs.org/api/assert.html).

Notice how the tests are easier to read and faster to write with just a couple of helper methods. Unit testing frameworks such as Jest can provide even more generic helper methods like this, so tests are even easier to write. I’ll talk about that in chapter 2. First, let’s talk a bit about the main subject of this book: good unit tests.

## 1.7 Characteristics of a good unit test

No matter what programming language you’re using, one of the most difficult aspects of defining a unit test is defining what’s meant by a _good_ one. Of course, good is relative, and it can change whenever we learn something new about coding. That may seem obvious, but it really isn’t. I need to explain _why_ we need to write better tests—understanding what a unit of work is isn’t enough.

Based on my own experience, involving many companies and teams over the years, most people who try to unit test their code either give up at some point or don’t actually perform unit tests. They waste a lot of time writing problematic tests, and they give up when they have to spend a lot of time maintaining them, or worse, they don’t trust their results.

There’s no point in writing a bad unit test, unless you’re in the process of learning how to write a good one. There are more downsides than upsides to writing bad tests, such as wasting time debugging buggy tests, wasting time writing tests that bring no benefit, wasting time trying to understand unreadable tests, and wasting time writing tests only to delete them a few months later. There’s also a huge issue with maintaining bad tests, and with how they affect the maintainability of production code. Bad tests can actually slow down your development speed, not only when writing test code, but also when writing production code. I’ll touch on all these things later in the book.

By learning what a good unit test is, you can be sure you aren’t starting off on a path that will be hard to fix later on, when the code becomes a nightmare. We’ll also define other forms of tests (component, end to end, and more) later in the book.

### 1.7.1 What is a good unit test?

Every good automated test (not just unit tests) should have the following properties:

- It should be easy to understand the intent of the test author.
    
- It should be easy to read and write.
    
- It should be automated.
    
- It should be consistent in its results (it should always return the same result if you don’t change anything between runs).
    
- It should be useful and provide actionable results.
    
- Anyone should be able to run it with the push of a button.
    
- When it fails, it should be easy to detect what was expected and determine how to pinpoint the problem.
    

A good unit test should also exhibit the following properties:

- It should run quickly.
    
- It should have _full control_ of the code under test (more on that in chapter 3).
    
- It should be fully isolated (run independently of other tests).
    
- It should run in memory without requiring system files, networks, or databases.
    
- It should be as synchronous and linear as possible when that makes sense (no parallel threads if we can help it).
    

It’s impossible for all tests to follow the properties of a good unit test, and that’s fine. Such tests will simply transition to the realm of integration testing (the topic of section 1.8). Still, there are ways to refactor some of your tests to conform to these properties.

Replacing the database (or another dependency) with a stub

We’ll discuss stubs in later chapters, but, in short, they are fake dependencies that emulate the real ones. Their purpose is to simplify the process of testing because they are easier to set up and maintain.

Beware of in-memory databases, though. They can help you isolate tests from each other (as long as you don’t share database instances between tests) and thus adhere to the properties of good unit tests, but such databases lead to an awkward, in-between spot. In-memory databases aren’t as easy to set up as stubs. At the same time, they don’t provide as strong guarantees as real databases. Functionality-wise, an in-memory database may differ drastically from the production one, so tests that pass an in-memory database may fail the real one, and vice versa. You’ll often have to rerun the same tests manually against the production database to gain additional confidence that your code works. Unless you use a small and standardized set of SQL features, I recommend sticking to either stubs (for unit tests) or real databases (for integration testing).

The same is true for solutions like jsdom. You can use it to replace the real DOM, but make sure it supports your particular use cases. Don’t write tests that require you to manually recheck them.

Emulating asynchronous processing with linear, synchronous tests

With the advent of promises and `async/await`, asynchronous coding has become standard in JavaScript. Our tests can still verify asynchronous code synchronously, though. Usually that means triggering callbacks directly from the test or explicitly waiting for an asynchronous operation to finish executing.

### 1.7.2 A unit test checklist

Many people confuse the act of testing their software with the concept of a unit test. To start off, ask yourself the following questions about the tests you’ve written and executed up to now:

- Can I run and get results from a test I wrote two weeks or months or years ago?
    
- Can any member of my team run and get results from tests I wrote two months ago?
    
- Can I run all the tests I’ve written in no more than a few minutes?
    
- Can I run all the tests I’ve written at the push of a button?
    
- Can I write a basic test in no more than a few minutes?
    
- Do my tests pass when there are bugs in another team’s code?
    
- Do my tests show the same results when run on different machines or environments?
    
- Do my tests stop working if there’s no database, network, or deployment?
    
- If I delete, move, or change one test, do other tests remain unaffected?
    

If you answered “no” to any of these questions, there’s a high probability that what you’re implementing either isn’t fully automated or it isn’t a unit test. It’s definitely _some_ kind of test, and it might be as important as a unit test, but it has drawbacks compared to tests that would let you answer yes to all of those questions.

“What was I doing until now?” you might ask. You’ve been doing integration testing.

## 1.8 Integration tests

I consider _integration tests_ to be any tests that don’t live up to one or more of the conditions outlined previously for good unit tests. For example, if the test uses the real network, the real rest APIs, the real system time, the real filesystem, or a real database, it has stepped into the realm of integration testing.

If a test doesn’t have control of the system time, for example, and it uses the current `new` `Date()` in the test code, then every time the test executes, it’s essentially a different test because it uses a different time. It’s no longer consistent. That’s not a bad thing per se. I think integration tests are important counterparts to unit tests, but they should be separated from them to achieve a feeling of “safe green zone,” which is discussed later in this book.

If a test uses the real database, it’s no longer only running in memory—its actions are harder to erase than when using only in-memory fake data. The test will also run longer, and we won’t easily be able to control how long data access takes. Unit tests should be _fast_. Integration tests are usually much slower. When you start having hundreds of tests, every half-second counts.

Integration tests increase the risk of another problem: testing too many things at once. For example, suppose your car breaks down. How do you learn what the problem is, let alone fix it? An engine consists of many subsystems working together, each relying on the others to help produce the final result: a moving car. If the car stops moving, the fault could be with any of the subsystems—or with more than one. It’s the integration of those subsystems (or layers) that makes the car move. You could think of the car’s movement as the ultimate integration test of these parts as the car goes down the road. If the test fails, all the parts fail together; if it succeeds, all the parts succeed.

The same thing happens in software. The way most developers test their functionality is through the final functionality of the app or REST API or UI. Clicking some button triggers a series of events—functions, modules, and components working together to produce the final result. If the test fails, all of these software components fail as a team, and it can be difficult to figure out what caused the failure of the overall operation (see figure 1.7).

![01-07](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-07.png)  

Figure 1.7 You can have many failure points in an integration test. All the units have to work together, and each could malfunction, making it harder to find the source of a bug.

As defined in _The Complete Guide to Software Testing_ by Bill Hetzel (Wiley, 1988), integration testing is “an orderly progression of testing in which software and/or hardware elements are combined and tested until the entire system has been integrated.” Here’s my own variation on defining integration testing:

Integration testing is testing a unit of work without having full control over all of its real dependencies, such as other components by other teams, other services, the time, the network, databases, threads, random number generators, and more.

To summarize, an integration test uses real dependencies; unit tests isolate the unit of work from its dependencies so that they’re easily consistent in their results and can easily control and simulate any aspect of the unit’s behavior.

Let’s apply the questions from section 1.7.2 to integration tests and consider what you want to achieve with real-world unit tests:

- _Can I run and get results from a test I wrote two weeks or months or years ago?_  
    If you can’t, how would you know whether you broke a feature that you created earlier? Shared data and code changes regularly during the life of an application, and if you can’t (or won’t) run tests for all the previously working features after changing your code, you just might break it without knowing—this is known as a _regression_. Regressions seem to occur a lot near the end of a sprint or release, when developers are under pressure to fix existing bugs. Sometimes they introduce new bugs inadvertently as they resolve old ones. Wouldn’t it be great to know that you broke something within 60 seconds of breaking it? You’ll see how that can be done later in this book.
    

DEFINITION A _regression_ is broken functionality—code that used to work. You can also think of it as one or more units of work that once worked and now don’t.

- _Can any member of my team run and get results from tests I wrote two months ago?_  
    This goes with the previous point but takes it up a notch. You want to make sure that you don’t break someone else’s code when you change something. Many developers fear changing legacy code in older systems for fear of not knowing what other code depends on what they’re changing. In essence, they risk changing the system into an unknown state of stability.  
    Few things are scarier than not knowing whether the application still works, especially when you didn’t write that code. If you have that safety net of unit tests and know you aren’t breaking anything, you’ll be much less afraid of taking on code you’re less familiar with.  
    Good tests can be accessed and run by anyone.
    

DEFINITION _Legacy code_ is defined by Wikipedia as “old computer source code that is no longer supported on the standard hardware and environments” ([https://en.wikipedia.org/wiki/Legacy_system](https://en.wikipedia.org/wiki/Legacy_system)), but many shops refer to any older version of the application currently under maintenance as legacy code. It often refers to code that’s hard to work with, hard to test, and usually even hard to read. A client once defined legacy code in a down-to-earth way: “code that works.” Many people like to define legacy code as “code that has no tests.” _Working Effectively with Legacy Code_ by Michael Feathers (Pearson, 2004) uses “code that has no tests” as an official definition of legacy code, and it’s a definition to be considered while reading this book.

- _Can I run all the tests I’ve written in no more than a few minutes?_
    
    If you can’t run your tests quickly (seconds are better than minutes), you’ll run them less often (daily, or even weekly or monthly in some places). The problem is that when you change code, you want to get feedback as early as possible to see if you broke something. The more time required between running the tests, the more changes you make to the system, and the (many) more places you’ll have to search for bugs when you find that you broke something.  
    Good tests should run _quickly_.
    
- _Can I run all the tests I’ve written at the push of a button?_  
    If you can’t, it probably means that you have to configure the machine on which the tests will run so that they run correctly (setting up a Docker environment, or setting connection strings to the database, for example), or it may mean that your unit tests aren’t fully automated. If you can’t fully automate your unit tests, you’ll probably avoid running them repeatedly, as will everyone else on your team.  
    No one likes to get bogged down with configuring details to run tests, just to make sure that the system still works. Developers have more important things to do, like writing more features into the system. But they can’t do that if they don’t know the state of the system.  
    Good tests should be easily executed in their original form, not manually.
    
- _Can I write a basic test in no more than a few minutes?_  
    One of the easiest ways to spot an integration test is that it takes time to prepare correctly and to implement, not just to execute. It takes time to figure out how to write it because of all the internal, and sometimes external, dependencies. (A database may be considered an external dependency.) If you’re not automating the test, dependencies are less of a problem, but you’re losing all the benefits of an automated test. The harder it is to write a test, the less likely you are to write more tests or to focus on anything other than the “big stuff” that you’re worried about. One of the strengths of unit tests is that they tend to test every little thing that might break, not only the big stuff. People are often surprised at how many bugs they can find in code they thought was simple and bug free.  
    When you concentrate only on the big tests, the overall confidence in your code is still very much lacking. Many parts of the code’s core logic aren’t tested (even though you may be covering more components), and there may be many bugs that you haven’t considered and might be “unofficially” worried about.  
    Good tests against the system should be easy and quick to write, once you’ve figured out the patterns you want to use to test your specific set of objects, functions, and dependencies (the _domain model_).
    
- _Do my tests pass when there are bugs in another team’s code? Do my tests show the same results when run on different machines or environments? Do my tests stop working if there’s no database, network, or deployment?_  
    These three points refer to the idea that our test code is isolated from various dependencies. The test results are consistent because we have control over what those indirect inputs into our system provide. We can have fake databases, fake networks, fake time, and fake machine culture. In later chapters, I’ll refer to those points as _stubs_ and _seams_ in which we can inject those stubs.
    
- _If I delete, move, or change one test, do other tests remain unaffected?_  
    Unit tests usually don’t need to have any shared state, but integration tests often do, such as an external database or service. Shared state can create a dependency between tests. For example, running tests in the wrong order can corrupt the state for future tests.
    

WARNING Even experienced unit testers can find that it may take 30 minutes or more to figure out how to write the very _first_ unit test against a domain model they’ve never unit tested before. This is part of the work and is to be expected. The second and subsequent tests on that domain model should be very easy to accomplish once you’ve figured out the entry and exit points of the unit of work.

We can recognize three main criteria in the previous questions and answers:

- _Readability_—If we can’t read it, then it’s hard to maintain, hard to debug, and hard to know what’s wrong.
    
- _Maintainability_—If maintaining the test or production code is painful because of the tests, our lives will become a living nightmare.
    
- _Trust_—If we don’t trust the results of our tests when they fail, we’ll start manually testing again, losing all the time benefit the tests are supposed to provide. If we don’t trust the tests when they _pass_, we’ll start debugging more, again losing any time benefit.
    

From what I’ve explained so far about what a unit test is not and what features need to be present for testing to be useful, I can now start to answer the primary question this chapter poses: what is a good unit test?

## 1.9 Finalizing our definition

Now that I’ve covered the important properties that a unit test should have, I’ll define unit tests once and for all:

A unit test is an automated piece of code that invokes the unit of work through an entry point and then checks one of its exit points. A unit test is almost always written using a unit testing framework. It can be written easily and runs quickly. It’s trustworthy, readable, and maintainable. It is consistent as long as the production code we control has not changed.

This definition certainly looks like a tall order, particularly considering how many developers implement unit tests poorly. It makes us take a hard look at the way we, as developers, have implemented testing up until now, compared to how we’d like to implement it. (Trustworthy, readable, and maintainable tests are discussed in depth in chapters 7 through 9.)

In the first edition of this book, my definition of a unit test was slightly different. I used to define a unit test as “only running against control flow code,” but I no longer think that’s true. Code without logic is usually used as part of a unit of work. Even properties with no logic will get used by a unit of work, so they don’t have to be specifically targeted by tests.

DEFINITION _Control flow code_ is any piece of code that has some sort of logic in it, small as it may be. It has one or more of the following: an `if` statement, a loop, calculations, or any other type of decision-making code.

Getters and setters are good examples of code that usually doesn’t contain any logic and so don’t require specific targeting by the tests. It’s code that will probably get used by the unit of work you’re testing, but there’s no need to test it directly. But watch out: once you add any logic inside a getter or setter, you’ll want to make sure that logic is being tested.

In the next section, we’ll stop talking about what is a good test and talk about _when_ you might want to write tests. I’ll discuss test-driven development, because it is often put in the same bucket as doing unit testing. I want to make sure we set the record straight on that.

## 1.10 Test-driven development

Once you know how to write readable, maintainable, and trustworthy tests with a unit testing framework, the next question is _when_ to write the tests. Many people feel that the best time to write unit tests for software is after they’ve created some functionality and just before they merge their code into remote source control.

Also, to be a bit blunt, a lot of people don’t believe writing tests is a good idea, but have realized through trial and error that there are strict testing requirements in source control reviews, so they _have_ to write tests to appease the code review gods and get their code merged into the main branch. (That kind of dynamic is a great source of bad tests, and I’ll address it in the third part of this book.)

A growing number of developers prefer writing unit tests incrementally, during the coding session and before each piece of very small functionality is implemented. This approach is called _test-first_ or _test-driven_ development (TDD).

NOTE There are many different views on exactly what test-driven development means. Some say it’s test-first development, and some say it means you have a lot of tests. Some say it’s a way of designing, and others feel it could be a way to drive your code’s behavior with only some design. In this book, TDD means test-first development, with design taking an incremental role in the technique (besides this section, TDD won’t be discussed in this book).

Figures 1.8 and 1.9 show the differences between traditional coding and TDD. TDD is different from traditional development, as figure 1.9 shows. You begin by writing a test that fails; then you move on to creating the production code, seeing the test pass, and continuing on to either refactor your code or create another failing test.

![01-08](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-08.png)  

Figure 1.8 The traditional way of writing unit tests

 

![01-09](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-09.png)  

Figure 1.9 Test-driven development—a bird’s-eye view. Notice the circular nature of the process: write the test, write the code, refactor, write the next test. It shows the incremental nature of TDD: small steps lead to a quality end result with confidence.

This book focuses on the technique of writing good unit tests, rather than on TDD, but I’m a big fan of TDD. I’ve written several major applications and frameworks using TDD, I’ve managed teams that utilize it, and I’ve taught hundreds of courses and workshops on TDD and unit testing techniques. Throughout my career, I’ve found TDD to be helpful in creating quality code, quality tests, and better designs for the code I was writing. I’m convinced that it can work to your benefit, but it’s not without a price (time to learn, time to implement, and more). It’s definitely worth the admission price, though, if you’re willing to take on the challenge of learning it.

### 1.10.1 TDD: Not a substitute for good unit tests

It’s important to realize that TDD doesn’t ensure project success or tests that are robust or maintainable. It’s quite easy to get caught up in the technique of TDD and not pay attention to the way unit tests are written: their naming, how maintainable or readable they are, and whether they test the right things or might themselves have bugs. That’s why I’m writing this book—because writing good tests is a separate skill from TDD.

The technique of TDD is quite simple:

1. _Write a failing test to prove code or functionality is missing from the end product._ The test is written _as if_ the production code were already working, so the test failing means there’s a bug in the production code. How do I know? The test is written such that it would pass if the production code had no bugs.  
    In some languages other than JavaScript, the test might not even compile at first, since the code doesn’t exist yet. Once it does run, it should be failing, because the production code is still not working. This is where a lot of the “design” in test-driven-design thinking happens.
    
2. _Make the test pass by adding functionality to the production code that meets the expectations of your test._ The production code should be kept as simple as possible. Don’t touch the test. You have to make it pass only by touching production code.
    
3. _Refactor your code._ When the test passes, you’re free to move on to the next unit test or to refactor your code (both production code and tests) to make it more readable, to remove code duplication, and so on. This is another point where the “design” part happens. We refactor and can even redesign our components while still keeping the old functionality.  
    Refactoring steps should be very small and incremental, and we run all the tests after each small step to make sure we didn’t break anything with our changes. Refactoring can be done after writing several tests or after writing each test. It’s an important practice, because it ensures your code gets easier to read and maintain, while still passing all of the previously written tests. There’s a whole section (8.3) on refactoring later in the book.
    

DEFINITION _Refactoring_ means changing a piece of code _without_ changing its functionality. If you’ve ever renamed a method, you’ve done refactoring. If you’ve ever split a large method into multiple smaller method calls, you’ve refactored your code. The code still does the same thing, but it becomes easier to maintain, read, debug, and change.

The preceding steps sound technical, but there’s a lot of wisdom behind them. Done correctly, TDD can make your code quality soar, decrease the number of bugs, raise your confidence in the code, shorten the time it takes to find bugs, improve your code’s design, and keep your manager happier. If TDD is done incorrectly, it can cause your project schedule to slip, waste your time, lower your motivation, and lower your code quality. It’s a double-edged sword, and many people find this out the hard way.

Technically, one of the biggest benefits of TDD that nobody tells you about is that by seeing a test fail, and then seeing it pass without changing the test, you’re basically testing the test itself. If you expect it to fail and it passes, you might have a bug in your test or you’re testing the wrong thing. If the test failed, you fixed it, and now you expect it to pass, and it still fails, your test could have a bug, or maybe it’s expecting the wrong thing to happen.

This book deals with readable, maintainable, and trustworthy tests, but if you add TDD on top, your confidence in your own tests will increase by seeing the failed, you fixed it, tests failing when they should and passing when they should. In test-after style, you’ll usually only see them pass when they should, and fail when they shouldn’t (since the code they test should already be working). TDD helps with that a lot, and it’s also one of the reasons developers do far less debugging when practicing TDD than when they’re simply unit testing after the fact. If they trust the tests, they don’t feel a need to debug it “just in case.” That’s the kind of trust you can only gain by seeing both sides of the test—failing when it should and passing when it should.

### 1.10.2 Three core skills needed for successful TDD

To be successful in test-driven development, you need three different skill sets: knowing how to write good tests, writing them test-first, and designing the tests and the production code well. Figure 1.10 shows these more clearly:

- _Just because you write your tests first doesn’t mean they’re maintainable, readable, or trustworthy._ Good unit testing skills are what this book is all about.
    
- _Just because you write readable, maintainable tests doesn’t mean you’ll get the same benefits as when writing them test-first._ Test-first skills are what most of the TDD books out there teach, without teaching the skills of good testing. I would especially recommend Kent Beck’s _Test-Driven Development: By Example_ (Addison-Wesley Professional, 2002).
    
- _Just because you write your tests first, and they’re readable and maintainable, doesn’t mean you’ll end up with a well-designed system._ Design skills are what make your code beautiful and maintainable. I recommend _Growing Object-Oriented Software, Guided by Tests_ by Steve Freeman and Nat Pryce (Addison-Wesley Professional, 2009) and _Clean Code_ by Robert C. Martin (Pearson, 2008) as good books on the subject.
    

![01-10](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297489/files/OEBPS/Images/01-10.png)  

Figure 1.10 Three core skills of test-driven development

A pragmatic approach to learning TDD is to learn each of these three aspects separately; that is, to focus on one skill at a time, ignoring the others in the meantime. The reason I recommend this approach is that I often see people trying to learn all three skill sets at the same time, having a really hard time in the process, and finally giving up because the wall is too high to climb. By taking a more incremental approach to learning this field, you relieve yourself of the constant fear that you’re getting it wrong in a different area than you’re currently focusing on.

In the next chapter, you’ll start writing your first unit tests using Jest, one of the most commonly used test frameworks for JavaScript.

## Summary

- A good unit test has these qualities:
    
    - It should run quickly.
        
    - It should have full control of the code under test.
        
    - It should be fully isolated (it should run independently of other tests).
        
    - It should run in memory without requiring filesystem files, networks, or databases.
        
    - It should be as synchronous and linear as possible (no parallel threads).
        
- Entry points are public functions that are the doorways into our units of work and trigger the underlying logic. Exit points are the places you can inspect with your test. They represent the effects of the units of work.
    
- An exit point can be a return value, a change of state, or a call to a third-party dependency. Each exit point usually requires a separate test, and each type of exit point requires a different testing technique.
    
- A unit of work is the sum of actions that take place between the invocation of an entry point up until a noticeable end result through one or more exit points. A unit of work can span a function, a module, or multiple modules.
    
- Integration testing is just unit testing with some or all of the dependencies being real and residing outside of the current execution process. Conversely, unit testing is like integration testing, but with all of the dependencies in memory (both real and fake), and we have control over their behavior in the test.
    
- The most important attributes of any test are readability, maintainability, and trust. _Readability_ tells us how easy it is to read and understand the test. _Maintainability_ is the measure of how painful it is to maintain the test code. Without _trust_, it’s harder to introduce important changes (such as refactoring) in a codebase, which leads to code deterioration.
    
- Test-driven development (TDD) is a technique that advocates for writing tests before the production code. This approach is also referred to as a test-first approach (as opposed to code-first).
    
- The main benefit of TDD is verifying the correctness of your tests. Seeing your tests fail before writing production code ensures that these same tests would fail if the functionality they cover stops working properly.