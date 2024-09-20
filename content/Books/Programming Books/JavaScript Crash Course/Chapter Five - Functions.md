---
id: 01J83S5XZAZWXQ7G6QG8FNWBXM
title: Chapter Five - Functions
modified: 2024-09-18T20:37:27-04:00
description: Learn how to use functions in JavaScript
tags:
  - javascript
  - books
  - web-development
  - programming
---
# Chapter Five - Functions
- Learn how to create functions in various ways
- ## Declaring and Calling Functions
	- Function declaration
		- block of code that defines a function
```javascript
function sayHello(name) {
  console.log(`Hello ${name}!`);
}
```


- ## Passing a Function as an Argument
	- Functions are first class citizens, which means they can be used like any other value, like a number or a string.
		- You can store a function in a variable or pass a function as an argument to another function
		- Callback - function is passed as an argument because the function it's passed to is said to "call it back" by executing it
		- We can illustrate this with `setTimeout` with the following
```javascript
function sayHi() {
  console.log("Hi!");
}
setTimeout(sayHi, 2000);
```
- ## Other Function Syntax
	- There are other ways to call functions other than declarations, like expressions
	- ### Function Expressions
		- Also known as a function literal - code literal whose value is a function just as 123 is  a literal value is the number 123
		- Whereas a function declaration creates a function and binds it to a name, a function expression is an expression that evaluates to (returns) a function for you to do with what you will.
		- Function expression doesn't have to include a name but it can if you want
			- Without names, they are called anonymous functions
		- Can't be written at the start of a line of code of JavaScript will think it's a function declaration; there has to be some code before the function keyword - that is why function expressions are often used in contexts where functions need to be treated as values.
			- Below you'll see the function keyword appears on the right side of an assignment statement rather than at the start of a line, so JavaScript treats this as a function expression
			- Function is anonymous since we don't provide a name after the function keyword
```javascript
let addExpression = function (x, y) {
  return x + y;
}
addExpression(1, 2);
```
- Function expressions are very similar to declarations in a lot of ways.
	- But when it comes to passing functions as arguments, there are certain advantages with expressions.
		- Here is how we can do the earlier example of `sayHi` with `setTimeout`
```javascript
setTimeout(function() {
  console.log("Hi!");
}, 2000);
// 2
// Hi!
```
- ### Arrow Functions
	- More compact version of a function expression
	- You can use them anywhere a normal function expression would work to save yourself typing
	- The second example is an even shorter version called the concise arrow function
	- There's an either shorter version if there is only one parameter 
```javascript
let addArrow = (x, y) => {
  return x + y;
}
addArrow(2, 2)
// 4

let addArrowConcise = (x, y) => x + y;

let square = x => x * x;
squared(3);
// 9
```
- Let's consider our `sayHi` and `setTimeout` example again
```javascript
setInterval(() => {
  console.log("Beep");
}, 1000);
// 3
// Beep
```

Our arrow function takes no arguments, so it begins with an empty set of paraentheses for the parameter list. 
Closing brace at the end of the body is followed by a comma to separate the arrow function for the next argument
- ## Rest Parameters
	- Sometimes we want a function to accept a variable number of arguments, for example, you take an input of somebody's favorite colors and print them in a sentence
```javascript
let myColors = (name, ...favoriteColors) => {
  let colorString = favoriteColors.join(", ");
  console.log(`My name is ${name} and my favorite colors are ${colorString.}`);
};
myColors("Nick", "blue", "green", "orange");
// My name is Nick and my favorite colors are blue, green, orange.
```