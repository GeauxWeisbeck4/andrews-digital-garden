---
id: 01J6DK4ZGKT7F01RV9AMM4PC9C
modified: 2024-08-28T19:16:46-04:00
title: Chapter 1 - TypeScript Fundamentals
description: Getting started with TypeScript
tags:
  - typescript
  - web-development
  - programming
  - books
---
# Chapter 1 TypeScript Fundamentals
# Key differences between TypeScript and JavaScript

TypeScript is a superset of JavaScript, meaning everything you can do in JavaScript, you can also do in TypeScript. It works in all the same places as JavaScript – browsers, Node.js, Bun, and so on. Before the code execution, TypeScript is first transpiled into JavaScript. So, what actually runs is plain JavaScript. TypeScript exists only during development.

This means all your JavaScript knowledge is still valuable in TypeScript. But TypeScript adds a bunch of cool stuff:

- **Type annotations**: TypeScript allows type annotations, where you can explicitly declare variable types. JavaScript does not support this natively.
- **Interfaces**: TypeScript introduces interfaces, a way to define custom types and ensure objects conform to specific structures. JavaScript lacks this feature.
- **Access Modifiers**: TypeScript supports access modifiers such as **private**, **protected**, and **public**, for controlling access to class members. JavaScript does not have this feature built-in.
- **Enums**: TypeScript provides enums, a feature for defining a set of named constants. This is not available in standard JavaScript.
- **Namespaces and modules**: TypeScript offers namespaces for grouping code and avoiding global scope pollution, and it has robust support for ES6 modules. JavaScript primarily relies on ES6 modules.
- **Advanced types**: TypeScript has advanced type features, such as generics, union types, and tuple types, allowing more precise type definitions and manipulation. JavaScript does not have these advanced type features.
- **Tooling and compilation**: TypeScript requires a compilation step to transpile TypeScript code to JavaScript, which can be integrated into build processes. JavaScript can be run directly in browsers and Node.js without this compilation step.

As we develop our chat application, we’ll dive deeper into these TypeScript features and put them to practical use. But before that, let’s discuss the advantages and disadvantages of using TypeScript.

# The advantages of using TypeScript in modern web development

Using TypeScript comes with a few advantages, mostly of which come directly from using types:

- **Type safety**: TypeScript’s static typing helps catch errors at compile time, long before the code is executed. This leads to fewer runtime errors and more robust, reliable code.
- **Improved tooling**: The static type system allows for better tooling support like auto-completion, navigation, and refactoring tools, making the development process more efficient.
- **Easier maintenance**: For large code bases, TypeScript’s type system makes the code easier to understand and maintain. Changes can be made with greater confidence, reducing the likelihood of introducing bugs.
- **Better documentation**: The type annotations serve as a form of documentation, making it easier for new developers to understand what the code is doing.

At the same time, as with every technology, TypeScript is not all that shiny; there are disadvantages as well:

- **Learning curve**: For developers familiar with JavaScript, learning TypeScript introduces an additional layer of complexity due to its static typing and other advanced features.
- **Compilation step**: TypeScript code must be compiled to JavaScript before it can be executed. This adds an extra step to the development process and can complicate build and deployment pipelines. The configuration can become even more complex than plain JavaScript, and with any additional step in the configuration, it is more likely to break.
- **Potentially verbose**: Type annotations and interfaces can make TypeScript code more verbose than JavaScript. This can lead to longer, more complex code, which might be seen as a downside for simple projects.
- **Community and ecosystem adjustments**: While TypeScript is widely adopted, some libraries and third-party tools may still have better support for JavaScript. This means developers might need to invest additional effort to find or create TypeScript type definitions for existing JavaScript libraries.
- **Integration with JavaScript infrastructure**: Since TypeScript is converted to JavaScript for runtime, the type information is lost during execution. This can make using third-party TypeScript libraries a bit inconvenient. When exploring a function from a library, you often end up in the ***.d.ts** files, which define only the type structure of the functions and not the definitions.

Overall, TypeScript isn’t flawless, but the type safety it offers leads to more reliable and error-free code. This benefit is often more crucial for larger projects than the drawbacks TypeScript might present. With the understanding you have now, you’re well prepared to move on to something more practical – the basic syntax of TypeScript.

# Basic syntax of TypeScript

Let’s start with two main aspects of TypeScript – simple types and interfaces.

## Simple types

JavaScript has types, but they are implicit and not strictly enforced. In TypeScript, you’ll find most types are explicit, and you’re required to declare them. Let’s declare one:
```typescript
let messageText: string = "my first chat message"
messageText = 5 // error
```


The difference you can see with the ordinary JavaScript is **: string**. This part defines the time of the variable that we are going to use. Here, we’ve clearly stated that **messageText** is a string. If we try to assign a value of a different type, like a number, we’ll get an error:

```typescript
messageText = 5
```

This line will trigger an error since **5** is not a string.

Another benefit is when the compiler knows the variable is a string, your IDE will suggest actual functions that exist on the string type, which is quite useful. Similarly, we can use other basic types to annotate variables.

There are a few more basic types to know:

- **Number** and **boolean**: The first two, **number** and **boolean**, are also primitives, like the string type.
- **Array<T>**: **Array<T>** is a generic type, a concept we’ll explore in [_Chapter 2_](https://learning.oreilly.com/library/view/full-stack-web-development/9781835885581/B22111_02.xhtml#_idTextAnchor026), but it’s essentially for declaring arrays with a specific type of elements. This can also be written using **[]**.
- Let’s declare two arrays of numbers to demonstrate:
- const numbers: Array<number> = [1, 2]
    const bigNumbers: number[] = [300, 400]
    
    In this code block, we define two variables – **numbers** and **bigNumbers**. They define their types differently, but essentially **Array<number>** and **number[]** mean the same – an array of numbers.
    
- **any**: The last type, **any**, represents any type. Using **any** tells TypeScript to stop checking the type. It’s useful for converting JavaScript to TypeScript, but it basically negates all the benefits TypeScript offers.

I encourage you to experiment with these basic types in your IDE for a better grasp. We’ll also be using them extensively as we build the app, so don’t worry if it doesn’t all click right away. Now, let’s briefly touch on interfaces.

## Interfaces

An **interface** in TypeScript defines the structure of an object, specifying what fields it should have and the types of those fields. Let’s create a simple interface for a chat object and then define a few instances of the **Chat** type. We will complete the code piece with a function called **displayChat**, which accepts a parameter of type **Chat** and logs its details:

interface Chat {
    name: string;
    model: string;
}
const foodChat: Chat = { name: 'food recipes exploration', model: 'gpt-4' };
const typescriptChat: Chat = {name: 'typescript teacher', model: 'gpt-3.5-turbo'};
function displayChat(chat: Chat) {
    console.log(`Chat: ${chat.name}, Model: ${chat.model}`);
}

First, we define an interface called **Chat** that contains two fields **name** and **model** of type **string**. By doing so, we create our own type that we can use in variable type annotations as we did with basic types in the beginning of the _Simple types_ section. When we use it on a variable, it essentially means that the object that we assign to the variable has to follow the structure of the interface.

Then, we define the two variables, **foodChat** and **typescriptChat**, both of which satisfy the **Chat** interface. They hold different data, but both are of type **Chat**. The **displayChat** function accepts any parameter that satisfies the **Chat** interface, meaning that it is an object with **name** and **model** fields of the **string** type.

Interfaces also give an error if you try to access a property that doesn’t exist on the object. They can be extended, have optional properties, and include function definitions. We’ll explore all these features in the upcoming chapters.

# Summary

In this chapter, we’ve covered a brief introduction to TypeScript – how it relates to JavaScript, its advantages and disadvantages, its history, and some basic functionality. With this information in hand, we’re ready to dive deeper into TypeScript. In the next chapter, we are going to cover more advanced aspects of TypeScript: **generics**, **unions**, **classes**, and other useful functionality of its type system.
