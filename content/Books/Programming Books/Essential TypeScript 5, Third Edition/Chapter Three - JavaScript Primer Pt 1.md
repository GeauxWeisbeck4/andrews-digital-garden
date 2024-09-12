---
id: 01J7AHPFFRKVATVYT0J4AAHFGV
modified: 2024-09-12T01:32:49-04:00
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

The double equal sign performs a comparison using type coercion so that JavaScript will try to convert the values it is working with to produce a useful result. This is known as the JavaScript _abstract equality comparison_, and when a `number` is compared to a `string`, the `string` value is converted to a `number` value, and then the comparison is performed. This means when the `number` value `100` is compared with the `string` value `100`, the `string` is converted to the `number` value `100`, and this is the reason why the `if` expression evaluates to `true`.

TIP You can read the sequence of steps that JavaScript follows in an abstract equality comparison in the JavaScript specification, [https://262.ecma-international.org/13.0/#sec-islooselyequal](https://262.ecma-international.org/13.0/#sec-islooselyequal). The specification is well-written and surprisingly interesting. But before you spend a day getting lost in the implementation details, you should bear in mind that TypeScript constrains the use of some of the most unusual and exotic features.

The second time coercion is used in listing 3.6 is when the prices are totaled.

...
let totalPrice = **hatPrice + bootsPrice**;
...

When you use the `+` operator on a `number` and a `string`, one of the values is converted. The confusing part is that the conversion isn’t the same as for comparisons. If either of the values is a `string`, the other value is converted to a `string`, and both `string` values are concatenated. This means that when the `number` value `100` is added to the `string` value `100`, the `number` is converted to a `string` and concatenated to produce the `string` result `100100`.

Avoiding Unintentional Type Coercion

Type coercion can be a useful feature, and it has gained a poor reputation only because it is applied unintentionally, which is easy to do when the types being processed are changed with new values. As you will learn in later chapters, TypeScript provides features that help manage unwanted coercion. But JavaScript also provides features to prevent coercion, as shown in listing 3.10.

Listing 3.10 Preventing coercion in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**if (hatPrice === bootsPrice) {**
    console.log("Prices are the same");    
} else {
    console.log("Prices are different");    
}

**let totalPrice = Number(hatPrice) + Number(bootsPrice);**
console.log(`Total Price: ${totalPrice}`);

let myVariable = "Adam";
console.log(`Type: ${typeof myVariable}`);
myVariable = 100;
console.log(`Type: ${typeof myVariable}`);

The double equal sign (`==`) performs a comparison that applies type coercion. The triple equal sign (`===`) applies a strict comparison that will return `true` only if the values have the same type and are equal.

To prevent string concatenation, values can be explicitly converted to numbers before the `+` operator is applied using the built-in `Number` function, with the effect that numeric addition is performed. The code in listing 3.10 produces the following output:

Hat price: 100
Boots price: 100
**Prices are different**
**Total Price: 200**
Type: string
Type: number

Appreciating the Value of Explicitly Applied Type Coercion

Type coercion can be a useful feature when it is explicitly applied. One useful feature is the way that values are coerced into the `boolean` type by the logical OR operator (`||`). Values that are `null` or `undefined` are converted into the `false` value, and this makes an effective tool for providing fallback values, as shown in listing 3.11.

Listing 3.11 Handling null values in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

if (hatPrice === bootsPrice) {
    console.log("Prices are the same");    
} else {
    console.log("Prices are different");    
}

let totalPrice = Number(hatPrice) + Number(bootsPrice);
console.log(`Total Price: ${totalPrice}`);

let myVariable = "Adam";
console.log(`Type: ${typeof myVariable}`);
myVariable = 100;
console.log(`Type: ${typeof myVariable}`);

**let firstCity;**
**let secondCity = firstCity || "London";**
**console.log(`City: ${ secondCity }`);**

The value of the variable named `secondCity` is set with an expression that checks the `firstCity` value: if `firstCity` is converted to the `boolean` value `true`, then the value of `secondCity` will be the value of `firstCity`.

The `undefined` type is used when variables are defined but have not been assigned a value, which is the case for the variable named `firstCity`, and the use of the `||` operator ensures that the fallback value for `secondCity` will be used when `firstCity` is `undefined` or `null`.

Understanding Nullish Coalescing

One problem with the logical OR operator is that it isn’t just `null` or `undefined` that is converted into a `false` value, which can cause unexpected results, as shown in listing 3.12.

Listing 3.12 The effect of type coercion in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**let taxRate; // no tax rate has been defined**
**console.log(`Tax rate: ${taxRate || 10}%`);**
**taxRate = 0; // zero-rated for tax**
**console.log(`Tax rate: ${taxRate || 10}%`);**

In addition to `null` and `undefined`, the logical OR operator will also coerce the number value `0` (zero), the empty string value (`""`), and the special `NaN` number value to `false`. These values, in addition to the `false` value, are collectively known as the JavaScript “falsy” values and cause a lot of confusion. In listing 3.12, the logical OR operator uses the fallback value when the `taxRate` variable is assigned zero and produces the following output:

Hat price: 100
Boots price: 100
Tax rate: 10%
Tax rate: 10%

The code doesn’t differentiate between an unassigned value and the zero value, which can be a problem when zero is a required value. In this example, it is impossible to set a tax rate of zero, even though this is a legitimate rate. To address this problem, JavaScript supports the nullish coalescing operator, `??`, which only coerces `undefined` and `null` values and not the other falsy values, as shown in listing 3.13.

Listing 3.13 Using the nullish operator in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

let taxRate; // no tax rate has been defined
**console.log(`Tax rate: ${taxRate ?? 10}%`);**
taxRate = 0; // zero-rated for tax
**console.log(`Tax rate: ${taxRate ?? 10}%`);**

In the first statement, the fallback value will be used because `taxRate` is `undefined`. In the second statement, the fallback value will not be used because zero is not coerced by the `??` operator, producing the following output:

Hat price: 100
Boots price: 100
Tax rate: 10%
**Tax rate: 0%**
### 3.3.3 Working with functions

The fluid approach that JavaScript takes to types is followed through in other parts of the language, including functions. Listing 3.14 adds a function to the example JavaScript file and removes some of the statements from previous examples for brevity.

Listing 3.14 Defining a function in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**function sumPrices(first, second, third) {**
    **return first + second + third;**
**}**

**let totalPrice = sumPrices(hatPrice, bootsPrice);**
**console.log(`Total Price: ${totalPrice}`);**

A function’s parameter types are determined by the values used to invoke the function. A function may assume that it will receive `number` values, for example, but there is nothing to prevent the function from being invoked with `string`, `boolean`, or `object` arguments. Unexpected results can be produced if the function doesn’t take care to validate its assumptions, either because the JavaScript runtime coerces values or because features specific to a single type are used.

The `sumPrices` function in listing 3.14 uses the `+` operator, intended to sum a set of `number` parameters, but one of the values used to invoke the function is a string, and as explained earlier in the chapter, the `+` operator applied to a `string` value performs concatenation. The code in listing 3.14 produces the following output:

Hat price: 100
Boots price: 100
Total Price: 100100undefined

JavaScript doesn’t enforce a match between the number of parameters defined by a function and the number of arguments used to invoke it. Any parameter for which a value is not provided will be `undefined`. In the listing, no value is provided for the parameter named `third`, and the `undefined` value is converted to the `string` value `undefined` and included in the concatenation output.

Total Price: 100100**undefined**

Working with Function Results

The differences between JavaScript types and those of other languages are magnified by functions. A consequence of the JavaScript type features is that the arguments used to invoke a function can determine the type of the function’s result, as shown in listing 3.15.

Listing 3.15 Invoking a function in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

function sumPrices(first, second, third) {
    return first + second + third;
}

let totalPrice = sumPrices(hatPrice, bootsPrice);
**console.log(`Total: ${totalPrice} ${typeof totalPrice}`);**

**totalPrice = sumPrices(100, 200, 300);**
**console.log(`Total: ${totalPrice} ${typeof totalPrice}`);**

**totalPrice = sumPrices(100, 200);**
**console.log(`Total: ${totalPrice} ${typeof totalPrice}`);**

The value of the `totalPrice` variable is set three times by invoking the `sumPrices` function. After each function call, the `typeof` keyword is used to determine the type of the value returned by the function. The code in listing 3.15 produces the following output:

Hat price: 100
Boots price: 100
**Total: 100100undefined string**
**Total: 600 number**
**Total: NaN number**

The first function call includes a `string` argument, which causes all of the function’s parameters to be converted to `string` values and concatenated, meaning that the function returns the `string` value `100100undefined`.

The second function call uses three `number` values, which are added together and produce the `number` result `600`. The final function call uses number arguments but doesn’t provide a third value, which causes an `undefined` parameter. JavaScript coalesces `undefined` to the special `number` value `NaN` (meaning not a number). The result of addition that includes `NaN` is `NaN`, which means that the type of the result is `number` but the value isn’t useful and is unlikely to be what was intended.

Avoiding Argument Mismatch Problems

Although the results in the previous section can confuse, they are the outcomes described in the JavaScript specification. The problem isn’t that JavaScript is unpredictable but that its approach is different from other popular programming languages.

JavaScript provides features that can be used to avoid these issues. The first is default parameter values that are used if the function is invoked without a corresponding argument, as shown in listing 3.16.

Listing 3.16 Using a default parameter value in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**function sumPrices(first, second, third = 0) {**
    return first + second + third;
}

let totalPrice = sumPrices(hatPrice, bootsPrice);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, 300);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

The name of the `third` parameter is followed by the equal sign and the value that should be used if the function is invoked without a corresponding value. The result is that the statement that invokes the `sumPrices` function with two `number` values will no longer produce the `NaN` result, as shown in the output:

Hat price: 100
Boots price: 100
Total: 1001000 string
Total: 600 number
**Total: 300 number**

A more flexible approach is a rest parameter, which is prefixed with three periods (`...`) and must be the last parameter defined by the function, as shown in listing 3.17.

Listing 3.17 Using a rest parameter in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**function sumPrices(...numbers) {**
    **return numbers.reduce(function(total, val) {**
        **return total + val**
    **}, 0);**
**}**

let totalPrice = sumPrices(hatPrice, bootsPrice);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, 300);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

A rest parameter is an array containing all the arguments for which parameters are not defined. The function in listing 3.17 defines only a rest parameter, which means that its value will be an array containing all of the arguments used to invoke the function. The contents of the array are summed using the built-in array `reduce` method. JavaScript arrays are described in the “Working with Arrays” section, and the `reduce` method is used to invoke a function for each object in the array to produce a single result value. This approach ensures that the number of arguments doesn’t affect the result, but the function invoked by the `reduce` method uses the addition operator, which means that `string` values will still be concatenated. The listing produces the following output:

Hat price: 100
Boots price: 100
Total: 100100 string
Total: 600 number
Total: 300 number

To ensure the function produces a useful sum of its parameter values however they are received, they can be converted to numbers and filtered to remove any that are `NaN`, as shown in listing 3.18.

Listing 3.18 Converting and filtering parameters in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

function sumPrices(...numbers) {
    return numbers.reduce(function(total, val) {
        **return total + (Number.isNaN(Number(val)) ? 0 : Number(val));**
    }, 0);
}

let totalPrice = sumPrices(hatPrice, bootsPrice);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, 300);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

**totalPrice = sumPrices(100, 200, undefined, false, "hello");**
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

The `Number.isNaN` method is used to check whether a `number` value is `NaN`, and the code in listing 3.18 explicitly converts each parameter to a `number` and substitutes zero for those that are `NaN`. Only parameter values that can be treated as numbers are processed, and the `undefined`, `boolean`, and `string` arguments added to the final function call do not affect the result:

Hat price: 100
Boots price: 100
Total: 200 number
Total: 600 number
**Total: 300 number**

Using Arrow Functions

Arrow functions—also known as _fat arrow functions_ or _lambda expressions_—are an alternative way of concisely defining functions and are often used to define functions that are arguments to other functions. Listing 3.19 replaces the standard function used with the array `reduce` method with an arrow function.

Listing 3.19 Using an arrow function in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

function sumPrices(...numbers) {
    **return numbers.reduce((total, val) =>** 
        **total + (Number.isNaN(Number(val)) ? 0 : Number(val)));**
}

let totalPrice = sumPrices(hatPrice, bootsPrice);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, 300);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, undefined, false, "hello");
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

There are three parts to an arrow function: the input parameters, then an equal sign with a greater-than sign (the “arrow”), and finally the result value. The `return` keyword and curly braces are required only if the arrow function needs to execute more than one statement. This listing produces the same output as listing 3.18.

Arrow functions can be used anywhere that a function is required, and their use is a matter of personal preference, except for the issue described in the “Understanding the this Keyword” section. Listing 3.20 redefines the `sumPrices` function in the arrow syntax. This listing produces the same output as listing 3.18.

Listing 3.20 Replacing a function in the index.js file in the primer folder

let hatPrice = 100;
console.log(`Hat price: ${hatPrice}`);
let bootsPrice = "100";
console.log(`Boots price: ${bootsPrice}`);

**let sumPrices = (...numbers) => numbers.reduce((total, val) =>** 
    **total + (Number.isNaN(Number(val)) ? 0 : Number(val)));**

let totalPrice = sumPrices(hatPrice, bootsPrice);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, 300);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

totalPrice = sumPrices(100, 200, undefined, false, "hello");
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

Functions—regardless of which syntax is used—are values, too. They are a special category of the `object` type, described in the “Working with Objects” section, and functions can be assigned to variables passed as arguments to other functions and used like any other value.

In listing 3.20, the arrow syntax is used to define a function that is assigned a variable called `sumPrices`. Functions are special because they can be invoked, but being able to treat functions as values allows complex functionality to be expressed concisely, although it is easy to create code that can be difficult to read. There are more examples of arrow functions and using functions as values throughout the book.

## 3.4 Working with arrays

JavaScript arrays follow the approach taken by most programming languages, except they are dynamically resized and can contain any combination of values and, therefore, any combination of types. Listing 3.21 shows how an array is defined and used.

Listing 3.21 Defining and using an array in the index.js file in the primer folder

**let names = ["Hat", "Boots", "Gloves"];**
**let prices = [];**

**prices.push(100);**
**prices.push("100");**
**prices.push(50.25);**

**console.log(`First Item: ${names[0]}: ${prices[0]}`);**

let sumPrices = (...numbers) => numbers.reduce((total, val) => 
    total + (Number.isNaN(Number(val)) ? 0 : Number(val)));

**let totalPrice = sumPrices(...prices);**
**console.log(`Total: ${totalPrice} ${typeof totalPrice}`);**

The size of an array is not specified when it is created, and capacity will be allocated automatically as items are added or removed. JavaScript arrays are zero-based and are defined using square brackets, optionally with the initial contents separated by commas. The `names` array in the example is created with three `string` values. The `prices` array is created empty, and the `push` method is used to append items to the end of the array. The listing produces the following output:

First Item: Hat: 100
Total: 250.25 number

Elements in the array can be read or set using square brackets or processed using the methods described in table 3.2.

Table 3.2 Useful array methods

|Method|Description|
|---|---|
|`concat(otherArray)`|This method returns a new array that concatenates the array on which it has been called with the array specified as the argument. Multiple arrays can be specified.|
|`join(separator)`|This method joins all the elements in the array to form a string. The argument specifies the character used to delimit the items.|
|`pop()`|This method removes and returns the last item in the array.|
|`shift()`|This method removes and returns the first element in the array.|
|`push(item)`|This method appends the specified item to the end of the array.|
|`unshift(item)`|This method inserts a new item at the start of the array.|
|`reverse()`|This method returns a new array that contains the items in reverse order.|
|`slice(start,end)`|This method returns a section of the array.|
|`sort()`|This method sorts the array. An optional comparison function can be used to perform custom comparisons. Alphabetic sorting is performed if no comparison function is defined.|
|`splice(index, count)`|This method removes `count` items from the array, starting at the specified `index`. The removed items are returned as the result of the method.|
|`every(test)`|This method calls the `test` function for each item in the array and returns `true` if the function returns `true` for all of them and `false` otherwise.|
|`some(test)`|This method returns `true` if calling the `test` function for each item in the array returns `true` at least once.|
|`filter(test)`|This method returns a new array containing the items for which the `test` function returns `true`.|
|`find(test)`|This method returns the first item in the array for which the `test` function returns `true`.|
|`findIndex(test)`|This method returns the index of the first item in the array for which the `test` function returns `true`.|
|`forEach(callback)`|This method invokes the `callback` function for each item in the array, as described in the previous section.|
|`includes(value)`|This method returns `true` if the array contains the specified value.|
|`map(callback)`|This method returns a new array containing the result of invoking the `callback` function for every item in the array.|
|`reduce(callback)`|This method returns the accumulated value produced by invoking the callback function for every item in the array.|

### 3.4.1 Using the spread operator on arrays

The spread operator can be used to expand the contents of an array so that its elements can be used as arguments to a function. The spread operator is three periods (`...`) and is used in listing 3.21 to pass the contents of an array to the `sumPrices` function.

...
let totalPrice = sumPrices(**...prices**);
...

The operator is used before the array name. The spread operator can also be used to expand the contents of an array for easy concatenation, as shown in listing 3.22.

Listing 3.22 Using the spread operator in the index.js file in the primer folder

let names = ["Hat", "Boots", "Gloves"];
let prices = [];

prices.push(100);
prices.push("100");
prices.push(50.25);
console.log(`First Item: ${names[0]}: ${prices[0]}`);

let sumPrices = (...numbers) => numbers.reduce((total, val) => 
    total + (Number.isNaN(Number(val)) ? 0 : Number(val)));

let totalPrice = sumPrices(...prices);
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

**let combinedArray = [...names, ...prices];**
**combinedArray.forEach(element =>** 
    **console.log(`Combined Array Element: ${element}`));**

The spread operator is used to create an array that contains the elements from the `names` and `prices` arrays. The code in listing 3.22 produces the following output:

First Item: Hat: 100
Total: 250.25 number
Combined Array Element: Hat
Combined Array Element: Boots
Combined Array Element: Gloves
Combined Array Element: 100
Combined Array Element: 100
Combined Array Element: 50.25

### 3.4.2 Destructuring arrays

Values from arrays can be unpacked using a destructuring assignment, which assigns selected values to variables, as shown in listing 3.23.

Listing 3.23 Destructuring an array in the index.js file in the primer folder

let names = ["Hat", "Boots", "Gloves"];

**let [one, two] = names;**
**console.log(`One: ${one}, Two: ${two}`);**

The left side of the expression is used to specify the variables to which values will be assigned. In this example, the first value in the `names` array will be assigned to a variable named `one`, and the second value will be assigned to a variable named `two`. The number of variables doesn’t have to match the number of elements in the array: any elements for which there are no variables in the destructuring assignment are ignored, and any variables in the destructuring assignment for which there is no corresponding array element will be `undefined`. The code in listing 3.23 produces the following output:

One: Hat, Two: Boots

Ignoring Elements When Destructuring an Array

You can ignore elements by not specifying a name in the assignment, as shown in listing 3.24.

Listing 3.24 Ignoring elements in the index.js file in the primer folder

let names = ["Hat", "Boots", "Gloves"];

**let [, , three] = names;**
**console.log(`Three: ${three}`);**

No name is specified in the first two positions in the assignment, which means the first two elements in the array are ignored. The third element is assigned to the variable named `three`, and the code produces the following output:

Three: Gloves

Assigning Remaining Elements to an Array

The last variable name in a destructuring assignment can be prefixed with three periods (`...`), known as the _rest expression_ or _rest pattern_, which assigns any remaining elements to an array, as shown in listing 3.25. (The rest expression is often referred to as the _spread operator_ for consistency since both are three periods and behave in similar ways.)

Listing 3.25 Assigning remaining elements in the index.js file in the primer folder

let names = ["Hat", "Boots", "Gloves"];

let [, , three] = names;
console.log(`Three: ${three}`);

**let prices = [100, 120, 50.25];**
**let [, ...highest] = prices.sort((a, b) => a - b);**
**highest.forEach(price => console.log(`High price: ${price}`));**

The `prices` array is sorted, the first element is discarded, and the remaining elements are assigned to an array named `highest`, which is enumerated so that the values can be written to the console, producing the following output:

Three: Gloves
High price: 100
High price: 120

## 3.5 Working with objects

JavaScript objects are collections of properties, each of which has a name and a value. The simplest way to define an object is to use the literal syntax, as shown in listing 3.26.

Listing 3.26 Creating an object in the index.js file in the primer folder

**let hat = {**
    **name: "Hat",**
    **price: 100**
**};**

**let boots = {**
    **name: "Boots",**
    **price: "100"**
**}**

**let sumPrices = (...numbers) => numbers.reduce((total, val) =>** 
    **total + (Number.isNaN(Number(val)) ? 0 : Number(val)));**
 
**let totalPrice = sumPrices(hat.price, boots.price);**
**console.log(`Total: ${totalPrice} ${typeof totalPrice}`);**

The literal syntax uses braces to contain a list of property names and values. Names are separated from their values with colons and from other properties with commas. Objects can be assigned to variables, used as arguments to functions, and stored in arrays. Two objects are defined in listing 3.26 and assigned to variables named `hat` and `boots`. The properties defined by the object can be accessed through the variable name, as shown in this statement, which gets the values of the `price` properties defined by both objects:

...
let totalPrice = sumPrices(**hat.price, boots.price**);
...

The code in listing 3.26 produces the following output:

Total: 200 number

### 3.5.1 Adding, changing, and deleting object properties

Like the rest of JavaScript, objects are dynamic. Properties can be added and removed, and values of any type can be assigned to properties, as shown in listing 3.27.

Listing 3.27 Manipulating an object in the index.js file in the primer folder

let hat = {
    name: "Hat",
    price: 100
};

let boots = {
    name: "Boots",
    price: "100"
}

**let gloves = {**
    **productName: "Gloves",**
    **price: "40"**
**}**

**gloves.name = gloves.productName;**
**delete gloves.productName;**
**gloves.price = 20;**

let sumPrices = (...numbers) => numbers.reduce((total, val) => 
    total + (Number.isNaN(Number(val)) ? 0 : Number(val)));

**let totalPrice = sumPrices(hat.price, boots.price, gloves.price);**
console.log(`Total: ${totalPrice} ${typeof totalPrice}`);

The `gloves` object is created with `productName` and `price` properties. The statements that follow create a `name` property, use the `delete` keyword to remove a property, and assign a `number` value to the `price` property, replacing the previous `string` value. The code in listing 3.27 produces the following output:

Total: 220 number

Guarding Against Undefined Objects and Properties

Care is required when using objects because they may not have the shape (the term used for the combination of properties and values) that you expect or that was originally used when the object was created.

Because the shape of an object can change, setting or getting the value of a property that has not been defined is not an error. If you set a nonexistent property, then it will be added to the object and assigned the specified value. If you read a nonexistent property, then you will receive `undefined`. One useful way to ensure that code always has values to work with is to rely on the type coercion feature and the nullish or logical OR operators, as shown in listing 3.28.

Listing 3.28 Guarding against undefined values in the index.js file in the primer folder

let hat = {
    name: "Hat",
    price: 100
};

let boots = {
    name: "Boots",
    price: "100"
}

let gloves = {
    productName: "Gloves",
    price: "40"
}

gloves.name = gloves.productName;
delete gloves.productName;
gloves.price = 20;

**let propertyCheck = hat.price ?? 0;**
**let objectAndPropertyCheck = (hat ?? {}).price ?? 0;**
**console.log(`Checks: ${propertyCheck}, ${objectAndPropertyCheck}`);**

The code can be difficult to read, but the `??` operator will coerce `undefined` and `null` values to `false` and other values to `true`. The checks can be used to provide a fallback for an individual property, for an object, or a combination of both.

The first check in listing 3.28 assumes the `hat` variable has been assigned a value but checks to make sure `hat.price` is defined and has been assigned a value. The second statement is more cautious—but harder to read—and checks that a value has been assigned to `hat` before also checking the `price` property. The code in listing 3.28 produces the following output:

Checks: 100, 100

The second check in listing 3.28 can be simplified using optional chaining, as shown in listing 3.29.

Listing 3.29 Using optional chaining in the index.js file in the primer folder

let hat = {
    name: "Hat",
    price: 100
};

let boots = {
    name: "Boots",
    price: "100"
}

let gloves = {
    productName: "Gloves",
    price: "40"
}

gloves.name = gloves.productName;
delete gloves.productName;
gloves.price = 20;

let propertyCheck = hat.price ?? 0;
**let objectAndPropertyCheck = hat?.price ?? 0;**
console.log(`Checks: ${propertyCheck}, ${objectAndPropertyCheck}`);

The optional changing operator (the `?` character) will stop evaluating an expression if the value it is applied to is `null` or `undefined`. In the listing, I have applied the operator to `hat`, which means that the expression won’t try to read the value of the price property if `hat` is `undefined` or `null`. The result is that the fallback value will be used if `hat` or `hat.price` is `undefined` or `null`.

### 3.5.2 Using the spread and rest operators on objects

The spread operator can be used to expand the properties and values defined by an object, which makes it easy to create one object based on the properties defined by another, as shown in listing 3.30.

Listing 3.30 Using the object spread operator in the index.js file in the primer folder

let hat = {
    name: "Hat",
    price: 100
};

let boots = {
    name: "Boots",
    price: "100"
}

**let otherHat = { ...hat };**
**console.log(`Spread: ${otherHat.name}, ${otherHat.price}`);**

The spread operator is used to include the properties of the `hat` object as part of the object literal syntax. The use of the spread operator in listing 3.30 has the effect of copying the properties from the `hat` object to the new `otherHat` object. The code in listing 3.30 produces the following output:

Spread: Hat, 100

The spread operator can also be combined with other properties to add, replace, or absorb properties from the source object, as shown in listing 3.31.

Listing 3.31 Adding, replacing, and absorbing in the index.js file in the primer folder

let hat = {
    name: "Hat",
    price: 100
};

let boots = {
    name: "Boots",
    price: "100"
}

**let additionalProperties = { ...hat, discounted: true};**
**console.log(`Additional: ${JSON.stringify(additionalProperties)}`);**
 
**let replacedProperties = { ...hat, price: 10};**
**console.log(`Replaced: ${JSON.stringify(replacedProperties)}`);**
 
**let { price , ...someProperties } = hat;**
**console.log(`Selected: ${JSON.stringify(someProperties)}`);**

The property names and values expanded by the spread operator are treated as though they had been expressed individually in the object literal syntax, which means the shape of an object can be altered by mixing the spread operator with other properties. This statement, for example:

...
let additionalProperties = { ...hat, **discounted: true**};
...

will be expanded so that the properties defined by the `hat` object will be combined with the `discounted` property, equivalent to this statement:

let additionalProperties = { **name: "Hat", price: 100,** discounted: true};

If a property name is used twice in the object literal syntax, then the second value is the one that will be used. This feature can be used to change the value of a property that is obtained through the spread operator and means that this statement:

...
let replacedProperties = { ...hat, **price: 10**};
...

will be expanded so that it is equivalent to this statement:

let replacedProperties = { **name: "Hat", price: 100**, price: 10};

The effect is an object that has the `name` property and value from the `hat` object but with a `price` property whose value is `10`. The rest operator (which is the same three periods as the spread operator) can be used to select properties or to exclude them when used with the object literal syntax. This statement defines variables named `price` and `someProperties`:

...
let { **price , ...someProperties** } = hat;
...

The properties defined by the `hat` object are decomposed. The `hat.price` property is assigned to the new price property, and all the other properties are assigned to the `someProperties` object.

The built-in `JSON.stringify` method creates a string representation of an object using the JSON data format. It is useful only for representing simple objects; it doesn’t usefully deal with functions, for example, but it helps understand how objects are composed, and the code in listing 3.31 produces the following output:

Additional: {"name":"Hat","price":100,"discounted":true}
Replaced: {"name":"Hat","price":10}
Selected: {"name":"Hat"}

### 3.5.3 Defining getters and setters

Getters and setters are functions that are invoked when a property value is read or assigned, as shown in listing 3.32.

Listing 3.32 Using getters and setters in the index.js file in the primer folder

let hat = {
    name: "Hat",    
    **_price: 100,**
    **priceIncTax: 100 * 1.2,**
 
    **set price(newPrice) {**
        **this._price = newPrice;**
        **this.priceIncTax = this._price * 1.2;**
    **},**
 
    **get price() {**
        **return this._price;**
    **}**
};

let boots = {
    name: "Boots",
    price: "100",
 
    **get priceIncTax() {**
        **return Number(this.price) * 1.2;**
    **}**
}

**console.log(`Hat: ${hat.price}, ${hat.priceIncTax}`);**
**hat.price = 120;**
**console.log(`Hat: ${hat.price}, ${hat.priceIncTax}`);**

**console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);**
**boots.price = "120";**
**console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);**

The example introduces a `priceIncTax` property whose value is updated automatically when the `price` property is set. The `hat` object does this by using a getter and setter for the `price` property to update a backing property named `_price`. When a new value is assigned to the `price` property, the setter updates the backing property and the `priceIncTax` property. When the value of the `price` property is read, the getter responds with the value of the `_price` property. (A backing property is required because getters and setters are treated as properties and cannot have the same name as any of the conventional properties defined by the object.)

Understanding JavaScript private properties

JavaScript doesn’t have any built-in support for private properties, except in classes (which I describe in chapter 4). By private, I mean a property that can be accessed only by an object’s methods, getters, and setters. There are techniques to achieve a similar effect outside of classes, but they are complex, and so the most common approach is to use a naming convention to denote properties not intended for public use. This doesn’t prevent access to these properties, but it does at least make it obvious that doing so is undesirable. A widely used naming convention is to prefix the property name with an underscore, as demonstrated with the `_price` property in listing 3.32. This technique isn’t required in TypeScript development, which has its own approach to private properties, as described in chapter 11.

The `boots` object defines the same behavior as the `hat` object but does so by creating a getter that has no corresponding setter, which has the effect of allowing the value to be read but not modified and demonstrates that getters and setters don’t have to be used together. The code in listing 3.32 produces the following output:

Hat: 100, 120
Hat: 120, 144
Boots: 100, 120
Boots: 120, 144

### 3.5.4 Defining methods

JavaScript can be confusing at first, but digging into the details reveals a consistency that isn’t always apparent from casual use. One example is methods, which build on the features described in earlier sections, as shown in listing 3.33.

Listing 3.33 Defining methods in the index.js file in the primer folder

let hat = {
    name: "Hat",    
    _price: 100,
    priceIncTax: 100 * 1.2,
 
    set price(newPrice) {
        this._price = newPrice;
        this.priceIncTax = this._price * 1.2;
    },
 
    get price() {
        return this._price;
    },
 
    **writeDetails: function() {**
        **console.log(`${this.name}: ${this.price}, ${this.priceIncTax}`);**
    **}**
};

let boots = {
    name: "Boots",
    price: "100",

    get priceIncTax() {
        return Number(this.price) * 1.2;
    }
}

**hat.writeDetails();**
**hat.price = 120;**
**hat.writeDetails();**

console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);
boots.price = "120";
console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);

A method is a property whose value is a function, which means that all the features and behaviors that functions provide, such as default and rest parameters, can be used for methods. The method in listing 3.33 is defined using the `function` keyword, but there is a more concise syntax available, as shown in listing 3.34.

Listing 3.34 Using the concise methods syntax in the index.js file in the primer folder

...
**writeDetails() {**
    console.log(`${this.name}: ${this.price}, ${this.priceIncTax}`);
}
...

The `function` keyword and colon that separates a property name from its value are omitted, allowing methods to be defined in a style that many developers find natural. The following output is produced by the listings in this section:

Hat: 100, 120
Hat: 120, 144
Boots: 100, 120
Boots: 120, 144

## 3.6 Understanding the this keyword

The `this` keyword can be confusing to even experienced JavaScript programmers. In other programming languages, `this` is used to refer to the current instance of an object created from a class. In JavaScript, the `this` keyword can often appear to work the same way—right up until the moment a change breaks the application and `undefined` values start to appear.

To demonstrate, I used the fat arrow syntax to redefine the method on the `hat` object, as shown in listing 3.35.

Listing 3.35 Using the fat arrow syntax in the index.js file in the primer folder

let hat = {
    name: "Hat",    
    _price: 100,
    priceIncTax: 100 * 1.2,
 
    set price(newPrice) {
        this._price = newPrice;
        this.priceIncTax = this._price * 1.2;
    },
 
    get price() {
        return this._price;
    },
 
    **writeDetails: () =>** 
        **console.log(`${this.name}: ${this.price}, ${this.priceIncTax}`)**
};

let boots = {
    name: "Boots",
    price: "100",
 
    get priceIncTax() {
        return Number(this.price) * 1.2;
    }
}

hat.writeDetails();
hat.price = 120;
hat.writeDetails();

console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);
boots.price = "120";
console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);

The method uses the same `console.log` statement as listing 3.34, but when the change is saved and the code is executed, the output shows `undefined` values, like this:

**undefined: undefined, undefined**
**undefined: undefined, undefined**
Boots: 100, 120
Boots: 120, 144

Understanding why this happens and being able to fix the problem requires taking a step back and examining what the `this` keyword really does in JavaScript.

### 3.6.1 Understanding the this keyword in stand-alone functions

The `this` keyword can be used in any function, even when that function isn’t used as a method, as shown in listing 3.36.

Listing 3.36 Invoking a function in the index.js file in the primer folder

function writeMessage(message) {
    console.log(`${this.greeting}, ${message}`);
}

greeting = "Hello";

writeMessage("It is sunny today");

The `writeMessage` function reads a property named `greeting` from `this` in one of the expressions in the template string passed to the `console.log` method. The `this` keyword doesn’t appear again in the listing, but when the code is saved and executed, the following output is produced:

Hello, It is sunny today

JavaScript defines a global object, which can be assigned values that are available throughout an application. The global object is used to provide access to the essential features in the execution environment, such as the `document` object in browsers that allows interaction with the Document Object Model API.

Values assigned names without using the `let`, `const`, or `var` keyword are assigned to the global object. The statement that assigns the string value `Hello` creates a variable in the global scope. When the function is executed, `this` is assigned the global object, so reading `this.greeting` returns the `string` value `Hello`, explaining the output produced by the application.

The standard way to invoke a function is to use parentheses that contain arguments, but in JavaScript, this is a convenience syntax that is translated into the statement shown in listing 3.37.

Listing 3.37 Invoking a function in the index.js file in the primer folder

function writeMessage(message) {
    console.log(`${this.greeting}, ${message}`);
}

greeting = "Hello";

writeMessage("It is sunny today");
**writeMessage.call(global, "It is sunny today");**

As explained earlier, functions are objects, which means they define methods, including the `call` method. It is this method that is used to invoke a function behind the scenes. The first argument to the `call` method is the value for `this`, which is set to the global object. This is the reason that `this` can be used in any function and why it returns the global object by default.

The new statement in listing 3.37 uses the `call` method directly and sets the `this` value to the global object, with the same result as the conventional function call before it, which can be seen in the following output produced by the code when executed:

Hello, It is sunny today
Hello, It is sunny today

The name of the global object changes based on the execution environment. In code executed by Node.js, `global` is used, but `window` or `self` may be required in browsers. At the time of writing, there is a proposal to standardize the name `global`, but it has yet to be adopted universally.

Understanding the effect of strict mode

JavaScript supports strict mode, which disables or restricts features that have historically caused poor-quality software or that prevent the runtime from executing code efficiently. When strict mode is enabled, the default value for `this` is `undefined` to prevent accidental use of the global object, and values with global scope must be explicitly defined as properties on the global object. See [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) for details. The TypeScript compiler provides a feature for automatically enabling strict mode in the JavaScript code it generates, as described in chapter 5.

### 3.6.2 Understanding this in methods

When a function is invoked as an object’s method, `this` is set to the object, as shown in listing 3.38.

Listing 3.38 Invoking a function as a method in the index.js file in the primer folder

**let myObject = {**
    **greeting: "Hi, there",**
    
    **writeMessage(message) {**
        **console.log(`${this.greeting}, ${message}`);**        
    **}**
**}**

greeting = "Hello";

**myObject.writeMessage("It is sunny today");**

When the function is invoked via the object, the statement that invokes the function is equivalent to using the `call` method with the object as the first argument, like this:

...
myObject.writeMessage.call(**myObject**, "It is sunny today");
...

Care is required because `this` is set differently if the function is accessed outside of its object, which can happen if the function is assigned to a variable, as shown in listing 3.39.

Listing 3.39 Invoking a function in the index.js file in the primer folder

let myObject = {
    greeting: "Hi, there",
    
    writeMessage(message) {
        console.log(`${this.greeting}, ${message}`);
    }
}

greeting = "Hello";

myObject.writeMessage("It is sunny today");

**let myFunction = myObject.writeMessage;**
**myFunction("It is sunny today");**

Functions can be used like any other value, including assigning them to variables outside of the object in which they were defined, as shown in the listing. If the function is invoked through the variable, then `this` will be set to the global object. This often causes problems when functions are used as arguments to other methods or as callbacks to handle events, and the effect is that the same function will behave differently based on how it is invoked, as shown in the output produced by the code in listing 3.39:

Hi, there, It is sunny today
Hello, It is sunny today

### 3.6.3 Changing the behavior of the this keyword

One way to control the `this` value is to invoke functions using the `call` method, but this is awkward and must be done every time the function is invoked. A more reliable method is to use the function’s `bind` method, which is used to set the value for `this` regardless of how the function is invoked, as shown in listing 3.40.

Listing 3.40 Setting the this value in the index.js file in the primer folder

let myObject = {
    greeting: "Hi, there",
    
    writeMessage(message) {
        console.log(`${this.greeting}, ${message}`);        
    }
}

**myObject.writeMessage = myObject.writeMessage.bind(myObject);**

greeting = "Hello";

myObjectwriteMessage("It is sunny today");

let myFunction = myObject.writeMessage;
myFunction("It is sunny today");

The `bind` method returns a new function that will have a persistent value for `this` when it is invoked. The function returned by the `bind` method is used to replace the original method, ensuring consistency when the `writeMessage` method is invoked. Using `bind` is awkward because the reference to the object isn’t available until after it has been created, which leads to a two-step process of creating the object and then calling `bind` to replace each of the methods for which a consistent `this` value is required. The code in listing 3.40 produces the following output:

Hi, there, It is sunny today
Hi, there, It is sunny today

The value of `this` is always set to `myObject`, even when the `writeMessage` function is invoked as a stand-alone function.

### 3.6.4 Understanding this in arrow functions

To add to the complexity of `this`, arrow functions don’t work in the same way as regular functions. Arrow functions don’t have their own `this` value and inherit the closest value of `this` they can find when they are executed. To demonstrate how this works, listing 3.41 adds an arrow function to the example.

Listing 3.41 Using an arrow function in the index.js file in the primer folder

let myObject = {
    greeting: "Hi, there",
 
    **getWriter() {**
        **return (message) => console.log(`${this.greeting}, ${message}`);**
    **}**
}

greeting = "Hello";

**let writer = myObject.getWriter();**
**writer("It is raining today");**
 
**let standAlone = myObject.getWriter;**
**let standAloneWriter = standAlone();**
**standAloneWriter("It is sunny today");**

In listing 3.41, the `getWriter` function is a regular function that returns an arrow function as its result. When the arrow function returned by `getWriter` is invoked, it works its way up its scope until it locates a value for `this`. As a consequence, the way that the `getWriter` function is invoked determines the value of `this` for the arrow function. Here are the first two statements that invoke the functions:

...
let writer = **myObject**.getWriter();
writer("It is raining today");
...

These two statements can be combined as follows:

...
**myObject**.getWriter()("It is raining today");
...

The combined statement is a little harder to read, but it helps emphasize that the value of `this` is based on how a function is invoked. The `getWriter` method is invoked through `myObject` and means that the value of `this` will be set to `myObject`. When the arrow function is invoked, it finds a value of `this` from the `getWriter` function. The result is that when the `getWriter` method is invoked through `myObject`, the value of `this` in the arrow function will be `myObject`, and the `this.greeting` expression in the template string will be `Hi,` `there`.

The statements in the second set treat `getWriter` as a stand-alone function, so `this` will be set to the global object. When the arrow function is invoked, the `this.greeting` expression will be `Hello`. The code in listing 3.41 produces the following output, confirming the `this` value in each case:

Hi, there, It is raining today
Hello, It is sunny today

### 3.6.5 Returning to the original problem

I started this section by redefining a function in the arrow syntax and showing that it behaved differently, producing `undefined` in its output. Here is the object and its function:

...
let hat = {
    name: "Hat",    
    _price: 100,
    priceIncTax: 100 * 1.2,
 
    set price(newPrice) {
        this._price = newPrice;
        this.priceIncTax = this._price * 1.2;
    },
 
    get price() {
        return this._price;
    },
 
    **writeDetails: () =>** 
        **console.log(`${this.name}: ${this.price}, ${this.priceIncTax}`)**
};
...

The behavior changed because arrow functions don’t have their own `this` value, and the arrow function isn’t enclosed by a regular function that can provide one. To resolve the issue and be sure that the results will be consistent, I must return to a regular function and use the `bind` method to fix the `this` value, as shown in listing 3.42.

Listing 3.42 Resolving the function problem in the index.js file in the primer folder

let hat = {
    name: "Hat",    
    _price: 100,
    priceIncTax: 100 * 1.2,
 
    set price(newPrice) {
        this._price = newPrice;
        this.priceIncTax = this._price * 1.2;
    },
 
    get price() {
        return this._price;
    },
 
    **writeDetails() {**
         **console.log(`${this.name}: ${this.price}, ${this.priceIncTax}`);**
    **}**
};

let boots = {
    name: "Boots",
    price: "100",
 
    get priceIncTax() {
        return Number(this.price) * 1.2;
    }
}

**hat.writeDetails = hat.writeDetails.bind(hat);**
hat.writeDetails();
hat.price = 120;
hat.writeDetails();

console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);
boots.price = "120";
console.log(`Boots: ${boots.price}, ${boots.priceIncTax}`);

With these changes, the value of `this` for the `writeDetails` method will be its enclosing object, regardless of how it is invoked, producing the following output:

Hat: 100, 120
Hat: 120, 144
Boots: 100, 120
Boots: 120, 144

## Summary

In this chapter, I introduced the basic features of the JavaScript type system. These are features that often confuse because they work differently from those in other programming languages. Understanding these features make working with TypeScript easier because they provide insight into the problems that TypeScript solves. JavaScript has a set of built-in data types that are used to represent all values.

- JavaScript will attempt to convert data types when they are combined with an operator.
    
- JavaScript functions can be defined with a literal syntax that declares parameters and a function body or using the fat arrow/lambda function syntax.
    
- JavaScript functions can accept a variable number of arguments, which can be captured using a rest parameter.
    
- JavaScript functions do not formally declare results and can return any result type.
    
- JavaScript arrays are variable-length and can accept values of any type.
    
- JavaScript objects are a collection of properties and values and can be defined using a literal syntax.
    
- JavaScript objects can be altered to add, change, or remove properties.
    
- JavaScript objects can be defined with methods, which are functions assigned to a property.
    
- The `this` keyword refers to different objects depending on how functions are invoked.
    

In the next chapter, I describe more of the JavaScript type features that are useful for understanding TypeScript.