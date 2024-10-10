---
id: 01J8ZVBKNN1ZYQR14KXHYYQXKZ
title: Chapter Three - TypeScript
modified: 2024-10-08T20:12:37-04:00
description: Learning the benefits of and how to use TypeScript
tags:
  - full-stack
  - web-development
  - typescript
  - books
---
TypeScript is a programming language that adds static typing to the dynamically typed JavaScript language. It’s a strict syntactic superset of JavaScript, which means that all existing JavaScript is valid TypeScript. By contrast, TypeScript is not valid JavaScript, because it supplies additional features.

This chapter will introduce you to the pitfalls of working with JavaScript’s dynamic types and explain how TypeScript’s static typing helps catch errors early, increasing the stability of your code. Full-stack developers have embraced TypeScript: it was the runner-up in the _most wanted_ category of a recent Stack Overflow Developer Survey, and 78 percent of participants in a _State of JS_ survey reported using it. According to [_https://builtwith.com_](https://builtwith.com/), TypeScript underlies 7 percent of the top 10,000 sites.

We’ll cover the essential and advanced TypeScript concepts necessary for building full-stack applications. Along the way, you’ll get to know the language’s most common configuration options, its most important types, and how and when to use TypeScript’s static typing features.

### Benefits of TypeScript

TypeScript makes working with JavaScript’s type system less error prone, as its compiler helps us see type errors instantly. Because JavaScript is _dynamically_ typed, you don’t need to specify a variable’s type when declaring it. As soon as the runtime executes the script, it checks these types based on usage. However, this means that errors resulting from invalid types (for example, calling array.map on a variable that holds a number instead of an array) won’t be discovered until runtime, at which point the complete program fails.

In addition to being dynamically typed, JavaScript is also _weakly_ typed, which means it implicitly converts variables to their most plausible values. [Listing 3-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-1) shows an implicit conversion from a number to a string.

```
let string = "1";
let number = 1;
let result;

result = number + number;
console.log("value: ", result, " type of ", typeof(result));

result = number + string;
console.log("value: ", result, " type of ", typeof(result));
```

Listing 3-1: Implicit conversion from a number to a string in JavaScript

We declare three variables, assigning the first a string, the second a numeric value, and the third the result of using the arithmetic plus (+) operator to add the number to itself. We then log the result of this sum operation and its type to the console. If you executed this code, you would see that the value is numeric and that the runtime assigned a type of number to the variable.

Next, we use the same operator again, but instead of adding a numeric value to the number variable, we add a string to it. You should see that the logged value is 11, not 2, as you might have expected. Moreover, the variable’s assigned type has changed to string. This happens because the runtime environment needs to handle an impossible task: adding a number and a string. It solves this issue by implicitly converting the number to a string, then using the plus operator to concatenate the two strings. Without TypeScript, we notice this conversion only when we run the code.

Another common problem caused by untyped variables relates to function and API _contracts_, or the agreements about what the code accepts and returns. When a function takes a parameter, it implicitly expects a parameter of a specific type. But without TypeScript, there is no way to ensure that the parameter type is correct. The same problem exists for the function’s return value. To illustrate this, [Listing 3-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-2) changes the code from [Listing 3-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-1) so that it uses a function to calculate the value of the result variable.

```
let string = "1";
let number = 1;
let result;

const calculate = (a, b) => a + b;

result = calculate(number, number);
console.log("value: ", result, " type of ", typeof(result));

result = calculate(number, string);
console.log("value: ", result, " type of ", typeof(result));
```

Listing 3-2: A function that could return an invalid type due to implicit type conversion

The new calculate function takes two parameters, a and b, and as before, adds the two values. Like in [Listing 3-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-1), as soon as we pass a number and a string as parameters, the function returns a string instead of a number. Our function might expect both parameters to be numbers, but we can’t verify this without manually checking the type by using logic similar to that in [Listing 3-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-3).

```
let string = "1";
let number = 1;
let result;

const calculate = (a, b) => {
    if (Number.isInteger(a) === false || Number.isInteger(b) === false) {
        throw new Error("Invalid type: a parameter is not an integer");
    } else {
        return a + b;
    }
};

result = calculate(number, number);
console.log("value: ", result, " type of ", typeof(result));

result = calculate(number, string);
console.log("value: ", result, " type of ", typeof(result));
```

Listing 3-3: The refactored type-safe function

Here we use the native isInteger function of the Number object to verify that the parameters a and b are integers. The first call of the function, in which we pass it two integers, should calculate the result as expected. The second call, in which we pass the function an integer and a string, looks fine in the editor. However, when we run the code, the runtime environment should throw the error Invalid type: a parameter is not an integer.

There are two main concerns with manually checking the types. First, it adds a lot of noise to our code, as we need to check for all possible types every time we work with function or API contracts, such as when we accept a parameter or return a value. Second, we’re not notified of issues during development. To see the errors in dynamically typed languages, we need to execute the code so that the interpreter can inform us about errors at runtime.

Unlike dynamically typed languages, _statically_ typed languages perform type checks on the code compilation, before runtime. The TypeScript Compiler (TSC) handles this chore; it can run in the background of our code editor or IDE and instantly report all errors based on invalid type usage. Therefore, you can catch errors and see each variable’s assigned types and data structures early.

Even if you don’t set up instant feedback like that, running your code through TSC is necessary before it can be used, which ensures that these kinds of errors are caught earlier than they otherwise would be. The ability to check for these errors is one of the most important benefits of using TypeScript over JavaScript. We will discuss how to benefit from type annotations and when to use them in “Type Annotations” on page 38.

### Setting Up TypeScript

TypeScript’s syntax isn’t valid JavaScript, so a regular JavaScript runtime environment can’t execute it. To run TypeScript in Node.js or a browser, we first need to use TSC to convert it to regular, backward-compatible JavaScript. We then execute the resulting JavaScript.

Despite being called a compiler, TSC doesn’t actually compile TypeScript into JavaScript. Instead, it _transpiles_ it. The difference lies in the level of abstraction. A compiler creates low-level code, while a transpiler is a source-to-source compiler that produces equivalent source code in a language of roughly the same abstraction. For example, you could transpile ES.Next to legacy JavaScript or Python 2 to Python 3. (That said, the terms _transpiling_ and _compiling_ are often used interchangeably.)

In addition to converting TypeScript to JavaScript, TSC checks your code for type errors and verifies the contracts between your functions. The transpiling and type-checking happen independently, and the TSC produces JavaScript regardless of the types you defined. TypeScript errors are merely warnings emitted during the build. They won’t stop the transpiling step as long as the JavaScript itself doesn’t produce an error.

The use of TypeScript won’t affect your code’s performance. The compiler removes types and type operations during the transpilation step, essentially stripping all TypeScript syntax from the actual JavaScript code. Therefore, they can’t affect the runtime or the size of the final code. TypeScript is consequently no slower than JavaScript, although the transpilation can take some time.

#### Installation in Node.js

If you’re using Node.js, you should define TypeScript and all type definitions as development dependencies with the --save-dev flag in your project’s _package.json_ file. There is no need to install TypeScript globally. Just add TypeScript directly to your project with this npm command:

```
$ npm install --save-dev typescript
```

TypeScript files use the extension _.ts_, and because TypeScript is a superset of JavaScript, all valid JavaScript code is automatically valid TypeScript code. Therefore, you can rename your _.js_ files to _.ts_ and instantly use the static type checker with your existing code.

A _tsconfig.json_ file defines TSC configuration options. We’ll cover the most important ones in the next section. For now, run the following command to generate a new file with the default configuration:

```
$ npx tsc -init
```

TSC looks for this file in the current path and all parent directories. The optional -p flag points the TypeScript compiler directly to the file. TSC then reads configuration information from this file and treats its folder as TypeScript’s root directory.

> NOTE

_If you want to follow this chapter’s examples without creating a dedicated project, you can run code in the online playground at_ [https://www.typescriptlang.org/play](https://www.typescriptlang.org/play) _instead of installing TypeScript locally._

#### The tsconfig.json File

Take a look at the basic structure of a _tsconfig.json_ file. The content of the generated file depends on your installed TypeScript version, and there are around 100 configuration properties, but for most projects, only the following few are relevant:

```
{
    "extends": "@tsconfig/recommended/tsconfig.json",
    "compilerOptions": {},
    "include": [],
    "exclude": []
}
```

The extends option is a string that configures the path to another similar configuration file. Usually, this property extends a preset you used as a template with minor, project-specific tweaks. It works similarly to class-based inheritance in object-oriented programming. The preset overrides the base configuration, and the configuration’s key-value pairs overwrite the preset. The example shown here uses the recommended configuration file for TypeScript to override the default settings.

The compilerOptions field configures the transpiling step. We list its options in [Appendix A](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-A.xhtml). The value for include is an array of strings that specifies the patterns or filenames to include for transpiling. The value for exclude is an array of strings that specifies patterns or filenames to exclude. Keep in mind that TSC applies these patterns on the list of files found with the included pattern. Usually, we don’t need to include or exclude files, as our whole project will consist of TypeScript code. Hence, we can leave the arrays empty.

#### Dynamic Feedback with TypeScript

Most modern code editors have support for TypeScript, and they show us the errors generated by TSC directly inside the code. Remember the calculate function we used to explain how TypeScript verifies function contracts? [Figure 3-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#fig3-1) is a screenshot from Visual Studio Code highlighting the type error and hinting at the solution.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure3-1.jpg)

Figure 3-1: Working with TypeScript in Visual Studio Code
.
You can use any code editor or IDE you’d like to write your TypeScript code, though one that shows dynamic feedback like this is recommended.

### Type Annotations

A type annotation is an optional way to explicitly tell the runtime environment which types to expect. You add them following this schema: variable: type. The following example shows a version of the calculate function in which we type both parameters as numbers:

```
const calculate = (a: number, b: number) => a + b;
```

Some developers tend to add types to everything in their code, and by doing so, they add noise that makes the code less readable. This antipattern, called _over-typing_, stems from a false understanding of how type annotations should work. The TypeScript compiler infers types from usage. Therefore, you don’t need to explicitly type everything. Instead, the code editor runs TSC in the background and leverages the results to display the inferred type information and compiler errors as you saw in the “Dynamic Feedback with TypeScript” section.

Rather, type annotations are a way to ensure that code honors the API contracts. There are three scenarios in which you’ll want to verify the contract, and only one of them is especially important. The first scenario, upon a variable’s declaration, is usually not recommended. The second, annotating the return value of a function, is optional, whereas the third scenario, annotating a function’s parameters, is essential. We’ll now take a look at all three of these cases in detail.

#### Declaring a Variable

The most obvious place to type a variable is upon an assignment or declaration. [Listing 3-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-4) demonstrates this by explicitly typing the variable weather as a string and then assigning it a string value.

```
let weather: string = "sunny";
```

Listing 3-4: Over-typing during the variable’s declaration

In most cases, however, this is a form of over-typing, as you could instead leverage the compiler’s type inference. [Listing 3-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-5) shows the alternative pattern of using type inference.

```
let weather = "sunny";
```

Listing 3-5: Inferring the variable’s type based on its value

Because TSC automatically infers the type of this variable, the code editor should show the type information when you hover over the variable. Without the explicit annotation, we have a much cleaner syntax and avoid the noise that the redundant type declaration adds to the code. This improves code readability, which is why this kind of over-typing is usually to be avoided.

#### Declaring a Return Value

Although TypeScript can infer a function’s return type, you’ll usually want to annotate it explicitly. This code pattern ensures that the function’s contract is honored, as the compiler shows implementation errors where the function is defined instead of where it is used.

Another reason to use type annotations in this situation is that, as a programmer, you must explicitly define what a function does. By clarifying the function’s input and output types, you’ll gain a better understanding of what you actually want the function to do. [Listing 3-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-6) shows you how to declare a function’s return type upon declaration.

```
function getWeather(): string {
    const weather = "sunny";
    return weather;
}
```

Listing 3-6: Typing a function’s return value upon declaration

We create a function that returns the weather variable we declared earlier. The weather variable has the inferred string type. Hence, the function returns a string. Our type definition explicitly sets the function’s return type.

#### Declaring a Function’s Parameters

It’s essential to annotate the parameters of a function, because TypeScript doesn’t have enough information to infer function parameters in most cases. By typing these parameters, you’re telling the compiler to check the types when you call the function and pass it arguments. Take a look at [Listing 3-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-7) to see this pattern in action.

```
const weather = "sunny";
function getWeather(weather: string): string {
    return weather;
};
getWeather(weather);
```

Listing 3-7: Typing a function’s parameters

Instead of declaring the weather variable as a constant inside the function, we want the returned value to be dynamic. Therefore, we modify the function to accept a parameter and return it immediately. We then call the function with the weather constant as a parameter.

Good TypeScript code avoids noise and relies on inferring type annotations. It always annotates a function’s parameters and opts for annotated return values but never annotates local variables.

### Built-in Types

Before you can use TypeScript and its annotations, you need to know what types are available to you. One of TypeScript’s main benefits is that it enables you to declare any of JavaScript’s primitive types explicitly. In addition, TypeScript adds its own types, the most important of which are unions, tuples, any, and void. You can also define custom types and interfaces.

#### Primitive JavaScript Types

JavaScript has five primitive types: strings, numbers, Booleans, undefined, and null. Everything else in the language is considered an object. [Listing 3-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-8) shows the syntax for defining variables of these primitive JavaScript types with additional TypeScript type annotations. (Remember that, most of the time, you can just rely on the compiler’s type inference in this situation.)

```
let stringType: string = "bar";
let booleanType: boolean = true;
let integerType: number = 1;
let floatType: number = 1.5;
let nullType: null = null;
let undefinedType: undefined = undefined;
```

Listing 3-8: JavaScript’s primitive types with TypeScript’s type annotations

First we define a string variable and a Boolean with the TypeScript annotations. These are identical to strings and Booleans in JavaScript. Then we define two numbers. Like JavaScript, TypeScript uses a single generic type for numbers, without differentiating between integers and floating points. Finally, we look at TypeScript’s null and undefined types. These behave the same as JavaScript’s primitive types of the same name. _Null_ refers to a value that either is empty or doesn’t exist, and it indicates the intentional absence of a value. In contrast, _undefined_ indicates the unintentional absence of a value. We did not assign a value in [Listing 3-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-5) for the undefined type, because we don’t know it.

#### The union Type

There are a few additional types you should know about, because the more precise your type annotations are, the more helpful you’ll find TSC to be. TypeScript introduced the union type to the JavaScript ecosystem. _Unions_ are variables or parameters that can have more than one data type. [Listing 3-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-9) shows an example of a union type that can be a string or a number.

```
let stringOrNumberUnionType: string | number;
stringOrNumberUnionType = "bar";
stringOrNumberUnionType = 1;
stringOrNumberUnionType = true;
```

Listing 3-9: TypeScript’s union type

We declare a union-type variable that can contain either a string or a number, but nothing else. As soon as we assign a Boolean variable, TSC throws an error, and the IDE shows the message Type 'boolean' is not assignable to type 'string | number'.

While you might find union types useful for annotating function parameters and arrays that can contain different types, you should use them sparingly and avoid them whenever possible. This is because, before working with union-typed items, you need to perform additional manual type checks; otherwise, they could cause errors. For example, if you iterated over an array of strings or numbers and then added all items, you would first need to convert all strings to numbers. Otherwise, JavaScript would implicitly convert the numbers to strings, as shown earlier in this chapter.

#### The array Type

TypeScript provides a generic array type that offers array functions similar to JavaScript’s array. However, take a close look at the syntax for typing the array, shown in [Listing 3-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-10). You’ll notice that the type of the array depends on the type of the array items.

```
let genericArray: [] = [];
genericArray.push(1);

let numberArray: number[] = [];
numberArray.push(1);
```

Listing 3-10: Typed arrays

First we define an array without specifying the type of its items. Unfortunately, what seems to be a definition of a generic array leads to issues down the road. As soon as we try to add a value, TSC throws the error Argument of type 'number' is not assignable to parameter of type 'never', because the array is not typed.

Hence, we need to type the items in the array. Therefore, we create an array, numberArray, in which each item has the type of number. Now we can add numeric values to the array without running into errors.

#### The object Type

TypeScript’s built-in object type is the same as JavaScript’s object. Although you can define the properties’ types for TSC to type-check, the compiler can’t ensure the order of the properties. Nonetheless, it typechecks them, as shown in [Listing 3-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-11).

```
let weatherDetail: {
    weather: string,
    zipcode: string,
    temp: number
} = {weather: "sunny", zipcode: "00000", temp: 1};
weatherDetail.weather = 2;
```

Listing 3-11: Typed objects

Here we define an object with three properties: two that take a string and another that takes a number. Then we try to assign a number to the property weather annotated as a string. Now TSC notifies us with an error explaining that we assigned a value of the wrong type.

Note that, usually, you should avoid typing objects inline, as in this example. Instead, it is a best practice to create a custom type, which is reusable and avoids cluttering our code, enhancing its readability. We discuss how to create and use them in “Custom Types and Interfaces” on page 44.

#### The tuple Type

Another common type that TypeScript adds to JavaScript is the tuple type. Shown in [Listing 3-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-12), _tuples_ are arrays with a specified number of typed items. TypeScript’s tuples are similar to those you might have encountered in programming languages such as Python and C#.

```
let validTuple: [string, number] = ["bar", 1];
let invalidTuple: [string, number] = [1, "bar"];
```

Listing 3-12: TypeScript’s tuple type

We define two tuples. In both, the first array item is a string, and the second is a number. If the type, order, or number of items added to the tuple differs from the tuple’s declaration, TSC throws an error. Here the first assignment is acceptable, whereas the second one throws two errors indicating a mismatch in types.

#### The any Type

TypeScript’s any type is generic, meaning it can take any value, and you should avoid using it. As you can see in [Listing 3-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-13), it accepts all values without throwing an error, which defeats the purpose of static typing.

```
let indifferent: any = true;
indifferent = 1;
indifferent = [];
```

Listing 3-13: TypeScript’s any type

Using any might seem like an easy choice, and it is tempting to rely on it as an escape hatch. Avoid this at all costs. When you pass any as a value to, say, a function, you break the contract you specified in the function declaration, and when you use any to define the contract, there effectively isn’t one.

To view a scenario in which using the any type causes problems, take a look at [Listing 3-14](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-14).

```
const calculate = (a: any, b: any): any => a + b;
console.log(calculate (1,1));
console.log(calculate ("1",1));
```

Listing 3-14: Problems caused by the any type

We reuse the calculate function, which adds two numbers. When we pass two numeric values, we receive the expected output of 2. In a previous example, we typed the parameters as numbers, thus preventing the use of invalid types as arguments.

However, when we use any instead of a number and pass a string to the function, TSC doesn’t throw an error. JavaScript implicitly converts the number to a string and returns an unexpected value of 11. We saw this behavior at the beginning of the chapter, in the untyped version of the function. As you can see, using any is the same as using no types at all.

While convenient, the any type masks your bugs during programming and hides your type designs, rendering type-checking useless. It also prevents your IDE from displaying errors and invalid types.

#### The void Type

TypeScript’s void type is the opposite of any: it indicates no type at all. Its only use case is to annotate the return value of a function that shouldn’t have one, as shown in [Listing 3-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-15).

```
function log(msg: string): void {
    console.log(msg);
}
```

Listing 3-15: TypeScript’s void type

The custom log function we define here passes a parameter to the console. It doesn’t return anything, so we use void as the return type.

To learn more about TypeScript types and other important details of the language, take a look at _The TypeScript Handbook_ at [_https://www.typescriptlang.org/docs/handbook/intro.html_](https://www.typescriptlang.org/docs/handbook/intro.html).

### Custom Types and Interfaces

The previous sections introduced you to enough TypeScript to begin using the language. However, you’ll find it helpful to know a few more advanced concepts. This section shows you how to create custom types and use untyped third-party libraries in your TypeScript code. You’ll also learn when to create a new type and use a custom interface.

While working with TypeScript, remember that a TypeScript file _without_ top-level imports or exports is not a module; therefore, it runs in the _global_ scope. Consequently, all of its declarations are accessible in other modules. By contrast, a TypeScript file _with_ top-level imports or exports is its own module, and all declarations are limited to the _module_ scope, meaning they’re available in the scope of this module only.

#### Defining Custom Types

TypeScript lets you define custom types by using the type keyword. Custom types are a great way to simplify your code. To see how, take a second look at the code shown back in [Listing 3-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-8), when you created a typed object. Now consider [Listing 3-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-16), which optimizes the code with a custom type definition. You should find it much cleaner and easier to read.

```
type WeatherDetailType = {
    weather: string;
    zipcode: string;
    temp?: number;
};

let weatherDetail: WeatherDetailType = {
    weather: "sunny",
    zipcode: "00000",
    temp: 30
};
const getWeatherDetail = (data: WeatherDetailType): WeatherDetailType => data;
```

Listing 3-16: Custom types for typed objects with TypeScript

We create a custom type, WeatherDetailType, with the type keyword. Note that the overall syntax is similar to that used to define an object; we use the equal sign (=) to assign the definition to the custom type.

The custom type has two required properties: weather and zipcode. In addition, it has an optional temp property, as indicated by the question mark (?). Now when we create the getWeatherDetail function, we can annotate the parameter, weatherDetail, as an object with a type of WeatherDetailType. Using this technique, we avoid using inline annotations and can reuse our custom type later, such as to annotate the return type of a function.

#### Defining Interfaces

In addition to types, TypeScript has interfaces. However, the difference between a type and an interface is blurry. You can freely decide which one to use, so long as you follow a convention in your code.

In general, we consider a _type_ definition to answer the question, “Which type is this data?” A possible answer might be a union or a tuple. An _interface_ is a way to describe the shape of some data, such as the properties of an object. It answers the question, “Which properties does this object have?” The most practical difference is that, unlike an interface, we cannot directly modify a type after we’ve declared it. For an in-depth look at the distinction, consult _The TypeScript Handbook_.

As a rule of thumb, use an interface to define a new object or the method of an object. More generally, consider using interfaces over types, as they provide more precise error messages. A classic React use case for interfaces is to define the properties of a specific component. [Listing 3-17](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-17) shows how to use the interface keyword to create a new interface to replace the type in [Listing 3-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-16).

```
interface WeatherProps {
    weather: string;
    zipcode: string;
    temp?: number;
}

const weatherComponent = (props: WeatherProps): string => props.weather;
```

Listing 3-17: Custom interfaces for TypeScript functions

Here we use the interface keyword to define a new interface. Unlike a custom type’s definition, an interface definition does not use the equal sign to assign the interface’s properties to its name. We then use the custom interface to type the properties object props of the weatherComponent, which returns a string.

#### Using Type Declaration Files

To use custom types universally, you can define them in _type declaration files_, which have the _.d.ts_ extension. Unlike regular TypeScript files with the _.ts_ or _.tsx_ extension, type declaration files shouldn’t contain any implementation code. Instead, TSC uses these type definitions to understand custom types and perform type checks. They aren’t transpiled to JavaScript and are never part of the executed script.

Type declaration files prove useful when you find yourself working with external code bases. Often, third-party libraries aren’t written in TypeScript. Therefore, they don’t provide type declaration files for their code bases. Luckily, the DefinitelyTyped repository at [_http://definitelytyped.github.io_](http://definitelytyped.github.io/) provides type declaration files for more than 7,000 libraries. Use these files to add TypeScript support to these libraries.

Type declaration files are collected under the @types scope in npm. This scope holds all the declarations from DefinitelyTyped. Hence, they are easy to find and are grouped next to each other in your _package.json_ file. All type declaration files from the @types scope should be considered development dependencies of your project. Hence, we use the --save-dev flag on the npm install command to add them.

[Listing 3-18](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-18) shows a minimal example of a type declaration file that exports a type and interface for an API.

```
interface WeatherQueryInterface {
    zipcode: string;
}

type WeatherDetailType = {
    weather: string;
    zipcode: string;
    temp?: number;
};
```

Listing 3-18: Defining custom types and interfaces

Save these definitions in a file called _custom.d.ts_ in your root directory. TSC should automatically load these definitions. You can now use the types and interfaces from the file in your TypeScript modules.

Exercise 3: Extend Express.js with TypeScript

Let’s use your new knowledge of TypeScript to rewrite the Express.js server you created in Exercises 1 and 2. In addition to adding type annotations, we’ll add a new route to the server by using custom types.

#### Setting Up

Begin by adding TypeScript to the project following the steps described in “Setting Up TypeScript” on page 36. Next, because Express.js isn’t typed, add type definitions from DefinitelyTyped to your project by running the following:

```
$ npm install --save-dev @types/express
```

Your _package.json_ file should now look like this:

```
{
    "name": "sample-express",
    "version": "1.0.0",
    "description": "sample express server",
    "license": "ISC",
    "type": "module",
    "dependencies": {
        "express": "^4.18.2",
        "node-fetch": "^3.2.6"
    },
    "devDependencies": {
        "@types/express": "^4.17.15",
        "typescript": "^4.9.4"
    }
}
```

Now you can create configuration and type declaration files for the project.

#### Creating the tsconfig.json File

Either create a new _tsconfig.json_ file in the _sample-express_ folder, next to the _index.ts_ file, or open the one you created earlier. Then add or replace its content with the following code:

```
{
    "compilerOptions": {
        "esModuleInterop": true,
        "module": "es6",
        "moduleResolution": "node",
        "target": "es6",
        "noImplicitAny": true
    }
}
```

We configure TypeScript for our simple Express.js server, which requires only a few settings. We use ES.Next modules for our TypeScript code, and because we want to keep them after transpiling the TypeScript to JavaScript, we set module and target to es6. The _express_ package is a CommonJS module. Therefore, we need to use the esModuleInterop option and set the moduleResolution to node. Finally, we use the noImplicitAny option to disallow the implicit use of the any type and require explicit typing. [Appendix A](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-A.xhtml) describes these configuration options in more detail.

#### Defining Custom Types

For our server, we’ll follow a simple rule of thumb: every time we use an object, we should consider adding a custom type or interface to our project. If the object is a function parameter, we’ll create a custom interface. If we use this particular object more than once, we’ll create a custom type.

To define the custom types for this sample project, we create a file _custom.d.ts_ next to the _index.ts_ file in the _sample-express_ folder and add the code from [Listing 3-19](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-19).

```
type responseItemType = {
    id: string;
    name: string;
};

type WeatherDetailType = {
    zipcode: string;
    weather: string;
    temp?: number;
};

interface WeatherQueryInterface {
    zipcode: string;
}
```

Listing 3-19: The custom.d.ts file

We create two custom types and an interface. One defines the response items of the asynchronous API call. The other type and the interface are similar to examples shown earlier in this chapter. They are necessary for the new weather route we will create shortly.

#### Adding Type Annotations to the routes.ts File

Next, we must add type annotations to our server code. Rename the _routes.js_ file in the _sample-express_ folder to _routes.ts_ to enable the TSC for this file. You should instantly see the errors and warnings appear in your editor. Take some time to look at these and then adjust the contents to match the code in [Listing 3-20](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-20). We’ve bolded all type annotations.

```
import fetch from "node-fetch";

const routeHello = (): string => "Hello World!";

const routeAPINames = async (): Promise<string> => {
    const url = "https://www.usemodernfullstack.dev/api/v1/users";
    let data: responseItemType[];
    try {
        const response = await fetch(url);
        data = (await response.json()) as responseItemType[];
    } catch (err) {
        return "Error";
    }
    const names = data
        .map((item) => `id: ${item.id}, name: ${item.name}`)
        .join("<br>");
    return names;
};

const routeWeather = (query: WeatherQueryInterface): WeatherDetailType =>
    queryWeatherData(query);

const queryWeatherData = (query: WeatherQueryInterface): WeatherDetailType => {
    return {
        zipcode: query.zipcode,
        weather: "sunny",
        temp: 35
    };
};

export {routeHello, routeAPINames, routeWeather};
```

Listing 3-20: The typed routes.ts file

Following the principle discussed in “Type Annotations” on page 38, we annotate only a function’s parameters and return types. We also annotate local variables only when their types cannot be inferred, as when converting the fetch response to JSON. Here we need to explicitly type the variable with our custom responseItemType and cast the conversion’s return value as an array of responseItemTypes.

In the rest of the listing, we create the functions for the additional weather route. We use the custom interface for typing both functions’ parameters and the custom type for their return types. In this basic example, the query function returns mostly static data, except the ZIP code, which it takes from the passed parameters. A regular implementation would query a database with the ZIP code and retrieve actual data.

Finally, we add the new route for the weather endpoint to the export statement.

#### Adding Type Annotations to the index.ts File

Rename the file _index.js_ in the _sample-express_ folder to _index.ts_ and adjust the code to match [Listing 3-20](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-20). In addition to the necessary type annotations, create a new endpoint and follow the TypeScript convention to prefix unused parameters with an underscore (_), shown in [Listing 3-21](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#Lis3-21).

```
import {routeHello, routeAPINames, routeWeather} from "./routes.js";
import express, {Request, Response} from "express";

const server = express();
const port = 3000;

server.get("/hello", function (_req: Request, res: Response): void {
    const response = routeHello();
    res.send(response);
});

server.get("/api/names",
    async function (_req: Request, res: Response): Promise<void> {
        let response: string;
        try {
            response = await routeAPINames();
            res.send(response);
        } catch (err) {
            console.log(err);
        }
    }
);

server.get(
    "/api/weather/:zipcode",
    function (req: Request, res: Response): void {
        const response = routeWeather({zipcode: req.params.zipcode});
        res.send(response);
    }
);

server.listen(port, function (): void {
    console.log("Listening on " + port);
});
```

Listing 3-21: The typed index.ts file

First we import the new weather route from the available routes and the Request and Response types from the _express_ package. These are all named exports. Thus, we use curly brackets ({}).

Then, following best practices, we add code annotations and, at the same time, prefix the unused req parameters with an underscore. TSC will follow the convention of functional programming languages by ignoring these parameters. The _api/names_ entry point is marked as an async function, so it needs to return a value wrapped in a promise. Hence, nothing is returned, and we return void as the promise’s value.

In the following lines of code, we create an additional route for a new _/api/weather/:zipcode_ endpoint. The colon (:) creates a parameter on the request’s params object. We retrieve the value for zipcode with req.params .zipcode and pass it down to the routeWeather function. Note that there is no underscore on the request parameter this time. Finally, we use the same function as before to start the Express.js server and listen to port 3000.

#### Transpiling and Running the Code

To transpile the code with the TypeScript compiler to JavaScript, run TSC with npx on the command line:

```
$ npx tsc
```

TSC generates two new files, _index.js_ and _routes.js_, from the TypeScript files. Start the server from your command line with the regular Node.js call:

```
$ node index.js
Listening on 3000
```

Now visit _http://localhost:3000/api/weather/12345_ in your browser. You should see the weather details with the ZIP code 12345, as shown in [Figure 3-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter3.xhtml#fig3-2).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098168810/files/images/Figure3-2.jpg)

Figure 3-2: Browser response from the Node.js web server

Success! You wrote your first TypeScript application.

### Summary

This chapter taught you what you need to know about TypeScript to create a full-stack application. We set up TypeScript and TSC in a new project, then discussed its most important configuration options. Next, you learned to use TypeScript efficiently, leveraging type-annotation inference to avoid over-typing.

We also discussed primitive and advanced built-in types and how to create custom types and interfaces. Finally, you used your new knowledge to add TypeScript to the Express.js server built in previous exercises and refactored the code with type annotations, custom types, and interfaces.

If you want to become a TypeScript expert, I recommend _The TypeScript Handbook_ and the tutorials at [_https://www.typescripttutorial.net_](https://www.typescripttutorial.net/). In the next chapter, you’ll get to know React, a declarative JavaScript library for building user interfaces.