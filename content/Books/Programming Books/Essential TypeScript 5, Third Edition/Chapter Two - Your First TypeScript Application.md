---
id: 01J6VKW0NEK3HABV991SDY4J9R
modified: 2024-09-09T00:46:21-04:00
title: Chapter Two - Your First TypeScript Application
description: Creating your first application with TypeScript
tags:
  - typescript
  - programming
  - web-development
  - books
---
# Chapter Two - Your First TypeScript Application
- ### Creating the Compiler Config File
```typescript
{
    "compilerOptions": {
        "target": "ES2022",
        "outDir": "./dist",
        "rootDir": "./src",
        "module": "CommonJS"
    }
}
```
- I describe the TypeScript compiler in chapter 5, but these settings tell the compiler that I want to use the latest version of JavaScript, that the project’s TypeScript files will be found in the `src` folder, that the output it produces should be placed in the `dist` folder, and that the `CommonJS` setting should be used when loading code from separate files.
- ### Compiling and executing code
	- TypeScript files must be compiled to produce pure JavaScript code that can be executed by browsers or the Node.js runtime installed at the start of this chapter. Use the command line to run the compiler in the `todo` folder using the command in listing 2.8.
	- `Listing 2.8`
```bash
tsc
```
- The compiler reads the configuration settings in the `tsconfig.json` file and locates the TypeScript files in the `src` folder. The compiler creates the `dist` folder and uses it to write out the JavaScript code. If you examine the `dist` folder, you will see that it contains an `index.js` file, where the `js` file extension indicates the file contains JavaScript code. If you examine the contents of the `index.js` file, you will see that it contains the following statements:

```typescript
console.clear();
console.log("Adam's Todo List");
```

- The TypeScript file and the JavaScript file contain the same statements because I have not yet used any TypeScript features. As the application starts to take shape, the contents of the TypeScript file will start to diverge from the JavaScript files that the compiler produces.

**CAUTION Do not make changes to the files in the `dist` folder because they will be overwritten the next time the compiler runs. In TypeScript development, changes are made to files with the `ts` extension, which are compiled into JavaScript files with the `js` extension.**

- To execute the compiled code, use the command prompt to run the command shown in listing 2.9 in the `todo` folder.
```typescript
node dist/index.js
```
- ### Defining the data model
	- The example application will manage a list of to-do items. The user will be able to see the list, add new items, mark items as complete, and filter the items. In this section, I start using TypeScript to define the data model that describes the application’s data and the operations that can be performed on it. To start, add a file called `todoItem.ts` to the `src` folder with the code shown in listing 2.10.
```typescript
export class TodoItem {
    public id: number;
    public task: string;
    public complete: boolean = false;
    
    public constructor(id: number, task: string, 
            complete: boolean = false) {
        this.id = id;
        this.task = task;
        this.complete = complete;
    }
 
    public printDetails() : void {
        console.log(`${this.id}\t${this.task} ${this.complete 
            ? "\t(complete)": ""}`);
    }
}
```
- Classes are templates that describe a data type. I describe classes in more detail in chapter 4, but the code in listing 2.10 will look familiar to any programmer with knowledge of languages such as C# or Java, even if not all of the details are obvious.
- The class in listing 2.10 is named `TodoItem`, and it defines `id`, `task`, and `complete` properties and a `printDetails` method that writes a summary of the to-do item to the console. TypeScript is built on JavaScript, and the code in listing 2.10 is a mix of standard JavaScript features with enhancements that are specific to TypeScript. JavaScript supports classes with constructors, properties, and methods, for example, but features such as access control keywords (such as the `public` keyword) are provided by TypeScript. The headline TypeScript feature is static typing, which allows the type of each property and parameter in the `TodoItem` class to be specified, like this:
```typescript
...
public id: **number**;
...
```
- This is an example of a _type annotation_, and it tells the TypeScript compiler that the `id` property can only be assigned values of the `number` type. As I explain in chapter 3, JavaScript has a fluid approach to types, and the biggest benefit that TypeScript provides is making data types more consistent with other programming languages while still allowing access to the normal JavaScript approach when needed.
- I wrote the class in listing 2.10 to emphasize the similarity between TypeScript and languages such as C# and Java, but this isn’t the way that TypeScript classes are usually defined. listing 2.11 revises the `TodoItem` class to use TypeScript features that allow classes to be defined concisely.
```typescript
export class TodoItem {
    
    **constructor(public id: number,** 
                **public task: string,** 
                **public complete: boolean = false) {**
        **// no statements required**
    }
 
    **printDetails() : void {**
        **console.log(`${this.id}\t${this.task} ${this.complete** 
            **? "\t(complete)": ""}`);**        
    **}**
}
```
- Support for static data types is only part of the broader TypeScript objective of safer and more predictable JavaScript code. The concise syntax used for the constructor in listing 2.11 allows the `TodoItem` class to receive parameters and use them to create instance properties in a single step, avoiding the error-prone process of defining a property and explicitly assigning it the value received by a parameter.
- The change to the `printDetails` method removes the `public` access control keyword, which isn’t needed because TypeScript assumes that all methods and properties are `public` unless another access level is used. (The `public` keyword is still used in the constructor because that’s how the TypeScript compiler recognizes that the concise constructor syntax is being used, as explained in chapter 11.)
### Creating a Todo Collection Class
- The next step is to create a class that will collect together the to-do items so they can be managed more easily. Add a file named `todoCollection.ts` to the `src` folder with the code shown in listing 2.12.
```typescript
import { TodoItem } from "./todoItem";

export class TodoCollection {
    private nextId: number = 1;
    
    constructor(public userName: string, 
                public todoItems: TodoItem[] = []) {
        // no statements required
    }
 
    addTodo(task: string): number {
        while (this.getTodoById(this.nextId)) {
            this.nextId++;
        }        
        this.todoItems.push(new TodoItem(this.nextId, task));
        return this.nextId;
    }
 
    getTodoById(id: number) : TodoItem {
        return this.todoItems.find(item => item.id === id);
    }
 
    markComplete(id: number, complete: boolean) {
        const todoItem = this.getTodoById(id);
        if (todoItem) {
            todoItem.complete = complete;
        }
    }
}
```
### Checking the Basic Data Model Features
- Before going any further, I am going to make sure the initial features of the `TodoCollection` class work as expected. I explain how to perform unit testing for TypeScript projects in chapter 6, but for this chapter, it will be enough to create some `TodoItem` objects and store them in a `TodoCollection` object. listing 2.13 replaces the code in the `index.ts` file, removing the placeholder statements added at the start of the chapter.
```typescript
import { TodoItem } from "./todoItem";
import { TodoCollection } from "./todoCollection";

let todos = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection = new TodoCollection("Adam", todos);

console.clear();
console.log(`${collection.userName}'s Todo List`);

let newId = collection.addTodo("Go for run");
let todoItem = collection.getTodoById(newId);

todoItem.printDetails();
```
- All the statements shown in listing 2.13 use pure JavaScript features. The `import` statements are used to declare dependencies on the `TodoItem` and `TodoCollection` classes, and they are part of the JavaScript modules feature, which allows code to be defined in multiple files (described in chapter 4). Defining an array and using the `new` keyword to instantiate classes are also standard features, along with the calls to the `console` object.
- The TypeScript compiler tries to help developers without getting in the way. During compilation, the compiler looks at the data types that are used and the type information I applied in the `TodoItem` and `TodoCollection` classes and can infer the data types used in listing 2.13. The result is code that doesn’t contain any explicit static type information but that the compiler can check for type safety anyway. To see how this works, listing 2.14 adds a statement to the `index.ts` file.
```typescript
...
collection.addTodo(todoItem);
```
- The new statement calls the `TodoCollection.addTodo` method using a `TodoItem` object as the argument. The compiler looks at the definition of the `addTodo` method in the `todoItem.ts` file and can see that the method expects to receive a different type of data.
```tyepscript
...
**addTodo(task: string): number {**
    while (this.getTodoById(this.nextId)) {
        this.nextId++;
    }
    this.todoItems.push(new TodoItem(this.nextId, task));
    return this.nextId;
}
...
```
- The type information for the `addTodo` method tells the TypeScript compiler that the `task` parameter must be a `string` and that the result will be a `number`. (The `string` and `number` types are built-in JavaScript features and are described in chapter 3.) Run the command shown in listing 2.15 in the `todo` folder to compile the code.
- TypeScript does a good job of figuring out what is going on and identifying problems, allowing you to add as much or as little type information as you like in a project. In this book, I tend to add type information to make the listings easier to follow, since many of the examples in this book are related to how the TypeScript compiler handles data types. Listing 2.16 adds types to the code in the `index.ts` file and disables the statement that causes the compiler error.
```typescript
import { TodoItem } from "./todoItem";
import { TodoCollection } from "./todoCollection";

**let todos: TodoItem[] = [**
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

**let collection: TodoCollection = new TodoCollection("Adam", todos);**

console.clear();
console.log(`${collection.userName}'s Todo List`);

**let newId: number = collection.addTodo("Go for run");**
**let todoItem: TodoItem = collection.getTodoById(newId);**
todoItem.printDetails();
```
- The type information added to the statements in listing 2.16 doesn’t change the way the code works, but it does make the data types being used explicit, which can make the purpose of the code easier to understand and doesn’t require the compiler to infer the data types being used. Run the commands shown in listing 2.17 in the `todo` folder to compile and execute the code.
Run compiler
```bash
tsc node dist/index.js
```
### 2.2.6 Adding features to the collection class

The next step is to add new capabilities to the `TodoCollection` class. First, I am going to change the way that `TodoItem` objects are stored so that a JavaScript `Map` is used, as shown in listing 2.18.

Listing 2.18 Using a map in the todoCollection.ts file in the src folder

import { TodoItem } from "./todoItem";

export class TodoCollection {
    private nextId: number = 1;
    **private itemMap = new Map<number, TodoItem>();**
    
    **constructor(public userName: string, todoItems: TodoItem[] = []) {**
        **todoItems.forEach(item => this.itemMap.set(item.id, item));**
    }
 
    addTodo(task: string): number {
        while (this.getTodoById(this.nextId)) {
            this.nextId++;
        }        
        **this.itemMap.set(this.nextId, new TodoItem(this.nextId, task));**
        return this.nextId;
    }
 
    getTodoById(id: number) : TodoItem {
        **return this.itemMap.get(id);**
    }
 
    markComplete(id: number, complete: boolean) {
        const todoItem = this.getTodoById(id);
        if (todoItem) {
            todoItem.complete = complete;
        }
    }
}

TypeScript supports generic types, which are placeholders for types that are resolved when an object is created. The JavaScript `Map`, for example, is a general-purpose collection that stores key/value pairs. Because JavaScript has such a dynamic type system, a `Map` can be used to store any mix of data types using any mix of keys. To restrict the types that can be used with the `Map` in listing 2.18, I provided generic type arguments that tell the TypeScript compiler which types are allowed for the keys and values.

...
private itemMap = new Map**<number, TodoItem>**();
...

The generic type arguments are enclosed in angle brackets (the `<` and `>` characters), and the `Map` in listing 2.18 is given generic type arguments that tell the compiler that the `Map` will store `TodoItem` objects using `number` values as keys. The compiler will produce an error if a statement attempts to store a different data type in the `Map` or use a key that isn’t a `number` value. Generic types are an important TypeScript feature and are described in detail in chapter 12.

Providing access to to-do items

The `TodoCollection` class defines a `getTodoById` method, but the application will need to display a list of items, optionally filtered to exclude completed tasks. Listing 2.19 adds a method that provides access to the `TodoItem` objects that the `TodoCollection` is managing.

Listing 2.19 Providing access to items in the todoCollection.ts file in the src folder

import { TodoItem } from "./todoItem";

export class TodoCollection {
    private nextId: number = 1;
    private itemMap = new Map<number, TodoItem>();
    
    constructor(public userName: string, todoItems: TodoItem[] = []) {
        todoItems.forEach(item => this.itemMap.set(item.id, item));
    }
 
    addTodo(task: string): number {
        while (this.getTodoById(this.nextId)) {
            this.nextId++;
        }        
        this.itemMap.set(this.nextId, new TodoItem(this.nextId, task));
        return this.nextId;
    }
 
    getTodoById(id: number) : TodoItem {
        return this.itemMap.get(id);
    }
 
    **getTodoItems(includeComplete: boolean): TodoItem[] {**
        **return [...this.itemMap.values()]**
            **.filter(item => includeComplete || !item.complete);**
    **}**
 
    markComplete(id: number, complete: boolean) {
        const todoItem = this.getTodoById(id);
        if (todoItem) {
            todoItem.complete = complete;
        }
    }    
}

The `getTodoItems` method gets the objects from the `Map` using its `values` method and uses them to create an array using the JavaScript spread operator, which is three periods. The objects are processed using the `filter` method to select the objects that are required, using the `includeComplete` parameter to decide which objects are needed.

The TypeScript compiler uses the information it has been given to follow the types through each step. The generic type arguments used to create the `Map` tell the compiler that it contains `TodoItem` objects, so the compiler knows that the `values` method will return `TodoItem` objects and that this will also be the type of the objects in the array. Following this through, the compiler knows that the function passed to the `filter` method will be processing `TodoItem` objects and knows that each object will define a `complete` property. If I try to read a property or method not defined by the `TodoItem` class, the TypeScript compiler will report an error. Similarly, the compiler will report an error if the result of the `return` statement doesn’t match the result type declared by the method.

In listing 2.20, I have updated the code in the `index.ts` file to use the new `TodoCollection` class feature and display a simple list of to-do items to the user.

Listing 2.20 Getting the collection items in the index.ts file in the src folder

import { TodoItem } from "./todoItem";
import { TodoCollection } from "./todoCollection";

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection: TodoCollection = new TodoCollection("Adam", todos);

console.clear();
console.log(`${collection.userName}'s Todo List`);

**//let newId: number = collection.addTodo("Go for run");**
**//let todoItem: TodoItem = collection.getTodoById(newId);**
**//todoItem.printDetails();**
//collection.addTodo(todoItem);
**collection.getTodoItems(true).forEach(item => item.printDetails());**

The new statement calls the `getTodoItems` method defined in listing 2.19 and uses the standard JavaScript `forEach` method to write a description of each `TodoItem` object using the `console` object.

Run the commands shown in listing 2.21 in the `todo` folder to compile and execute the code.

Listing 2.21 Compiling and executing

tsc
node dist/index.js

When the code is executed, the following output will be produced:

Adam's Todo List
1       Buy Flowers
2       Get Shoes
3       Collect Tickets
4       Call Joe        (complete)

Removing completed tasks

As tasks are added and then marked complete, the number of items in the collection will grow and eventually become difficult for the user to manage. Listing 2.22 adds a method that removes the completed items from the collection.

Providing item counts

The final feature I need for the `TodoCollection` class is to provide counts of the total number of `TodoItem` objects, the number that are complete, and the number still outstanding.

I have focused on classes in earlier listings because this is the way that most programmers are used to creating data types. JavaScript objects can also be defined using literal syntax, for which TypeScript can check and enforce static types in the same way as for objects created from classes. When dealing with object literals, the TypeScript compiler focuses on the combination of property names and the types of their values, which is known as an object’s _shape_. A specific combination of names and types is known as a _shape type_. Listing 2.25 adds a method to the `TodoCollection` class that returns an object that describes the items in the collection.

Listing 2.25 Using a shape type in the todoCollection.ts file in the src folder

import { TodoItem } from "./todoItem";

**type ItemCounts = {**
    **total: number,**
    **incomplete: number**
**}**

export class TodoCollection {
    private nextId: number = 1;
    private itemMap = new Map<number, TodoItem>();
    
    constructor(public userName: string, todoItems: TodoItem[] = []) {
        todoItems.forEach(item => this.itemMap.set(item.id, item));
    }
 
    addTodo(task: string): number {
        while (this.getTodoById(this.nextId)) {
            this.nextId++;
        }        
        this.itemMap.set(this.nextId, new TodoItem(this.nextId, task));
        return this.nextId;
    }
 
    getTodoById(id: number) : TodoItem {
        return this.itemMap.get(id);
    }
 
    getTodoItems(includeComplete: boolean): TodoItem[] {
        return [...this.itemMap.values()]
            .filter(item => includeComplete || !item.complete);
    }
 
    markComplete(id: number, complete: boolean) {
        const todoItem = this.getTodoById(id);
        if (todoItem) {
            todoItem.complete = complete;
        }
    }
    
    removeComplete() {
        this.itemMap.forEach(item => {
            if (item.complete) {
                this.itemMap.delete(item.id);
            }
        })
    }
 
    **getItemCounts(): ItemCounts {**
        **return {**
            **total: this.itemMap.size,**
            **incomplete: this.getTodoItems(false).length**
        **};**
    **}**
}

The `type` keyword is used to create a _type alias_, which is a convenient way to assign a name to a shape type. The type alias in listing 2.25 describes objects that have two `number` properties, named `total` and `incomplete`. The type alias is used as the result of the `getItemCounts` method, which uses the JavaScript object literal syntax to create an object whose shape matches the type alias. Listing 2.26 updates the `index.ts` file so that the number of incomplete items is displayed to the user.

Listing 2.26 Displaying item counts in the index.ts file in the src folder

import { TodoItem } from "./todoItem";
import { TodoCollection } from "./todoCollection";

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection: TodoCollection = new TodoCollection("Adam", todos);

console.clear();
**//console.log(`${collection.userName}'s Todo List`);**
**console.log(`${collection.userName}'s Todo List `** 
    **+ `(${ collection.getItemCounts().incomplete } items to do)`);**  
 
**//collection.removeComplete();**
collection.getTodoItems(true).forEach(item => item.printDetails());

Run the commands shown in listing 2.27 in the `todo` folder to compile and execute the code.

Listing 2.27 Compiling and executing

tsc
node dist/index.js

When the code is executed, the following output will be produced:

Adam's Todo List (3 items to do)
1       Buy Flowers
2       Get Shoes
3       Collect Tickets
4       Call Joe        (complete)

 ### 2.3.0 Using a third-party package with TypeScript 
The basic features are in place, but there is room for improvement. One of the joys of writing JavaScript code is the ecosystem of packages that can be incorporated into projects. TypeScript allows any JavaScript package to be used but with the addition of static type support. I am going to use the excellent Inquirer.js package ([https://github.com/SBoudrias/Inquirer.js](https://github.com/SBoudrias/Inquirer.js)) to deal with prompting the user for commands and processing responses.

### 2.3.1 Preparing for the third-party package

One of the drawbacks of writing JavaScript code is the number of competing standards for distributing and using packages. There was no standard package format when JavaScript was first released, and several competing standards arose. The JavaScript language specification now includes a common standard for modules, referred to as _ECMAScript modules_. Most JavaScript runtimes, including Node.js, are implementing support for ECMAScript modules, and most popular JavaScript packages are being updated so they are published in this format.

TypeScript supports ECMAScript modules but requires some changes to the project to enable this feature. Listing 2.28 adds a configuration property to the `package.json` file that denotes that this project requires ECMAScript module support.

Listing 2.28 Adding a configuration property in the package.json file in the todo folder

{
  "name": "todo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"**,**
  **"type": "module"**
}

Listing 2.29 changes the TypeScript compiler configuration so that it looks for the `type` property in the `package.json` file to determine which type of modules are being used.

Listing 2.29 Configuring the compiler in the tsconfig.json file in the todo folder

{
    "compilerOptions": {
        "target": "ES2022",
        "outDir": "./dist",
        "rootDir": "./src",
        **"module": "Node16"**        
    }
}

So far, I have been able to declare dependencies between code files without specifying a file extension, such as with this statement from the `todoCollection.ts` file:

...
import { TodoItem } from "./todoItem";
...

I describe `import` statements in more detail in chapter 4, but what’s important for this chapter is that I specified the file name without an extension. But the way that Node.js has implemented ECMAScript modules requires the file extension to be included, as shown in listing 2.30.

Listing 2.30 Adding a file extension in the todoCollection.ts file in the src folder

**import { TodoItem } from "./todoItem.js";**

type ItemCounts = {
    total: number,
    incomplete: number
}
...

The oddity here is that the `import` statement must specify the JavaScript file that will be generated from the TypeScript file. There are reasons for this, which I explain in later chapters, and the same change is required to the `import` statements in the `index.ts` file, as shown in listing 2.31.

Listing 2.31 Adding file extensions in the index.ts file in the src folder

**import { TodoItem } from "./todoItem.js";**
**import { TodoCollection } from "./todoCollection.js";**

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];
...

### 2.3.2 Installing and using the third-party package

To add Inquirer.js to the project, run the command shown in listing 2.32 in the `todo` folder.

Listing 2.32 Adding a package to the project

npm install inquirer@9.1.4

Packages are added to TypeScript projects just as they are for pure JavaScript projects, using the `npm install` command. To get started with the new package, I added the statements shown in listing 2.33 to the `index.ts` file.

Listing 2.33 Using a new package in the index.ts file in the src folder

import { TodoItem } from "./todoItem.js";
import { TodoCollection } from "./todoCollection.js";
**import inquirer from "inquirer";**

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection: TodoCollection = new TodoCollection("Adam", todos);

function displayTodoList(): void {
    console.log(`${collection.userName}'s Todo List ` 
        + `(${ collection.getItemCounts().incomplete } items to do)`);
    collection.getTodoItems(true).forEach(item => item.printDetails());
}

**enum Commands {**
    **Quit = "Quit"**
**}**

**function promptUser(): void {**
    **console.clear();**
    **displayTodoList();**
    **inquirer.prompt({** 
        **type: "list",** 
        **name: "command",**
        **message: "Choose option",**
        **choices: Object.values(Commands)**
    **}).then(answers => {**
        **if (answers["command"] !== Commands.Quit) {**
            **promptUser();**
        **}**
    **})**
**}**

**promptUser();**

TypeScript doesn’t get in the way of using JavaScript code, and the changes in listing 2.33 make use of the Inquirer.js package to prompt the user and offer a choice of commands. There is only one command available currently, which is `Quit`, but I’ll add more useful features shortly.

TIP I don’t describe the Inquirer.js API in detail in this book because it is not directly related to TypeScript. See [https://github.com/SBoudrias/Inquirer.js](https://github.com/SBoudrias/Inquirer.js) for details if you want to use Inquirer.js in your own projects.

The `inquirer.prompt` method is used to prompt the user for a response and is configured using a JavaScript object. The configuration options I have chosen present the user with a list that can be navigated using the arrow keys, and a selection can be made by pressing Return. When the user makes a selection, the function passed to the `then` method is invoked, and the selection is available through the `answers.command` property.

Listing 2.33 shows how TypeScript code and the JavaScript code from the Inquirer.js package can be used seamlessly together. The `enum` keyword is a TypeScript feature that allows values to be given names, as described in chapter 9, and will allow me to define and refer to commands without needing to duplicate string values through the application. Values from the enum are used alongside the Inquirer.js features, like this:

...
if (**answers**["command"] !== **Commands.Quit**) {
...

Run the commands shown in listing 2.34 in the `todo` folder to compile and execute the code.

Listing 2.37 Running the compiler

tsc

The compiler uses the type information installed in listing 2.35 and reports the following error:

src/index.ts:25:9 - error TS2769: No overload matches this call.
  Overload 1 of 2, '(questions: QuestionCollection<any>, initialAnswers?: 
    Partial<any>): Promise<any> & { ui: Prompt<any>; }', 
      gave the following error.
    Type '"list"' is not assignable to type '"number"'.
  Overload 2 of 2, '(questions: QuestionCollection<any>, initialAnswers?: 
    Partial<any>): Promise<any>', gave the following error.
    Type '"list"' is not assignable to type '"number"'.
25         type: "list",
           ~~~~
Found 1 error in src/index.ts:25

The type declaration allows TypeScript to provide the same set of features throughout the application, even though the Inquirer.js package is pure JavaScript. However, as this example shows, there can be limitations to this feature, and the addition of a property that isn’t supported has produced an error about the value assigned to the `type` property. This happens because it can be difficult to describe the types that pure JavaScript expects, and sometimes the error messages can be more of a general indication that something is wrong.

## 2.4 Adding commands

The example application doesn’t do a great deal at the moment and requires additional commands. In the sections that follow, I add a series of new commands and provide the implementation for each of them.

### 2.4.1 Filtering items

The first command I will add allows the user to toggle the filter to include or exclude completed items, as shown in listing 2.38.

Listing 2.38 Filtering items in the index.ts file in the src folder

import { TodoItem } from "./todoItem.js";
import { TodoCollection } from "./todoCollection.js";
import inquirer from "inquirer";

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection: TodoCollection = new TodoCollection("Adam", todos);
**let showCompleted = true;**

function displayTodoList(): void {
    console.log(`${collection.userName}'s Todo List ` 
        + `(${ collection.getItemCounts().incomplete } items to do)`);
    **//collection.getTodoItems(true).forEach(item => item.printDetails());**
    **collection.getTodoItems(showCompleted)**
        **.forEach(item => item.printDetails());**

}

enum Commands {
    **Toggle = "Show/Hide Completed",**    

    Quit = "Quit"
}

function promptUser(): void {
    console.clear();
    displayTodoList();
    inquirer.prompt({ 
        type: "list", 
        name: "command",
        message: "Choose option",
        choices: Object.values(Commands),
        **//badProperty: true**
    }).then(answers => {
        **switch (answers["command"]) {**
            **case Commands.Toggle:**
                **showCompleted = !showCompleted;**
                **promptUser();**
                **break;**
        **}**
    })
}

promptUser();

The process for adding commands is to define a new value for the `Commands` enum and the statements that respond when the command is selected. In this case, the new value is `Toggle`, and when it is selected, the value of the `showCompleted` variable is changed so that the `displayTodoList` function includes or excludes completed items. Run the commands shown in listing 2.39 in the `todo` folder to compile and execute the code.

Listing 2.39 Compiling and executing

tsc
node dist/index.js

Select the `Show/Hide Completed` option and press Return to toggle the completed tasks in the list, as shown in figure 2.3.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781633437319/files/OEBPS/Images/02-03.png)

Figure 2.3 Toggling completed items

### 2.4.2 Adding tasks

The example application isn’t much use unless the user can create new tasks. Listing 2.40 adds support for creating new `TodoItem` objects.

Listing 2.40 Adding tasks in the index.ts file in the src folder

import { TodoItem } from "./todoItem.js";
import { TodoCollection } from "./todoCollection.js";
import inquirer from "inquirer";

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection: TodoCollection = new TodoCollection("Adam", todos);
let showCompleted = true;

function displayTodoList(): void {
    console.log(`${collection.userName}'s Todo List ` 
        + `(${ collection.getItemCounts().incomplete } items to do)`);
    collection.getTodoItems(showCompleted)
        .forEach(item => item.printDetails());
}

enum Commands {
    **Add = "Add New Task",**
 
    Toggle = "Show/Hide Completed",    
    Quit = "Quit"
}

**function promptAdd(): void {**
    **console.clear();**
    **inquirer.prompt({ type: "input", name: "add", message: "Enter task:"})**
        **.then(answers => {if (answers["add"] !== "") {**
            **collection.addTodo(answers["add"]);**
        **}**
        **promptUser();**
    **})**
**}**

function promptUser(): void {
    console.clear();
    displayTodoList();
    inquirer.prompt({ 
        type: "list", 
        name: "command",
        message: "Choose option",
        choices: Object.values(Commands),
    }).then(answers => {
        switch (answers["command"]) {
            case Commands.Toggle:
                showCompleted = !showCompleted;
                promptUser();
                break;
            **case Commands.Add:** 
                **promptAdd();**
                **break;**
        }
    })
}

promptUser();

The Inquirer.js package can present different types of questions to the user. When the user selects the `Add` command, the `input` question type is used to get the task from the user, which is used as the argument to the `TodoCollection.addTodo` method. Run the commands shown in listing 2.41 in the `todo` folder to compile and execute the code.

Listing 2.41 Compiling and executing

tsc
node dist/index.js

Select the `Add New Task` option, enter some text, and press Return to create a new task, as shown in figure 2.4.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781633437319/files/OEBPS/Images/02-04.png)

Figure 2.4 Adding a new task

### 2.4.3 Marking tasks complete

Completing a task is a two-stage process that requires the user to select the item they want to complete. Listing 2.42 adds the commands and an additional prompt that will allow the user to mark tasks complete and remove the completed items.

Listing 2.42 Completing items in the index.ts file in the src folder

import { TodoItem } from "./todoItem.js";
import { TodoCollection } from "./todoCollection.js";
import inquirer from "inquirer";

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

let collection: TodoCollection = new TodoCollection("Adam", todos);
let showCompleted = true;
 
function displayTodoList(): void {
    console.log(`${collection.userName}'s Todo List ` 
        + `(${ collection.getItemCounts().incomplete } items to do)`);
    collection.getTodoItems(showCompleted)
        .forEach(item => item.printDetails());
}

enum Commands {
    Add = "Add New Task",    
    **Complete = "Complete Task",**
 
    **Toggle = "Show/Hide Completed",**    
    **Purge = "Remove Completed Tasks",**
 
    Quit = "Quit"
}

function promptAdd(): void {
    console.clear();
    inquirer.prompt({ type: "input", name: "add", message: "Enter task:"})
        .then(answers => {if (answers["add"] !== "") {
            collection.addTodo(answers["add"]);
        }
        promptUser();
    })
}

**function promptComplete(): void {**
    **console.clear();**
    **inquirer.prompt({ type: "checkbox", name: "complete",** 
        **message: "Mark Tasks Complete",**
        **choices: collection.getTodoItems(showCompleted).map(item =>** 
            **({name: item.task, value: item.id, checked: item.complete}))**
    **}).then(answers => {**
        **let completedTasks = answers["complete"] as number[];**
        **collection.getTodoItems(true).forEach(item =>** 
            **collection.markComplete(item.id,** 
                **completedTasks.find(id => id === item.id) != undefined));**
        **promptUser();**
    **})**
**}**

function promptUser(): void {
    console.clear();
    displayTodoList();
    inquirer.prompt({ 
        type: "list", 
        name: "command",
        message: "Choose option",
        choices: Object.values(Commands),
    }).then(answers => {
        switch (answers["command"]) {
            case Commands.Toggle:
                showCompleted = !showCompleted;
                promptUser();
                break;
            case Commands.Add: 
                promptAdd();
                break;
            **case Commands.Complete:**
                **if (collection.getItemCounts().incomplete > 0) {**
                    **promptComplete();**
                **} else {**
                    **promptUser();**
                **}**
                **break;**                
            **case Commands.Purge:**
                **collection.removeComplete();**
                **promptUser();**
                **break;**
        }
    })
}

promptUser();

The changes add a new prompt to the application that presents the user with the list of tasks and allows their state to be changed. The `showCompleted` variable is used to determine whether completed items are shown, creating a link between the `Toggle` and `Complete` commands.

The only new TypeScript feature of note is found in this statement:

...
let completedTasks = answers["complete"] **as number[]**;
...

Even with type definitions, there are times when TypeScript isn’t able to correctly assess the types that are being used. In this case, the Inquirer.js package allows any data type to be used in the prompts shown to the user, and the compiler isn’t able to determine that I have used only `number` values, which means that only `number` values can be received as answers. I used a _type assertion_ to address this problem, which allows me to tell the compiler to use the type that I specify, even if it has identified a different data type (or no data type at all). When a type assertion is used, it overrides the compiler, which means that I am responsible for ensuring that the type I assert is correct. Run the commands shown in listing 2.43 in the `todo` folder to compile and execute the code.

Listing 2.43 Compiling and executing

tsc
node dist/index.js

Select the `Complete Task` option, select one or more tasks to change using the spacebar, and then press Return. The state of the tasks you selected will be changed, which will be reflected in the revised list, as shown in figure 2.5.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781633437319/files/OEBPS/Images/02-05.png)

Figure 2.5 Completing items

## 2.5 Persistently storing data

To store the to-do items persistently, I am going to use another open-source package because there is no advantage in creating functionality when there are well-written and well-tested alternatives available. Run the commands shown in listing 2.44 in the `todo` folder to install the Lowdb package and the type definitions that describe its API to TypeScript.

Listing 2.44 Adding a package

npm install lowdb@5.1.0

Lowdb is an excellent database package that stores data in a JSON file and that is used as the data storage component for the `json-server` package, which I use to create HTTP web services in part 3 of this book.

Notice that I didn’t install any type declarations for this package. TypeScript has become so popular that many packages, including Lowdb, ship with type declarations as part of the JavaScript package.

TIP I don’t describe the Lowdb API in detail in this book because it is not directly related to TypeScript. See [https://github.com/typicode/lowdb](https://github.com/typicode/lowdb) for details if you want to use Lowdb in your projects.

I am going to implement persistent storage by deriving from the `TodoCollection` class. In preparation, I changed the access control keyword used by the `TodoCollection` class so that subclasses can access the `Map` that contains the `TodoItem` objects, as shown in listing 2.45.

Listing 2.45 Changing access control in the todoCollection.ts file in the src folder

import { TodoItem } from "./todoItem.js";

type ItemCounts = {
    total: number,
    incomplete: number
}

export class TodoCollection {
    private nextId: number = 1;
    **protected itemMap = new Map<number, TodoItem>();**
    
    constructor(public userName: string, todoItems: TodoItem[] = []) {
        todoItems.forEach(item => this.itemMap.set(item.id, item));
    }
 
    // ...methods omitted for brevity...
}

The `protected` keyword tells the TypeScript compiler that a property can be accessed only by a class or its subclasses. To create the subclass, I added a file called `jsonTodoCollection.ts` to the `src` folder with the code shown in listing 2.46.

Listing 2.46 The contents of the jsonTodoCollection.ts file in the src folder

import { TodoItem } from "./todoItem.js";
import { TodoCollection } from "./todoCollection.js";
import { LowSync } from "lowdb";
import { JSONFileSync  } from "lowdb/node";

type schemaType = {
    tasks: { id: number; task: string; complete: boolean; }[]
};

export class JsonTodoCollection extends TodoCollection {
    private database: LowSync<schemaType>;
 
    constructor(public userName: string, todoItems: TodoItem[] = []) {
        super(userName, []);
        this.database = new LowSync(new JSONFileSync("Todos.json"));
        this.database.read();
 
        if (this.database.data == null) {
            this.database.data = { tasks : todoItems};
            this.database.write();
            todoItems.forEach(item => this.itemMap.set(item.id, item));
        } else {
            this.database.data.tasks.forEach(item => 
                this.itemMap.set(item.id, 
                    new TodoItem(item.id, item.task, item.complete)));
        }
    }
 
    addTodo(task: string): number {
        let result = super.addTodo(task);
        this.storeTasks();
        return result;
    }
 
    markComplete(id: number, complete: boolean): void {
        super.markComplete(id, complete);
        this.storeTasks();
    }
 
    removeComplete(): void {
        super.removeComplete();
        this.storeTasks();
    }
 
    private storeTasks() {
        this.database.data.tasks = [...this.itemMap.values()];
        this.database.write();
    }
}

The type definition for Lowdb uses a schema to describe the structure of the data that will be stored, which is then applied using generic type arguments so that the TypeScript compiler can check the data types being used. For the example application, I need to store only one data type, which I describe using a type alias.

...
type schemaType = {
    tasks: { id: number; task: string; complete: boolean; }[]
};
...

The schema type is used when the Lowdb database is created, and the compiler can check the way that data is used when it is read from the database as in this statement, for example:

...
this.database.data.tasks.forEach(item => this.itemMap.set(item.id, 
    new TodoItem(item.id, item.task, item.complete)));
...

The compiler knows that the `tasks` property presented by the data corresponds to the `tasks` property in the schema type and will return an array of objects with `id`, `task`, and `complete` properties.

Listing 2.47 uses the `JsonTodoCollection` class in the `index.ts` file so that data will be stored persistently by the example application.

Listing 2.47 Using the persistent collection in the index.ts file in the src folder

import { TodoItem } from "./todoItem.js";
import { TodoCollection } from "./todoCollection.js";
import inquirer from "inquirer";
**import { JsonTodoCollection } from "./jsonTodoCollection.js";**

let todos: TodoItem[] = [
    new TodoItem(1, "Buy Flowers"), new TodoItem(2, "Get Shoes"),
    new TodoItem(3, "Collect Tickets"), new TodoItem(4, "Call Joe", true)];

**let collection: TodoCollection = new JsonTodoCollection("Adam", todos);**

let showCompleted = true;
...

Run the commands shown in listing 2.48 in the `todo` folder to compile and execute the code for the final time in this chapter.

Listing 2.48 Compiling and executing

tsc
node dist/index.js

When the application starts, a file called `Todos.json` will be created in the `todo` folder and used to store a JSON representation of the `TodoItem` objects, ensuring that changes are not lost when the application is terminated.

## Summary

In this chapter, I created a simple example application to introduce you to TypeScript development and demonstrate some important TypeScript concepts. You saw that TypeScript provides features that supplement JavaScript, focus on type safety, and help avoid common patterns that trip up developers, especially those coming to JavaScript from languages such as C# or Java.

You saw that TypeScript isn’t used in isolation and that a JavaScript runtime is required to execute the JavaScript code that the TypeScript compiler produces. The advantage of this approach is that projects written with TypeScript have full access to the broad spectrum of JavaScript packages that are available, many of which have type definitions available for easy use.

- TypeScript development can be done with freely available tools.
    
- TypeScript builds on the JavaScript language, with the main feature being static types.
    
- The output from the TypeScript compiler is pure JavaScript, which can be executed by a suitable JavaScript runtime.
    
- TypeScript applications can use standard JavaScript packages, although a basic understanding of JavaScript modules can be required to prepare a TypeScript project before installing a package.
    
- Some JavaScript packages include type information for use with TypeScript.
    
- Separate type declaration packages are available for popular packages that don’t include type declarations.
    

The application I created in this chapter uses some of the most essential TypeScript features, but there are many more available, as you can tell from the size of this book. In the next chapter, I put TypeScript in context and describe the structure and content of this book.