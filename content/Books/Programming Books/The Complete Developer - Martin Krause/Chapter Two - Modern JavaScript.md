---
id: 01J8TFEM5NVVP9D8J83SH1YNH0
modified: 2024-09-29T11:32:38-04:00
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
## Exploring Practical Use Cases
- Because functions are first-class citizens in JavaScript, we can pass them as arguments to other functions. In [Chapter 1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter1.xhtml), you used this pattern to define callbacks in Node.js or previously when you worked with event handlers in the browser. But when you use regular functions as callbacks, the code quickly gets cluttered with function statements and curly brackets, even if the actual code in the callback is quite simple. Arrow functions allow for a clean and simple syntax in callbacks. In [Listing 2-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-12), we use a callback on the array filter method and define it as a traditional function and as an arrow function.
```javascript
let numbers = [-2, -1, 0, 1, 2];

let traditional = numbers.filter(function(num) {
	  return num >= 0;
	}
);

let arrow = numbers.filter(num => num >= 0);

console.log(traditional);
console.log(arrow);
```
- The first version of the callback is a traditional function, whereas the second implementation uses an arrow function with a concise body syntax. Both return the same array: [0, 1, 2]. We see that the actual functionality, to remove all negative numbers from the array, is a simple check to see if the current item is greater than or equal to zero. The traditional function is harder to understand, as it requires additional characters. Once you fully grasp the arrow syntax, you’ll enhance the readability of your code and, in turn, improve the code quality.
## Creating Strings
- Modern JavaScript introduces untagged and tagged template literals. _Template literals_ are a simple way to add variables and expressions to a string. This string interpolation can span multiple lines and include single and double quotation marks without requiring escaping. We enclose template literals in backticks (`) and indicate a variable or expression in the template by using a dollar sign ($) and curly brackets.
- An _untagged_ template literal is just a string enclosed in backticks. The parser interpolates the variables and expressions and returns a string. As a full-stack developer, you’ll use this pattern every time you want to add variables to a string or concatenate multiple strings. [Listing 2-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-13) shows an example of an untagged template literal. They can span multiple lines without the need for any control characters.

```javascript
function tag(literal, ...values) {
    console.log("literal", literal);
    console.log("values", values);

    let result;
    switch (literal[1]) {
        case " plus ":
            result = values[0] + values[1];
            break;
        case " minus ":
            result = values[0] - values[1];
            break;
    }
    return `${values[0]}${literal[1]}${values[1]} is ${result}`;
}

let a = 1;
let b = 2;
let output = tag`What is ${a} plus ${b}?`;

console.log(output)
```
## Asynchronous Scripts
- JavaScript is single-threaded, which means that it can run only one task at a time. Therefore, long-running tasks can block the application. A simple solution is _asynchronous_ programming, a pattern where you start a long-running task without blocking the whole application. While your script waits for a result, the rest of the application can still respond to interactions or user interface events and perform other calculations.
### Avoiding Traditional Callbacks
- Traditional JavaScript implements asynchronous code with callback functions executed after another function returns a result. You’ve probably already used callbacks when your code has needed to react to an event instead of running immediately. One common use case for this technique in full-stack web development is performing I/O operations in Node.js or calling remote APIs. [Listing 2-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-15) provides an example of an I/O operation. We import the Node.js fs module, which handles filesystem operations, and use a callback function to display the file’s contents as soon as the operation concludes.
- Reading a file is a common example of asynchronous scripting. We don’t want the application to be blocked while waiting for the file content to be ready; however, we also need to use the file’s content in a specific part of the application.
- Here we create the callback function and pass it as a parameter to the fs.readFile function. This function reads a file from the filesystem and executes the callback as soon as the I/O operation fails or succeeds. The callback receives the file data and an optional error object, which we write to the console for now.
- Callbacks are a clumsy solution to asynchronous scripting. As soon as you have multiple dependent callback functions, you end up in so-called callback hell, where every callback function takes another callback function as an argument. The result is a pyramid of functions that are difficult to read and prone to errors. Modern JavaScript introduced promises and async/await as an alternative to callbacks.
```javascript
const fs = require("fs");

const callback = (err, data) => {
    if (err) {
        return console.log("error");
    }
    console.log(`File content ${data}.`);
};

fs.readFile(" file.txt", callback);
```
### Using Promises
- _Promises_ provide a much cleaner syntax for chainable asynchronous tasks. Similar to callbacks, they defer further tasks until a previous action has completed or failed. Essentially, promises are function calls that do not return an immediate result. Instead, they promise to return the result at some later point. If there is an error, the promise is rejected instead of resolved.
- Like I/O operations on the filesystem, network requests are long-running tasks that block the application. Therefore, we should use asynchronous patterns to load remote datasets. As in [Listing 2-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-15), we need to wait until the operation is complete before we can process the requested data or handle an error.
- The fetch API is promise-based by default. As soon as the promise resolves and the state changes to fulfilled, the following then function receives the response object. We then parse the data and pass the JSON object to the next function in the _promise chain_, a sequence of functions connected with a dot (.then). If there is an error, the promise is rejected. In this case, we catch the error and write it to the console.
```javascript
function fetchData(url) {
    fetch(url)
        .then((response) => response.json())
        .then((json) => console.log(json))
        .catch((error) => {
            console.error(`Error: ${error}`);
        });
}
fetchData("https://www.usemodernfullstack.dev/api/v1/users");

```
### Simplifying Asynchronous Functions
- Modern JavaScript introduces a new, simpler pattern for handling asynchronous requests: the async/await keywords. Instead of relying on chained functions, we can write code whose structure is similar to regular synchronous code by employing these keywords.
- When using this pattern, you mark functions explicitly as asynchronous with async. Then you use await instead of the promise-based syntax for your asynchronous code. In [Listing 2-17](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-17), we use the native fetch API with async/await to perform another long-running task and fetch JSON data from a remote location. This code is functionally the same as [Listing 2-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-16), and you should see that its syntax is more intuitive and cleaner than the chain of then calls.
- First we declare the function as async to enable the await keyword inside the function. Then we use await to wait for the response of the fetch call. Unlike the promise syntax we used before, await simplifies the code. It awaits the response object and returns it. Thus, the code block looks similar to regular synchronous code.
- This pattern requires us to handle errors manually. Unlike with promises, there is no default reject function. Therefore, we must wrap await statements in a try...catch block to handle error states gracefully.
```javascript
async function fetchData (url) {
    try {
        const response = await fetch(url);
        const json = await response.json();
    } catch (error) {
        console.error(`Error: ${error}`);
    }
}

fetchData("https://www.usemodernfullstack.dev/api/v1/users");
```
## Looping through an array
- Modern JavaScript introduced a whole set of new array functions. The most important one for full-stack web development is array.map. It allows us to run a function on each array item and return a new array with the modified items, preserving the original array. Developers commonly use it in React to generate a list or populate JSX with datasets from arrays. You will use this pattern extensively once we introduce React in [Chapter 4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter4.xhtml).

In [Listing 2-18](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-18), we use array.map to iterate over an array of numbers and create an arrow function as a callback.

```javascript
const original = [1,2,3,4];
const multiplied = original.map((item) => item * 10);
console.log(`original array: ${original}`);
console.log(`multiplied array: ${multiplied}`);
```
We iterate over the array items and pass each of them to the callback function. Here we multiply each item by 10, and then array.map returns an array with the multiplied items.

When we log the initial array and the returned array, we see that the original array still contains the actual, unchanged numbers (1,2,3,4). Only the multiplied array contains the new, modified items (10,20,30,40).

- ## Dispersing Arrays and Objects**** 
	- Modern JavaScript’s spread operator is written as three dots (...). It _spreads out_, or expands, the values of an array or the properties of an object into their own variables or constants.
	- Technically, the spread operator copies its content to variables that allocate their own memory. In [Listing 2-19](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-19), we use the spread operator to assign the multiple values of an object to several constants. You’ll use this pattern in nearly all React code to access component properties.
```javascript
let object = { fruit: 'Apple', color: 'green' };
let {fruit, color} = {...object};
console.log(`fruit: ${fruit}, color: ${color}`);

let color = "red";
console.log(`object color: ${object.color}, color: ${color}`); 
```
We first create an object with two properties, fruit and color. Then we use the spread operator to expand the object into variables and log them to the console. The variables’ names are the same as the object properties’ names. However, we can now access the values directly from the variables instead of referring to the object. We do so in the template literal and see fruit: apple, color: green as the console output.

Also, as these variables allocate their own memory, they are complete clones. Therefore, modifying the variable color to red won’t change the original value: object.color still returns green when we log both variables to the console.

Using the spread operator to clone an array or object is useful because JavaScript treats arrays as references to its values. When you assign an array or object to a new variable or constant, this merely copies the reference to the original; it does not clone the array or object by allocating memory. Therefore, changing the copy also changes the original. Using spread instead of the equals operator (=) allocates memory and keeps no reference to the original value. Hence, it’s an excellent solution for cloning an array or object, as shown in [Listing 2-20](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-20).

```javascript
let originalArray = [1,2,3];
let clonedArray = [...originalArray];

clonedArray[0] = "one";
clonedArray[1] = "two";
clonedArray[2] = "three";

console.log(`originalArray: ${originalArray}, clonedArray: ${clonedArray}`);

```
Here we use the spread operator to copy the values from the original array to the cloned array in the same operation. Then we modify the cloned array’s items. Finally, we write the two arrays to the console and see that the original array differs from the cloned array.

## Exercise 2: Extending JavaScript with Express

Modern JavaScript provides the tools you need to write clean and efficient code. In [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml), you’ll use it in the Food Finder application. For now, let’s apply your new knowledge to optimize the simple Express.js server you created in [Chapter 1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter1.xhtml).

#### Editing the package.json File

We’ll replace the server’s require call with named modules for different routes. To do so, we need to explicitly specify that our project uses native modules. Otherwise, Node.js will throw an error. Modify your _package.json_ file so that it looks like [Listing 2-21](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-21).

```
{
    "name": "sample-express",
    "version": "1.0.0",
    "description": "sample express server",
    "license": "ISC",
    "type": "module",
    "dependencies": {
        "express":"^4.18.2",
        "node-fetch": "^3.2.6"
    },
    "devDependencies": {}
}
```

Listing 2-21: The modified package.json file

Add the property type with the value module. Also, you’ll want to install the _node-fetch_ package to make an asynchronous API call in one of your routes. Run npm install node-fetch to do so.

#### Writing an ES.Next Module with Asynchronous Code

Create the file _routes.js_ in the _sample-express_ folder, next to the _index.js_ file, and add the code in [Listing 2-22](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-22).

```
import fetch from "node-fetch";

const routeHello = () => "Hello World!";

const routeAPINames = async () => {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data;
    try {
        const response = await fetch(url);
        data = await response.json();
    } catch (err) {
        return err;
    }
    const names = data
        .map((item) => `id: ${item.id}, name: ${item.name}`)
        .join("<br>");
    return names;
};

export {routeHello, routeAPINames};
```

Listing 2-22: The route module in the routes.js file

First we import the fetch module for making asynchronous requests. Then we create the first route, for our existing _/hello_ endpoint. Its behavior should be the same as before; using a fat arrow function with a concise body syntax, it returns the string Hello World!

Next, we create a route for a new _/api/names_ endpoint. This endpoint will add a page to our web server displaying a list of usernames and IDs. But first we explicitly define an async function so that we can use the await syntax for our fetch call. Then we define the API endpoint in a constant and another variable to store asynchronous data. We need to define these before we use them because the await calls happen inside a try...catch block, and these variables are block-scoped. If we defined them inside the block, we wouldn’t be able to use them later.

We call the API and await the response data, which we convert to JSON as soon as the call succeeds. The data variable now holds an array of objects. We use array.map to iterate over the data and create the strings we want to display. Then we join all array items with break tags (<br>) to display them in rows and return the string.

Finally, we export the two routes under their names.

#### Adding the Modules to the Server

Modify the file _index.js_ in the _sample-express_ folder to match [Listing 2-23](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-23). We use native modules for importing the require module and the routes we created in [Listing 2-22](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#Lis2-22).

```
import {routeHello, routeAPINames} from "./routes.js";
import express from "express";

const server = express();
const port = 3000;

server.get("/hello", function (req, res) {
    const response = routeHello(req, res);
    res.send(response);
});

server.get("/api/names", async function (req, res) {
    let response;
    try {
        response = await routeAPINames(req, res);
    } catch (err) {
        console.log(err);
    }
    res.send(response);
});

server.listen(port, function () {
    console.log("Listening on " + port);
});
```

Listing 2-23: The basic Express.js server with modern JavaScript

First we import routes with the syntax for named imports. Then we replace the require call for the _express_ package with an import statement. The _/hello_ endpoint we created earlier calls the route we imported, and the server sends Hello World! as the response to the browser.

Finally, we create a new endpoint, _/api/names_, that contains asynchronous code. Therefore, we mark the handler as _async_ and await the route inside a try...catch block.

Start the server from your command line:

```
$ node index.js
Listening on 3000
```

Now visit _http://localhost:3000/api/names_ in your browser, as shown in [Figure 2-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter2.xhtml#fig2-1).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure2-1.jpg)

Figure 2-1: The response the browser receives from the Node.js web server

You should see the new list of user IDs and names.

### Summary

This chapter taught you enough modern JavaScript and ES.Next to create a full-stack application. We covered how to use JavaScript modules to create maintainable packages and import and export code, the different ways to declare variables and constants, the arrow function, and tagged and untagged template literals. We wrote asynchronous code with promises and async/await. We also covered array.map, the spread operator, and their usefulness for your full-stack code. Finally, you used your new knowledge to update the sample Node.js server from [Chapter 1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter1.xhtml) with modern JavaScript concepts.

Modern JavaScript has many more features than this chapter covers. From the freely available resources, I recommend the JavaScript tutorials at [_https://www.javascripttutorial.net_](https://www.javascripttutorial.net/).

In the next chapter, we cover TypeScript, a superset of JavaScript with support for types.