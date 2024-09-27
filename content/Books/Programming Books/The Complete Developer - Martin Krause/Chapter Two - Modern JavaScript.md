---
id: 01J8TFEM5NVVP9D8J83SH1YNH0
modified: 2024-09-27T16:11:36-04:00
title: Chapter Two - Modern JavaScript
description: Using Modern JavaScript
tags:
  - javascript
  - books
  - full-stack
  - web-development
  - programming
---
## Use Name and Default Exports
- There are two kinds of Next.js exports: _named_ and _default_. These exports use slightly different syntaxes when you import them later. Default exports require you to define new function names on import. For named exports, renaming is optional and done with the as statement.
- It’s considered a best practice to use named exports over default exports, because named exports define a clear and unique interface for the module’s functionality. When we use default exports, the user risks importing the same function under different names. TypeScript, which we’ll cover in [Chapter 3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml), recommends that we use default exports if the module has one clear purpose and a single export. In contrast, it recommends using named exports whenever the module exports more than one item.
- You should know the syntax of default exports so that you can work with third-party modules that use them. Unlike named exports, there can be only one default export per file, marked by the default keyword ([Listing 2-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-1)).##
## Importing Modules
```javascript
import getFoo from "default.js"
```
## Declaring Variables
- These variables differ in their _scope_, which defines the code area in which we can access and use them. JavaScript has multiple levels of scope: global, module, function, and block. _Block_ scope, which applies to any block of code enclosed in curly brackets, is the smallest unit of scope. Every time you use curly brackets, you create a new block scope. In comparison, you make a _function_ scope when you define a function. The scope is limited to the code area inside a specific function. The _module_ scope applies only to a specific module, whereas the _global_ scope applies to the entire program. Variables defined in the global scope are available in every part of your code.
- As you’ll see in the following code listings, a variable is always available in its own scope and all of its child scopes. Hence, you should remember that, for example, a function scope can contain multiple block scopes. The same variable name can be defined twice in one program as long as each variable occurs in different scopes.
## Hoisted Variables
- Traditional JavaScript declares variables with the var keyword. The scope of these variables is the current execution context (usually the enclosing function). If declared outside any function, the variable’s scope is global, and the variable creates a property on the global object.
- Unlike for all other variables, the runtime environment moves, or _hoists_, the declaration of var to the top of its scope upon execution. Therefore, you can call these variables in your code before you define them. [Listing 2-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-5) shows a short example of hoisting.
```javascript
function scope() {
	foo = 1;
	var foo;
}
```
In this listing, we assign a value to a variable before declaring it in the following line. In languages like Java and C, we can’t use variables before we declare them, and any attempt to do so will throw an error. However, because of hoisting in JavaScript, the parser moves all variable declarations defined with the var keyword to the top of the scope. Thus, the code is equivalent to that in [Listing 2-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-6).

```javascript
function scope() {
	var foo;
	foo = 1;
}
```
Because of hoisting, block scope does not apply to variables declared with the var keyword. They are always hoisted. To illustrate this, take a look at [Listing 2-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-7), where we declare a global variable globalVar, a variable foo inside the function scope, and a variable bar inside a block scope, all with the var keyword.
```javascript
var globalVar = "global";
function scope() {
	var foo = "1";
	if (true) {
		var bar = "2";
	}
	console.log(globalVar);
	console.log(window.globalVar);
	console.log(foo);
	console.log(bar);
}

```
## Constant Like Data
- Modern JavaScript introduced another new keyword, const, for declaring constants such as data types. Like let, const does not create properties of the global object when declared globally. They, too, are considered non-hoisted, as they cannot be accessed before being declared.
- Constants in JavaScript are different from those in many other languages, where they function as immutable data types. In JavaScript, constants only _look_ immutable. In fact, they are read-only references to their value. Therefore, you cannot directly reassign another value to the variable identifier for primitive data types. However, objects or arrays are non-primitive data types, so even when you use const, you can mutate their values through methods or direct property access.
```javascript
const primitiveDataType = 1;
try {
	primitiveDataType = 2;
} catch (err) {
	console.log(err);
}

const nonPrimitiveDataType = [];
nonPrimitiveDataType.push(1);

console.log(nonPrimitiveDataType);
```
- We declare and assign a value to two constant-like data structures. Now when we try to reassign a value to the primitive data structure, the runtime throws the error Attempted to assign to readonly property. Because we used const, we cannot reassign its value. In contrast, we can modify the nonPrimitiveDataType array (done here with the push method) and append a value without running into an error. The array should now contain one item with the value 1; hence, we see [1] in the console.
## Arrow Functions
- Modern JavaScript introduced arrow functions as alternatives to regular functions. There are two concepts you need to know about arrow functions. First, they use a different syntax than regular functions. Defining an arrow function is much quicker, requiring just a few characters and one line of code. The second important, but not so obvious, change is that they use something called a lexical scope, making them more intuitive and less error prone.
### Writing Arrow Functions
- Instead of using the function keyword to declare an arrow function, we use the equal-to and greater-than signs to form an arrow (=>). This syntax, also called the _fat arrow_, reduces noise and results in more compact code. Therefore, modern JavaScript prefers this syntax when passing functions as arguments.
- In addition, if an arrow function has only one parameter and one statement, we can omit the curly brackets and the return keyword. In this compact form, we call the function a _concise body_ function. [Listing 2-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-10) shows the definition of a traditional function followed by an arrow function.
```javascript
const tarditional = function (x) {
	return x * x;
}

const conciseBody = x => x * x;
```
- We first define a standard function with the function keyword and familiar return statement. Then we write the same functionality as an arrow function with the concise body syntax. Here we omit the curly brackets and use an implied return statement, without the return keyword.
## Understanding Lexical Scope
- Unlike regular functions, arrow functions do not bind their scope to the object that calls the function. Instead, they use a _lexical scope_, in which the surrounding scope determines the value of the this keyword. Therefore, the scope to which this refers in an arrow function always represents the object _defining_ the arrow function instead of the object _calling_ the function. [Listing 2-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-11) illustrates the concepts of lexical and defining scopes.
```javascript
this.scope = "lexical scope";

const scopeOf = {
	scope: "defining scope",
	
	traditional: function () {
		return this.scope;
	},
	arrow: () => {
		return this.scope;
	},
};

console.log(scopeOf.traditional());
console.log(scopeOf.arrow());
```