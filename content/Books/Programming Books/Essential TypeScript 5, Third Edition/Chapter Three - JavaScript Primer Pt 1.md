---
id: 01J7AHPFFRKVATVYT0J4AAHFGV
modified: 2024-09-09T00:48:00-04:00
title: Chapter Three - JavaScript Primer Pt 1
description: Learning about using and coercing JavaScript Types
tags:
  - typescript
  - programming
  - web-development
  - javascript
  - books
---
# 3 JavaScript primer, part 1

This chapter covers

- Using the JavaScript types
- Coercing JavaScript types
- Defining and using JavaScript functions and arrays
- Creating and implementing JavaScript objects
- Understanding the `this` keyword

Effective TypeScript development requires an understanding of how JavaScript deals with data types. This can be a disappointment to developers who adopt TypeScript because they found JavaScript confusing, but understanding JavaScript makes understanding TypeScript easier and provides valuable insights into what TypeScript offers and how its features work. In this chapter, I introduce the basic JavaScript type features, continuing with more advanced features in chapter 4.

## 3.1 Preparing for this chapter

To prepare for this chapter, create a folder called `primer` in a convenient location. Open a command prompt, navigate to the `primer` folder, and run the command shown in listing 3.1.

TIP You can download the example project for this chapter—and for all the other chapters in this book—from [https://github.com/manningbooks/essential-typescript-5](https://github.com/manningbooks/essential-typescript-5).

Listing 3.1 Preparing the project folder

npm init --yes

To install a package that will automatically execute the JavaScript file when its contents change, run the command shown in listing 3.2 in the `primer` folder.

Listing 3.2 Installing a package

npm install nodemon@2.0.20

The package, called `nodemon`, will be downloaded and installed. Once the installation is complete, create a file called `index.js` in the `primer` folder with the contents shown in listing 3.3.

Listing 3.3 The contents of the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);

Run the command shown in listing 3.4 to execute the contents of the JavaScript file and monitor it for changes.

Listing 3.4 Starting the JavaScript file monitor

npx nodemon index.js

The `nodemon` package will execute the contents of the `index.js` file and produce the following output:

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node index.js` 
**Hat price: 100**
[nodemon] clean exit - waiting for changes before restart

I have highlighted the part of the output that comes from the `index.js` file. To ensure that changes are detected correctly, alter the contents of the `index.js` file as shown in listing 3.5.

Listing 3.5 Making a change in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
**let bootsPrice = "100";**
**console.log(`Boots price: ${bootsPrice}`);**

When you save the changes, the `nodemon` package should detect that the `index.js` file has been modified and execute the code it contains. The code in listing 3.5 produces the following output, which is shown without the information provided by the `nodemon` package:

Hat price: 100
Boots price: 100

## 3.2 Getting confused by JavaScript

JavaScript has many features that are similar to other programming languages, and developers tend to start with code that looks like the statements in listing 3.5. Even if you are new to JavaScript, the statements in listing 3.5 will be familiar.

The building blocks for JavaScript code are statements, which are executed in the order they are defined. The `let` keyword is used to define variables (as opposed to the `const` keyword, which defines constant values) followed by a name. The value of a variable is set using the assignment operator (the equal sign) followed by a value.

JavaScript provides some built-in objects to perform common tasks, such as writing strings to the command prompt with the `console.log` method. Strings can be defined as literal values, using single or double quotes, or as template strings, using backtick characters and inserting expressions into the template using the dollar sign and braces.

But at some point, unexpected results appear. The cause of the confusion is the way that JavaScript deals with types. Listing 3.6 shows a typical problem.

Listing 3.6 Adding statements in the index.ts file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**if (hatPrice == bootsPrice) {**
    **console.log("Prices are the same");**    
**} else {**
    **console.log("Prices are different");**    
**}**

**let totalPrice = hatPrice + bootsPrice;**
**console.log(`Total Price: ${totalPrice}`);**

The new statements compare the values of the `hatPrice` and `bootsPrice` variables and assign their total to a new variable named `totalPrice`. The `console.log` method is used to write messages to the command prompt and produces the following output when the code is executed:

Hat price: 100
Boots price: 100
**Prices are the same**
Total Price: 100100

Most developers will notice that the value for `hatPrice` has been expressed as a number, while the `bootsPrice` value is a string of characters, enclosed in double quotes. But in most languages, performing operations on different types would be an error. JavaScript is different; comparing a string and a number succeeds, but trying to total the values actually concatenates them. Understanding the results from listing 3.6—and the reasons behind them—reveals the details of how JavaScript approaches data types and why TypeScript can be so helpful.