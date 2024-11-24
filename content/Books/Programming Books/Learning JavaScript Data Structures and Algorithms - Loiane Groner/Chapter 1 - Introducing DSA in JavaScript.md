---
id: 01JCKZD1K7XP43TA4QPTY8G5RA
modified: 2024-11-13T18:59:13-05:00
title: Chapter 1 - Introducing DSA in JavaScript
---
# 1 Introducing Data Structures and Algorithms in JavaScript

**Before you begin: Join our book community on Discord**

Give your feedback straight to the author himself and chat to other early readers on our Discord server (find the "learning-javascript-dsa-4e" channel under EARLY ACCESS SUBSCRIPTION).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file0.png)

[https://packt.link/EarlyAccess/](https://packt.link/EarlyAccess/)

**JavaScript** is an immensely powerful language. It is one of the most popular languages in the world and is one of the most prominent languages on the internet. For example, GitHub (the world's largest code host, available at [https://github.com](https://github.com/)) hosts over 300,000 JavaScript repositories at the time of writing (the largest number of active repositories available on GitHub are in JavaScript; refer to [http://githut.info](http://githut.info/)). The number of projects in JavaScript on GitHub grows every year.

JavaScript is an essential skill for any web developer. It offers a convenient environment for learning data structures and algorithms, requiring only a text editor or browser to get started. More importantly, JavaScript's widespread use in web development allows you to directly apply this knowledge to build efficient, scalable web applications, optimizing performance and handling complex tasks.

Data structures and algorithms are fundamental building blocks of software development. Data structures provide ways to organize and store data, while algorithms define the operations performed on that data. Mastering these concepts is crucial for creating well-structured, maintainable, and high-performing JavaScript code.

In this chapter, we'll cover the essential JavaScript syntax and functionalities needed to start building our own data structures and algorithms. Additionally, we'll introduce TypeScript, a language that builds upon JavaScript and offers enhanced code safety, structure, and tooling. This will enable us to create data structures and algorithms using both JavaScript and TypeScript, showcasing their respective strengths. We will cover:

- The importance of data structures
- Why algorithms matter
- Why companies ask for these concepts during interviews
- Why choose JavaScript to learn data structures and algorithms
- Setting up the environment
- JavaScript fundamentals
- **TypeScript** fundamentals

## The importance of data structures

A data structure is a way to organize and store data in a computer's memory, enabling the data to be efficiently accessed and modified.

Think about data structures as containers designed to hold specific types of information, that have their own way of arranging and managing the information. For example, in your home, you cook food in the kitchen, sleep in the bedroom, and take a shower in the bathroom. Each place in a house or apartment is designed with a specific goal for certain tasks so we can keep our home organized.

In the real world, data can often be overly complex, due to factors such as volume, various forms (numbers, dates, text, images, emails), and the speed that data is generated, among other factors. Data structures bring order to this chaos, allowing computers to handle vast amounts of information systematically and efficiently. Think of it like a well-organized library compared to a very large pile of books. Finding a specific book is easier in the library as the books are organized by genre and by author (alphabetically).

Good data structure choices help programs perform consistently regardless of the amount of data being processed. Imagine needing to store data for the weather forecast for 10 days versus 10 years. The use of the correct data structure can make a difference between an algorithm crashing or being able to scale.

And finally, a data structure can drastically impact how easy it is to write algorithms that work with that data. For example, finding the shortest route on a map can be efficiently solved using a graph data structure that shows which cities are connected by what distances, rather than an unordered array of city names.

## Why algorithms matter

Computers are powerful tools, but their intelligence is derived from the instructions we provide. Algorithms are the sets of rules and procedures that guide a computer's actions, enabling it to solve problems, make decisions, and perform complex tasks. In essence, algorithms are the language through which we communicate with computers, transforming them from mere machines into intelligent problem-solvers.

Algorithms can turn tasks into repeatable and automated processes. If you need to generate a report every day at work, this is a task that can be automated using an algorithm.

Algorithms are around us every day, from search engines to social networks, to self-driving cars. Algorithms and data structures are what make them function. Understanding data structures and algorithms unlocks the ability to create and innovate in the technology world.

As software developers, writing algorithms and manipulating data are core aspects of our work. This is precisely why companies emphasize these concepts in job interviews – they are essential skills for assessing a candidate's problem-solving abilities and their potential to contribute effectively to software development projects.

## Why companies ask for these concepts during interviews

There are many reasons why companies focus on data structures and algorithms concepts during job interviews, even if you are not going to use some of these concepts during daily tasks, including:

- Problem-solving skills: data structures and algorithms are a great tool to evaluate the candidate's problem-solving abilities. They can be used to evaluate how a person approaches unfamiliar problems, breaks them down into smaller tasks and designs a solution.
- Coding proficiency: companies can evaluate how candidates translate their solution into clean and efficient code, how candidates choose the appropriate data structure for the problem, design an algorithm with the correct logic, consider edge cases and optimize their code.
- Software performance: a strong understanding of data structures and algorithms translates directly to successful delivery of every development task, such as designing scalable solutions by choosing the right data structure and algorithms, especially when dealing with large datasets. In the realm of Big Data, where you're likely dealing with massive datasets, it is crucial that your solution not only functions correctly but also operates efficiently at scale. Performance optimization often relies on selecting the optimal data structure or tweaking existing algorithms.
- Debugging and troubleshooting: A solid grasp of data structures and algorithms can help engineers pinpoint where issues might occur within their code.
- Ability to learn and adapt: technology is always evolving, as well as programming languages and frameworks, however, data structures and algorithms concepts remain fundamental throughout the years. That is one of the reasons we often say these concepts are part of the basic knowledge about computer science. And once you learn the concepts in one language, you can easily adapt to a different programming language. This helps companies test if a person can adapt to changing requirements, which is essential in this industry.
- Communication: usually, when companies present a problem to be solved using data structures and algorithms, they are not looking for the eventual answer, but the process that the candidate follows to get to the definitive answer. Companies can evaluate how candidates are able to explain their thought process and reason behind their decision-making approach; and the candidate's ability to discuss different trade-offs involved in choosing different data structures and algorithms to resolve the program. This can be used to evaluate if a person can collaborate within a team setting and if the candidate can clearly communicate a message.

Of course, these are only some of the factors that companies will evaluate, and domain-specific knowledge, experience and cultural fit are also crucial factors when choosing the right candidate for a job position.

## Why choose JavaScript to learn data structures and algorithms?

JavaScript is one of the most popular programming languages in the world, according to various industry surveys, making it an excellent choice if you are already familiar with the basics of programming. The thriving JavaScript community and the abundance of online resources create a supportive and dynamic environment for learning, collaborating, and advancing your career as a JavaScript developer.

JavaScript is also a beginner friendly language and you do not need to worry about complex memory management concepts that exist in other languages such as C++. This is extremely helpful especially when learning data structures like linked lists, trees, and graphs, which are dynamic data structures due to their ability to grow or shrink in size during program execution (runtime), and when using JavaScript, you can focus on the data structure concepts, without mixing with memory management controls.

As JavaScript is used for web development, learning data structures and algorithms with JavaScript allows you to directly apply your skills to building interactive web applications.

However, there is one big disadvantage of using JavaScript: the lack of strict typing that exists in other languages such as C++ and Java. JavaScript is a dynamically typed language, meaning you do not need to explicitly declare the data type of variables. When working with data structures, we need to pay attention to not mix data types within the same data structure as it can lead to subtle errors. Typically, when working with data structures, it is considered best practice to ensure all data within the same structure is of the same type. We will take care of this gap by always using the same data type for the same data structure instance throughout this book and we will also resolve the lack of strict typing by providing the source code in **TypeScript**, which extends JavaScript by adding types to the language.

And it is important to remember: the best language is the one you are most comfortable using and that motivates you to learn. This book will present different data structures and algorithms using JavaScript and TypeScript, and you can always adapt the concepts to another programming language as well.

## Setting up the environment

One of the pros of the JavaScript language compared to other languages is that you do not need to install or configure a complicated environment to get started with it. To follow the examples in this book, you will need to download Node.js from [https://nodejs.org](https://nodejs.org/) so we can execute the source code. On the download page, you will find detailed steps to download and install Node.js in your operating system.

As a rule of thumb, always download the _LTS_ (_Long Term Support_) version, which is often used by enterprise companies.

While JavaScript can run in both browsers and Node.js, the latter provides a more streamlined and focused environment for studying data structures and algorithms. Node.js eliminates browser-specific complexities, offers powerful debugging tools, and facilitates a more direct approach to learning these core concepts.

The source code for this book is also available in TypeScript, which offers enhanced type safety and structure. To run TypeScript code, including the examples in this book, we'll need to transpile it into JavaScript, a process we will cover in detail.

### Installing a code editor or IDE

We also need an editor or **IDE** (_Integrated Development Environment_) to be able to develop an application in a comfortable environment. For the examples in this book, the author used **Visual Studio Code** (**VSCode**), a free and open-source editor. However, you can use any editor of your choice (Notepad++, WebStorm, and other editors or IDEs available on the market).

> You can download the **VSCode** installer for your operating system at [https://code.visualstudio.com](https://code.visualstudio.com/).

Now that we have everything we need, we can start coding our examples!

## JavaScript Fundamentals

Before we start diving into the various data structures and algorithms, let's have a quick overview of the JavaScript language. This section will present the JavaScript fundamental concepts required to implement the algorithms we will create in the subsequent chapters.

### Hello World

We will begin with the classic "Hello, World!" example, a simple program that displays the message "Hello, World!".

Let's create our first example together. Follow these steps:

1. Create a folder named `javascript-datastructures-algorithms`.
2. Inside it, create a folder named `src` (source, where we will create our files for this book).
3. Inside the `src` folder, create a folder named `01-intro`

We can place all the examples for this chapter inside this directory. Now let's create a `Hello, World` example. To do so, create a file named `01-hello-variables.js`. Inside the file, add the code below:

```
console.log('Hello, World!');
```

To run this example, you can use the default operating system terminal or command prompt (or in case you are using Visual Studio Code, open the built-in terminal) and execute the following command:

```
node src/01-intro/01-hello-variables.js
```

You will see the "`Hello, World!`" output, as shown in the image below:

![Figure 1.1 – Visual Studio Code with JavaScript Hello, World! example](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file1.png)

Figure 1.1 – Visual Studio Code with JavaScript Hello, World! example

> For every source file for this book, you will see in the first line the path of the file, followed by the source code we will create together, and the file will end with the command we can use to see the output in the terminal.

### Variables and data types

There are three ways of declaring a variable in JavaScript:

1. `var`: declares a variable, and it is optional to initialize it with a value. This is the oldest way to declare variables in JavaScript.
2. `let`: declares a local variable, block-scoped (this means the variable is only accessible within the specific block of code such as inside a loop or conditional statement), and it is also optional to initialize it with a value. For our algorithms, this will be our preferred way due to its more predictable behavior.
3. `const`: declares a read-only constant. It is mandatory to initialize it, and we will not be able to re-assign it.

Let's see a few examples:

```
var num = 1;
num = 'one' ;
let myVar = 2;
myVar = 4;
const price = 1.5;
```

Where:

1. In the first line, we have an example of how to declare a variable in JavaScript (the legacy way, before modern JavaScript). Although it is not necessary to use the `var` keyword declaration, it is a good practice to always specify when we declare a new variable.
2. In the second line, we updated an existing variable. JavaScript is not a strongly typed language. This means you can declare a variable, initialize it with a number, and then update it with a text or any other datatype. Assigning a value to a variable that is different from its original type is often not considered a good practice, although it is possible.
3. In the third line, we also declared a number, but this time we are using the `let` keyword to specify this is a local variable.
4. In the fourth line, we can change the value of `myVar` to a different number;
5. In the fifth line, we declared another variable, but this time using the `const` keyword. This means the value of this variable is final and if we try to assign another value, we will get an error (_assignment to constant variable_).

Let's see next what datatypes are supported by JavaScript.

#### Data types

- The latest **ECMAScript** standard (the JavaScript specification) defines a few primitive data types at the time of writing:
    - **Number**: an integer or floating number;
    - **String**: a text value;
    - **Boolean**: true or false values;
    - **null**: a special keyword denoting a null value;
    - **undefined**: a variable with no value or that has not been initialized;
    - **Symbol**, which are unique and immutable;
    - **BigInt**: an integer with arbitrary precision: `1234567890n`;
    - and **Object**.

Let's see an example of how to declare variables that hold different data types:

```
const price = 1.5; // number
const publisher = 'Packt'; // string
const javaScriptBook = true; // boolean
const nullVar = null; // null
let und; // undefined
```

If we want to see the value of each variable we declared, we can use `console.log` to do so, as listed in the following code snippet:

```
console.log('price: ' + price);
console.log('publisher: ' + publisher);
console.log('javaScriptBook: ' + javaScriptBook);
console.log('nullVar: ' + nullVar);
console.log('und: ' + und);
```

> The console.log method also accepts more than one argument. Instead of `console.log('num: ' + num)`, we can also use `console.log('num: ', num)`. While the first option will concatenate the result into a single string, the second allows us to add a description and visualize the variable content in case it is an object.

In JavaScript, the `typeof` operator is an operator that helps to determine the data type of a variable or expression. It returns a string that represents the type of the operand. In case we would like to check the type of the declared variables, we can use the following code:

```
console.log('typeof price: ', typeof price); // number
console.log('typeof publisher: ', typeof publisher); // string
console.log('typeof javaScriptBook: ', typeof javaScriptBook); // boolean
console.log('typeof nullVar: ', typeof nullVar); // object
console.log('typeof und: ', typeof und); // undefined
```

> In JavaScript, `typeof null` returns "object", which can be confusing since `null` is not actually an object. This is considered a historical quirk or bug in the language.

##### The object and symbol data types

In JavaScript, an object is a fundamental data structure that serves as a collection of _key-value_ pairs. These key-value pairs are often referred to as properties or methods of the object. Think of it as a container that can store various types of data and functionality.

If we want to represent a book in JavaScript with attributes like its title, we can effectively do so using an object:

```
const book = {
  title: 'Data Structures and Algorithms',
}
```

Objects are a cornerstone of JavaScript programming, providing a powerful way to structure data, encapsulate logic, and model real-world entities.

If we would like to output the title of the book, we can do so using the dot notation as follows:

```
console.log('book title: ', book.title);
```

Although we are not able to reassign values to a constant, we can _mutate_ it if its data type is an object. Let's see an example:

```
book.title = 'Data Structures and Algorithms in JavaScript';
// book = {anotherTitle:'Data Structures'} this will not work
```

Mutaing an object means changing the properties within the existing object as we just did. Reassigning an object means changing the entire object that a variable refers to as showed in the following example:

```
let book2 = {
  title: 'Data Structures and Algorithms',
}
book2 = { title: 'Data Structures' };
```

This concept is important, as we will use it in a lot of examples.

JavaScript also supports a special and unique data type called **symbol**. Symbols serve as unique identifiers and are primarily used for the following purposes:

- Unique property keys: symbols can be used as keys for object properties, ensuring that these properties are distinct and will not clash with any other keys (even strings with the same name). This is particularly useful when working with libraries or modules where naming conflicts might arise.
- Hidden properties: symbols are not enumerable by default, meaning they will not show up in `for...in` loops or `Object.keys()`. This makes them ideal for creating _private_ properties within objects.

Let's see an example for better understanding:

```
const title = Symbol('title');
const book3 = {
  [title]: 'Data Structures and Algorithms'
};
console.log(book3[title]); // Data Structures and Algorithms
```

In this example, we are creating a symbol `title` and using it as a key for the `book3` object. When we need to access this property, we cannot simply use `book3.title` using the dot notation, but `book3.[title]`, using the brackets notation.

We will not use symbols in the examples of this book, but it is an interesting concept to know.

### Control structures

JavaScript has a similar set of control structures as the C and Java languages. Conditional statements are supported by `if...else` and `switch`. JavaScript also supports different loop statements such as `for`, `while`, `do…while`, `for…in` and `for…of`.

In this section, we explore both conditional statements and loop statements, starting with conditional statements.

#### Conditionals

Conditional statements in JavaScript are essential building blocks for controlling the flow of your code. They allow you to make decisions based on certain conditions and execute different code blocks accordingly.

We can use the `if` statement if we want to execute a block of code only if the condition is true. And we can use the optional `else` statement if the condition is false:

```
let number = 0;
if (number === 1) {
  console.log('number is equal to 1');
} else {
  console.log('number is not equal to 1, the value of number is ' + number);
}
```

The `if...else` statement can also be represented by a ternary operator. For example, take a look at the following if...else statement:

```
if (number === 1) {
  number--;
} else {
  number++;
}
```

It can also be represented as follows:

```
number === 1 ? number-- : number++;
```

Ternary operators are expressions that evaluate to a value, whereas the if-statement is just an imperative statement. This means we can use it directly within another expression, assign its result to a variable, or use it as an argument in a function call. For example, we can rewrite the previous example as following, without changing its output:

```
number = number === 1 ? number - 1 : number + 1;
```

Also, if we have several scripts, we can use `if...else` several times to execute different scripts based on different conditions, as follows:

```
let month = 5;
if (month === 1) {
  console.log('January');
} else if (month === 2) {
  console.log('February');
} else if (month === 3) {
  console.log('March');
} else {
  console.log('Month is not January, February or March');
}
```

Finally, we have the `switch` statement. The `switch` statement provides an alternative way to write multiple `if...else` if chains. It evaluates an expression and then matches its value against a series of cases. When a match is found, the code associated with that case is executed:

```
switch (month) {
  case 1:
    console.log('January');
    break;
  case 2:
    console.log('February');
    break;
  case 3:
    console.log('March');
    break;
  default:
    console.log('Month is not January, February or March');
}
```

JavaScript will look for a match of the value (in this example, the variable `month`) in one of the `case` statements. If no match is found, then the `default` statement is executed. The `break` clause inside each `case` statement will halt the execution and break out of the `switch` statement. If we do not add the `break` statement, the code inside each subsequent `case` statement is also executed, including the `default` statement.

#### Loops

In JavaScript, loops are fundamental structures that allow you to repeatedly execute a block of code as long as a specified condition is `true`. They are essential for automating repetitive tasks and iterating over collections of data like arrays or objects.

The `for` loop is the same as in C and Java. It consists of a loop counter that is usually assigned a numeric value, then the variable is compared against the condition to break the loop, and finally the numeric value is increased or decreased.

In the following example, we have a `for` loop. It outputs the value of `i` in the console, while `i` is less than `10`. `i` is initiated with 0, so the following code will output the values 0 to 9:

```
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

In the `for` loop, first we have the initialization (`let i = 0`), which happens only once, before the loop starts. Next, we have the condition that is evaluated before each iteration (`i < 10`). If it evaluates to `true`, the loop's body is executed. If it is `false`, the loop ends. We then have the final expression, (`i++`), which is often used to update the counter variable. The final expression is executed after the body of the loop is executed.

The next loop construct we will look at is the `while` loop. The script inside the while loop is executed while the condition is true.

In the following code, we have a variable `i`, initiated with the value 0, and we want the value of `i` to be logged while `i` is less than 10 (or less than or equal to 9). The output will be the values from 0 to 9:

```
let i = 0;
while (i < 10) {
  console.log(i);
  i++;
}
```

The `do...while` loop is similar. The only difference is that in the `while` loop, the condition is evaluated before executing the script, and in the `do...while` loop, the condition is evaluated after the script is executed. The `do...while` loop ensures that the script is executed at least once. The following code also outputs the values from 0 to 9:

```
i = 0;
do {
  console.log(i);
  i++;
} while (i < 10);
```

The `for…in` loop iterates a variable over the properties of an object. This loop is especially useful when working with dictionaries and sets.

In the following code, we will declare an object, and output the name of each property, along with its value:

```
const obj = { a: 1, b: 2, c: 3 };
for (const key in obj) {
  console.log(key, obj[key]);
}
// output: a 1 b 2 c 3
```

The `for…of` loop iterates a variable over the values of an Array, Map or Set as shown by the following code:

```
const arr = [1, 2, 3];
for (const value of arr) {
  console.log(value);
}
// output: 1 2 3
```

### Functions

Functions are important when working with JavaScript. Functions are blocks of code designed to perform specific tasks. They are one of the fundamental building blocks in JavaScript and offer a way to organize code, promote reusability, and improve maintainability. We will also use functions in our examples.

The following code demonstrates the basic syntax of a function:

```
function sayHello(name) {
  console.log('Hello! ', name);
}
```

The purpose of this function is to create a reusable piece of code that greets a person by their name. We have one parameter named `name`. A parameter acts as a placeholder for a value that will be provided when the function is called.

To execute this code, we simply use the following statement:

```
sayHello('Packt');
```

The string "Packt" is passed as an argument. This value is assigned to the `name` parameter inside the function.

A function can also return a value, as follows:

```
function sum(num1, num2) {
  return num1 + num2;
}
```

> Note that in JavaScript, a function will always return a value. A function can explicitly return a value using the `return` keyword followed by an expression.
> 
> > If a function doesn't have a `return` statement (as in the `sayHello` example above) or reaches the end of its code block without encountering a `return`, it implicitly returns `undefined`.

This function calculates the sum of two given numbers and returns its result. We can use it as follows:

```
const result = sum(1, 2);
console.log(result); // outputs 3
```

We can also assign default values to the parameters. In case we do not pass the value, the function will use the default value:

```
function sumDefault(num1, num2 = 2) { // num2 has a default value
    return num1 + num2;
}
console.log(sumDefault(1)); // outputs 3
console.log(sumDefault(1, 3)); // outputs 4
```

We will explore more about functions throughout this book, as well as other advanced features related to functions, especially when covering the algorithms section.

#### Variable scope

The scope refers to where in the algorithm we can access the variable. To understand how variables scope work, let's use the following example:

```
let movie = 'Lord of the Rings';
function starWarsFan() {
  const movie = 'Star Wars';
  return movie;
}
function marvelFan() {
  movie = 'The Avengers';
  return movie;
}
```

Now let's log some output so we can see the results:

```
console.log(movie); // Lord of the Rings
console.log(starWarsFan()); // Star Wars
console.log(marvelFan()); // The Avengers
console.log(movie); // The Avengers
```

Following is the explanation of why we got this output:

- We start by declaring a variable named `movie` in the global scope and assigning it the value "Lord of the Rings". This variable can be accessed from anywhere in the code. The first `console.log` statement is printing the initial value of this variable.
- For the `starWarsFan` function, we declared a new variable named `movie` using `const`. This variable has the same name as the global variable `movie`, but it exists only within the function's scope. This is called "**shadowing**", where the local variable hides the global one within the function's context. The second `console.log` is calling the `starWarsFan` function. It prints "Star Wars" because inside the function we are working with the local variable `movie`, leaving the global `movie` variable unchanged.
- Next, we have the marvelFan function, which does not declare a new `movie` variable using `let` or `const`. Instead, it directly modifies the global `movie` variable by assigning it the value "The Avengers". This is possible because there is no local variable to shadow the global one. So, we print the third console.log calling this function, the output is "The Avengers".
- Finally, we have the last `console.log(movie)`. This again prints the global `movie` variable, which now holds the value "The Avengers" due to the previous `marvenFan` function call.

Let's review a second example, this time using only a function, to showcase how variable scope affects the visibility and value of variables within different parts of the code:

```
function blizzardFan() {
  const isFan = true;
  let phrase = 'Warcraft';
  console.log('Before if: ' + phrase);
  if (isFan) {
    let phrase = 'initial text';
    phrase = 'For the Horde!';
    console.log('Inside if: ' + phrase);
  }
  phrase = 'For the Alliance!';
  console.log('After if: ' + phrase);
}
```

When we call the blizzardFan() function, the output will be:

```
Before if: Warcraft
Inside if: For the Horde!
After if: For the Alliance!
```

Let's understand why:

1. We start by declaring a constant variable `isFan` and initializing it with the value `true`. Since it is declared with `const`, its value cannot be changed.
2. A variable `phrase` is declared using `let` and assigned the value 'Warcraft'.
3. Next, we have the first console.log that outputs the current value of the phrase variable which is Warcraft.
4. Then we have the `if (isFan)` block. Since `isFan` is `true`, the code inside the if block is executed.
5. Inside the if block, we declare a new variable, also named `phrase`, within the if block's scope. This creates a separate, block-scoped variable that shadows the outer `phrase` variable.
6. The value of the inner `phrase` variable (the one declared within the if block) is changed to "For the Horde!".
7. `console.log('Inside if: ' + phrase)` prints "Inside if: For the Horde!" because this is the inner `phrase` variable.
8. After the if block, the outer `phrase` variable (the one declared at the beginning of the function) is still accessible. Its value is changed to "For the Alliance!".
9. Finally, `console.log('After if: ' + phrase)` prints "After if: For the Alliance!" because we are printing the variable we declared in the first line of the function.

Now that we know the basics of the JavaScript language, let's see how we can use it in an Object-oriented programming approach.

### Object-oriented programming in JavaScript

**Object-Oriented Programming (OOP)** in JavaScript consist of five concepts:

1. Objects: these are the fundamental building blocks of OOP. They represent real-world entities or abstract concepts, encapsulating both data (properties) and behavior (methods).
2. Classes: these are a more structured way to create objects. A class serves as a blueprint for creating multiple objects (instances) of a similar type.
3. Encapsulation: this involves bundling data and the functions that operate on that data into a single unit (the object). It protects the object's internal state and allows you to control access to its properties and methods.
4. Inheritance: this allows us to create new classes (child classes) that inherit properties and methods from existing classes (parent classes). This promotes code reusability and establishes relationships between classes.
5. Polymorphism: this means _many forms_. In OOP, it refers to the ability of objects of different classes to respond to the same method call in their own unique way.

OOP helps organize code, promotes reusability, makes code more maintainable, and allows for better modeling of real-world relationships. Let's review each concept to understand how they work in JavaScript.

#### Objects, classes and encapsulation

JavaScript objects are simple collections of name-value pairs.

There are two ways of creating a simple object in JavaScript. An example of the first way is as follows:

```
let obj = new Object();
```

And an example of the second way is as follows:

```
obj = {};
```

The example of the second way is called an object literal, which is a means to create and define objects directly in the code using a convenient notation. It is one of the most common ways to work with objects in JavaScript and also the preferred way over the `new Object` constructor in the first example, due to convenience (compact syntax), and overall performance when creating objects.

We can also create an object entirely, as follows:

```
obj = {
  name: {
    first: 'Gandalf',
    last: 'the Grey'
  },
  address: 'Middle Earth'
};
```

To declare a JavaScript object, _[key, value]_ pairs are used, where the key can be considered a property of the object and the value is the property value. In the previous example, `address` is the key, and its value is "Middle Earth". We will use this concept when creating some data structures, such as Sets or Dictionaries.

Objects can contain other objects as their properties. We call them nested objects. This creates a hierarchical structure where objects can be nested within each other at multiple levels, as we can see in the previous example, where `name` is a nested object within `obj`.

In Object-Oriented Programming (OOP), an object is an instance of a class. A class defines the characteristics of the object and helps us with encapsulation, bundling the properties and methods so they can work as one unit (an object). For our algorithms and data structures, we will create some classes that will represent them. This is how we can define a class that represents a book:

```
class Book {
  #percentagePerSale = 0.12;
  constructor(title, pages, isbn) {
    this.title = title;
    this.pages = pages;
    this.isbn = isbn;
  }
  get price() {
    return this.pages * this.#percentagePerSale;
  }
  static copiesSold = 0;
  static sellCopy() {
    this.copiesSold++;
  }
  printIsbn() {
    console.log(this.isbn);
  }
}
```

We can declare properties in a JavaScript class through the constructor. JavaScript will automatically declare a property that is public, meaning it can be accessed and modified directly.

Inside the constructor, `this` refers to the object instance being created. In the case of our example, this is referencing itself. We could interpret the code as this book's title is being assigned the title value passed to the constructor.

Modern JavaScript also allows us to declare private properties by adding the prefix `#` as in `#percentagePerSale`. This property is only visible inside the class and cannot be accessed directly.

> Public members (properties and methods) are accessible from anywhere, both inside and outside the class. By default, all members in a JavaScript class are public.
> 
> > Private members are accessible only from within the class itself. They cannot be accessed or modified directly from outside the class, providing better encapsulation and data protection.

We can also create getters using the `get` keyword (`get price()`). These can be used to declare a property that returns a calculated value based on the object's other properties. In this case, the price of the book depends on the number of `pages` (which is a public property) and the percentage of profit per sale, which is private. We access a property locally inside the class by referencing the keyword `this`.

We can also declare methods in a class (`printIsbn`). Methods are simply functions that are associated with the class. They define the actions or behaviors that objects created from the class (instances) can perform.

Modern JavaScript also allows us to declare static properties using the `static` keyword and static methods. Static properties are shared between all instances of the class, which is a fantastic way to keep track of properties that are shared between every object in the class (such as a count of how many books have been sold in total, for example). In other languages such as Ruby, they are classes class variables.

Static methods do not require an instance of the class and can be accessed directly, such as `Book.sellCopy()`.

To instantiate this class, we can use the following code:

```
let myBook = new Book('title', 400, 'isbn');
```

Then, we can access its public properties and update them as follows:

```
console.log(myBook.title); // outputs the book title
myBook.title = 'new title'; // update the value of the book title
console.log(myBook.title); // outputs the updated value: new title
```

We can use the getter method to find out the price of the book as follows:

```
console.log(myBook.price); // 48
```

And access the static property and method as follows:

```
console.log(Book.copiesSold); // 0
Book.sellCopy();
console.log(Book.copiesSold); // 1
Book.sellCopy();
console.log(Book.copiesSold); // 2
```

What if we would like to represent another type of book, such as an e-book? Can we reuse some of the definitions we have declared in the `Book` class? Let's find out in the next section.

#### Inheritance and polymorphism

JavaScript also allows the use of inheritance, which is a powerful mechanism in OOP that allows us to create new classes (child class) that derive properties and methods from existing classes (parent class or superclass). Let's look at an example:

```
class Ebook extends Book {
  constructor(title, pages, isbn, format) {
    super(title, pages, isbn);
    this.format = format;
  }
  printIsbn() {
    console.log('Ebook ISBN:',this.isbn);
  }
}
Ebook.sellCopy();
console.log(Ebook.copiesSold); // 3
```

We can extend another class and inherit its behavior using the keyword `extends`. In our example, the Ebook is the child class, and the Book is the superclass.

Inside the `constructor`, we can refer to the constructor `superclass` using the keyword `super`. We can add more properties to the child class (`format`). The child class can still access static methods (`sellCopy`) and properties (`copiesSold`) from the superclass.

A child class can also provide its own implementation of a method initially declared in the superclass. This is called _method overriding_ and allows objects of the child class to exhibit different behavior for the same method call.

By overriding methods from the superclass, we can achieve a concept called polymorphism, which literally means many forms. In OOP, polymorphism is the ability of objects of different classes to respond to the same method call in their own unique way. For example:

```
const myBook = new Book('title', 400, 'isbn');
myBook.printIsbn(); // isbn
const myEbook = new Ebook('DS Ebook', 401, 'isbn 123', 'pdf');
myEbook.printIsbn(); // Ebook ISBN: isbn 123
```

We have two book instances here, and one of them is an e-book. We can call the method `printIsbn` from both instances, and we will get different outputs due to the different behavior in each instance.

Most of the data structures we will cover throughout this book will follow the JavaScript class approach.

> Although the class syntax in JavaScript is remarkably similar to other programming languages such as Java and C/C++, it is good to remember that JavaScript object-oriented programming is done through a prototype.

### Modern Techniques

JavaScript is a language that continues to evolve and get new features each year. There are some features that make some concepts easier when working with data structures and algorithms. Let's check them out.

#### Arrow functions

Arrow functions are a concise and expressive way to write functions in JavaScript. They offer a shorter syntax and some key differences in behavior compared to traditional function expressions. Consider the following example:

```
const circleAreaFn = function(radius) { 
  const PI = 3.14; 
  const area = PI * radius * radius; 
  return area; 
};
console.log(circleAreaFn(2)); // 12.56
```

With arrow functions, we can simplify the syntax of the preceding code to the following code:

```
const circleArea = (radius) => {
  const PI = 3.14; 
  return PI * radius * radius; 
};
```

The main difference is in the first line of the example, on which we can omit the keyword `function` using `=>`, hence the name arrow function.

If the function has a single statement, we can use a simpler version, by omitting the keyword `return` and the curly brackets as demonstrated in the following code snippet:

```
const circleAreaSimp = radius => 3.14 * radius * radius;
console.log(circleAreaSimp(2)); // 12.56
```

If the function does not receive any argument, we can use empty parenthesis as follows:

```
const hello = () => console.log('hello!');
hello(); // hello!
```

We will be using arrow functions to code some algorithms later in this book for a simpler syntax.

#### Spread and rest operators

In JavaScript, we can turn arrays into parameters using the `apply()` function. Modern JavaScript has the spread operator (`...`) for this purpose. For example, consider the function `sum` as follows:

```
const sum = (x, y, z) => x + y + z;
```

We can execute the following code to pass the `x`, `y`, and `z` parameters:

```
const numbers = [1, 2, 3];
console.log(sum(...numbers)); // 6
```

The preceding code is the same as the code written in classic JavaScript, as follows:

```
console.log(sum.apply(null, numbers));
```

The spread operator (`...`) can also be used as a rest parameter in functions to replace `arguments`. Consider the following example:

```
const restParamaterFunction = (x, y, ...a) => (x + y) * a.length;
console.log(restParamaterFunction(1, 2, 'hello', true, 7)); // 9
```

The preceding code is the same as the following (also outputs 9 in the console):

```
function restParamaterFunction(x, y) {
  const a = Array.prototype.slice.call(arguments, 2);
  return (x + y) * a.length;
}
console.log(restParamaterFunction(1, 2, 'hello', true, 7));
```

The rest and spread operators are going to be useful in some data structures and algorithms throughout this book.

#### Exponentiation operator

The exponentiation operator may come in handy when working with math algorithms. Let's use the formula to calculate the area of a circle as an example that can be improved/simplified with the exponentiation operator:

```
let area = 3.14 * radius * radius; 
```

The expression `radius * radius` is the same as radius squared. We could also use the `Math.pow` function available in JavaScript to write the same code:

```
area = 3.14 * Math.pow(radius, 2); 
```

In JavaScript, the exponentiation operator is denoted by two asterisks (**). It is used to raise a number (the base) to the power of another number (the exponent). We can calculate the area of a circle using the exponentiation operator as follows:

```
area = 3.14 * (radius ** 2);  
```

## TypeScript fundamentals

TypeScript is an open source **gradually typed** superset of JavaScript created and maintained by Microsoft. Gradual typing is a type system that combines elements of both static typing and dynamic typing within the same programming language.

TypeScript allows us to add types to our JavaScript code, improving code readability, improving early error detection as we can catch type-related errors during development and enhanced tooling as code editors and IDEs offer better code autocompletion and navigation.

Regarding the scope of this book, with TypeScript we can use some Object-Oriented concepts that are not available in JavaScript such as interfaces - this can be useful when working with data structures and sorting algorithms. And of course, we can also leverage the typing functionality, which is especially important for some data structures. In algorithms that modify data structures, like searching or sorting, ensuring consistent data types within the collection is crucial for smooth operation and predictable outcomes. TypeScript excels at automatically enforcing this type consistency, while JavaScript requires additional measures to achieve the same level of assurance.

All these functionalities are available at **compilation time**. Before TypeScript code can run in a browser or Node.js environment, it needs to be compiled into JavaScript. The TypeScript compiler (**tsc**) takes your TypeScript files (_.ts_ extension) and generates corresponding JavaScript files. During compilation, TypeScript checks the code for type-related errors and provides feedback. This helps catch potential issues early in development, leading to more reliable and easier-to-maintain code.

To work with TypeScript in our data structures and algorithms source code, we will leverage **npm** (_Node Package Manager_). Let's set up TypeScript as a development dependency within our "`javascript-datastructures-algorithms`" folder. This involves creating a `package.json` file, which will manage project dependencies. To initiate this process, execute the following command in the project directory using the terminal:

```
npm init
```

You will be prompted some questions, simply press Enter to proceed. And at the end, we will have a `package.json` file with the following content:

```
{
  "name": "javascript-datastructures-algorithms",
  "version": "4.0.0",
  "description": "Learning JavaScript Data Structures and Algorithms",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Loiane Groner"
}
```

Next, we will install TypeScript:

```
npm install --save-dev typescript
```

By running this command, we will save the TypeScript as a development dependency, meaning this will only be used locally during development time. This command will also create a `package-lock.json` file, that can help to ensure that anyone installing the dependencies for the source code from this book will use the exact same packages used for the examples.

> In case you prefer to download the source code, to install the dependencies locally run:

```
npm install
```

Next, we need to create a file with `ts` extension, which is the extension used for TypeScript files, such as `src/01-intro/08-typescript.ts` with the following content:

```
let myName = 'Packt';
myName = 10;
```

The code above is a simple JavaScript code. Now let's compile it using the `tsc` command:

```
npx tsc src/01-intro/08-typescript.ts
```

Where:

- `npx` is the Node Package eXecute, meaning it is a package runner that we will use to execute the TypeScript compiler command.
- `tsc` is the TypeScript compiler command and will compile and transform the source code from src/01-intro/08-typescript.ts into JavaScript.

On the terminal, we will get the following warning:

```
src/01-intro/08-typescript.ts:4:1 - error TS2322: Type 'number' is not assignable to type 'string'.
4 myName = 10;
  ~~~~~~
Found 1 error in src/01-intro/08-typescript.ts:4
```

The warning is due to assigning the numeric value 10 to the variable `myName` we initialized as string.

But if we verify inside the folder `src/01-intro` where we created the file, we will see it created a `08-typescript.js` file with the following content:

```
var myName = 'Packt';
myName = 10;
```

The generated code above is JavaScript code. Even with the error in the terminal (which in fact is a warning, not an error), the TypeScript compiler generated the JavaScript code as it should. This reinforces the fact that TypeScript does all the type and error checking during compile-time, it does not prevent the compiler from generating JavaScript code. This allows developers to leverage all these validations while we are writing the code and get a JavaScript code with less chance of errors or bugs.

### Type inference

While working with TypeScript, you can find code as follows:

```
let age: number = 20;
let existsFlag: boolean = true;
let language: string = 'JavaScript';
```

TypeScript allows us to assign a type to variable. But the code above is verbose. TypeScript has type inference, meaning TypeScript will verify and apply a type to the variable automatically based on the value that was assigned to it. Let's rewrite the preceding code with a cleaner syntax:

```
let age = 20; // number
let existsFlag = true; // boolean
let language = 'JavaScript'; // string
```

With the code above, TypeScript still knows that `age` is a number, `existsFlag` is a boolean and `language` is a string, based on the values that they have been assigned to, so no need to explicitly assign a type to these variables.

So, when do we type a variable? If we declare the variable and do not initialize it with a value, then it recommended to assign a type as demonstrated by the code below:

```
let favoriteLanguage: string;
let langs = ['JavaScript', 'Ruby', 'Python'];
favoriteLanguage = langs[0];
```

If we do not type a variable, then it is automatically typed as `any`, meaning it can receive any value, as it is in JavaScript.

> Although we have object types such as `String`, `Number`, `Boolean`, and so on in JavaScript, when typing a variable in TypeScript, it is not a good practice to use the object types with the first letter capital case. When typing a variable in TypeScript, always prefer `string`, `number`, `boolean` (with the lowercase).
> 
> > `String`, `Number`, and `Boolean` are wrapper objects for the respective primitive types. They provide additional methods and properties, but are generally less efficient and not typically used for basic variable typing. The primitive types (lowercase types) ensure better compatibility with existing JavaScript code and libraries.

### Interfaces

In TypeScript, there are two concepts for interfaces: types and OOP interfaces. Let's review each one.

#### Interface as a type

In TypeScript, interfaces are a powerful way to define the structure or shape of objects. Consider the following code:

```
interface Person {
  name: string;
  age: number;
}
function printName(person: Person) {
  console.log(person.name);
}
```

By declaring a `Person` interface, we are specifying the properties and methods an object might have to adhere to the description of what a `Person` is, meaning we can use the interface `Person` as a type, as we have declared as parameters in the `printName` function.

This allows editors such as VSCode to have autocomplete with intellisense as shown below:

![Figure 1.2 – Visual Studio Code with intellisense for a type interface](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file2.png)

Figure 1.2 – Visual Studio Code with intellisense for a type interface

Now let's try using the `printName` function:

```
const john = { name: 'John', age: 21 };
const mary = { name: 'Mary', age: 21, phone: '123-45678' };
printName(john);
printName(mary);
```

The code above does not have any compilation errors. The variable `john` has a `name` and `age` as expected by the `printName` function. The variable `mary` has `name` and `age`, but also has `phone` information.

So why does this code work? TypeScript has a concept called **Duck Typing**. If it looks like a duck, emits sounds like a duck and behaves like a duck, then it must be a duck! In the example, the variable `mary` behaves like the `Person` interface because it has `name` and `age` properties, so it must be a `Person`. This is a powerful feature of TypeScript.

And after running the `npx tsc src/01-intro/08-typescript.ts` command again, we will get the following output in the `08-typescript.js` file:

```
function printName(person) {
    console.log(person.name);
}
var john = { name: 'John', age: 21 };
var mary = { name: 'Mary', age: 21, phone: '123-45678' };
```

The code above is plain JavaScript. The code completion and type and error checking are available in compile-time only.

> By default, TypeScript compiles to ECMAScript 3. The variable declaration as let and const was only introduced in ECMAScript 6. You can specify the target version by creating a `tsconfig.json` file. Please check the documentation for the steps in case you would like to change this behavior.

#### OOP Interface

The second concept for the TypeScript interface is related to object-oriented programming, this is the same concept as in other OO languages such as Java, C#, Ruby, and so on. An interface is a contract. In this contract, we can define what behavior the classes or interfaces that will implement this contract should have. Consider the ECMAScript standard. ECMAScript is an interface for the JavaScript language. It tells the JavaScript language what functionalities it should have, but each browser might have a different implementation of it.

Let's see an example that will be useful for the data structures and algorithms we will implement throughout this book. Consider the code below:

```
interface Comparable {
  compareTo(b): number;
}
class MyObject implements Comparable {
  age: number;
 
  constructor(age: number) {
    this.age = age;
  }
  compareTo(b): number {
    if (this.age === b.age) {
      return 0;
    }
    return this.age > b.age ? 1 : -1;
  }
}
```

The interface `Comparable` tells the class `MyObject` that it should implement a method called `compareTo` that receives an argument. Inside this method, we can code the required logic. In this case, we are comparing two numbers, but we could use a different logic for comparing two strings or even a more complex object with different attributes. The `compareTo` method returns 0 in case the object is the same, 1, if the current object is bigger than, and -1 in case the current object is smaller than the other object. This interface behavior does not exist in JavaScript, but it is immensely helpful when working with sorting algorithms as an example.

To demonstrate the concept of polymorphism, we can use the code below:

```
function compareTwoObjects(a: Comparable, b: Comparable) {
  console.log(a.compareTo(b));
  console.log(b.compareTo(a));
}
```

In this case, the function `compareTwoObjects` receives two objects that implement the Comparable interface. It can be instances of `MyObject` or any other class that implements this interface.

### Generics

_Generics_ are a powerful feature in TypeScript (and many other strongly typed programming languages) that allow you to write reusable code that can work with various types while maintaining type safety. Think of them as templates or blueprints for functions, classes, or interfaces that can be parameterized with different types.

Let's modify the `Comparable` interface so we can define the type of the object the method `compareTo` should receive as an argument:

```
interface Comparable<T> {
  compareTo(b: T): number;
}
```

By passing the type `T` dynamically to the `Comparable` interface – between the diamond operator `<>`, we can specify the argument type of the `compareTo` function:

```
class MyObject implements Comparable<MyObject> {
  age: number;
 
  constructor(age: number) {
    this.age = age;
  }
  compareTo(b: MyObject): number {
    if (this.age === b.age) {
      return 0;
    }
    return this.age > b.age ? 1 : -1;
  }
}
```

This is useful so we can make sure we are comparing objects of the same type. This is done by ensuring the parameter `b` has a type of T that matches the T inside the diamond operator. By using this functionality, we also get code completion from the editor.

### Enums

**Enums** (short for _Enumerations_) are a way to define a set of named constants. They help organize your code and make it more readable by giving meaningful names to values.

We can use TypeScript `enums` to avoid code smells such as _magic numbers_. A magic number refers to a numerical constant with no explicit explanation of its meaning.

When working with comparing values or objects, which is quite common in sorting algorithms, we often see values such as -1, 1 and 0. But what do these numbers mean?

That is when `enums` come to the rescue to improve code readability. Let's refactor the `compareTo` function from the previous example using an `enum`:

```
enum Compare {
  LESS_THAN = -1,
  BIGGER_THAN = 1,
  EQUALS = 0
}
function compareTo(a: MyObject, b: MyObject): number {
  if (a.age === b.age) {
    return Compare.EQUALS;
  }
  return a.age > b.age ? Compare.BIGGER_THAN : Compare.LESS_THAN;
}
```

By assigning values to each `enum` constant, we can replace the values -1, 1 and 0 with a brief explanation without changing the output of the code.

### Type aliases

TypeScript also has a cool feature called **type aliases**. It allows you to create new names for existing types. It also makes code easier to understand, especially when you are dealing with complex types.

Let's check an example:

```
type UserID = string;
type User = {
  id: UserID;
  name: string;
}
```

In the preceding example, we are creating a type named `UserID` that is an alias for a `string`. And when declaring the second type `User`, we are saying that the `id` is of type `UserID`, making it easier to read the code and understand what the `id` means.

This feature is going to be useful when working with sorting algorithms, as we will be able to create aliases to compare functions so we can write the algorithms in the most generic viable way to work with any data type.

### TypeScript compile-time checking in JavaScript files

Some developers still prefer using plain JavaScript to develop their code instead of TypeScript. But it would be nice if we could use some of the type and error checking features from TypeScript in JavaScript as well, since JavaScript does not provide these features.

The good news is that TypeScript has a special functionality that allows us to have this compile-time error and type checking! To use it, we need to have TypeScript installed globally in our computer using the `npm install -g TypeScript command`.

Let's see how JavaScript is handling the types of a code we used previously in this chapter:

![Figure 1.3 – JavaScript code in VSCode without TypeScript compile-time checking](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file3.png)

Figure 1.3 – JavaScript code in VSCode without TypeScript compile-time checking

In the first line of the JavaScript files, if we want to use the type and error checking, we need to add `// @ts-check` as demonstrated below:

![Figure 1.4 – JavaScript code in VSCode with TypeScript compile-time checking](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file4.png)

Figure 1.4 – JavaScript code in VSCode with TypeScript compile-time checking

The type checking is enabled when we add JSDoc (JavaScript documentation) to our code. To do this, add the following code right before the function declaration:

```
/**
 * Arrow function example, calculates the area of a circle
 * @param {number} radius
 * @returns
 */
```

Then, if we try to pass a string to our circle (or `circleAreaFn`) function, we will get a compilation error:

![Figure 1.5 – Type checking in action for JavaScript](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781836205395/files/media/file5.png)

Figure 1.5 – Type checking in action for JavaScript

> To enable the inferred variable names and types in VSCode, open your settings, and search by _inlay hint_, and enable this feature. It can make a difference (for the better) when coding.

### Other TypeScript functionalities

This was a very quick introduction to TypeScript. The TypeScript documentation is a wonderful place for learning all the other functionalities and diving into the details of the topics we quickly covered in this chapter: [https://www.typescriptlang.org](https://www.typescriptlang.org/).

> The source code bundle of this book also contains a TypeScript version of the JavaScript data structures and algorithms we will develop throughout this book as an extra resource. And whenever TypeScript makes concepts related to data structures and algorithms easier to understand, we will also use it throughout this book.

## Summary

In this chapter, we learned the importance of learning data structures and algorithms, and how it can make us better developers and help us pass technical job interviews in technology. We also reviewed reasons why we chose JavaScript to learn and apply these concepts.

You learned how to set up the development environment to be able to create or execute the examples in this book. We also covered the basics of the JavaScript language that are needed prior to getting started with developing the algorithms and data structures we will cover throughout the book.

We also covered a comprehensive introduction to TypeScript, showcasing its ability to enhance JavaScript with static typing and error checking for more reliable code. We explored essential concepts like interfaces, type inference, and generics, empowering us to write more robust and maintainable data structures and algorithms.

In the next chapter, we'll shift our focus to the critical topic of Big O notation, a fundamental tool for evaluating and understanding the efficiency and performance of our code implementations.