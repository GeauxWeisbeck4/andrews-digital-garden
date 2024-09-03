---
id: 01J6VKW0NEK3HABV991SDY4J9R
modified: 2024-09-03T07:50:34-04:00
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