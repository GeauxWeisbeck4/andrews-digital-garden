---
id: 01J6CQY19Z145V6BN002CDD2G1
title: Chapter 2 - Systems Programming
modified: 2024-10-01T11:27:33-04:00
description: Learn about Objects and classes
tags:
  - python
  - books
  - software-design
  - programming
---
# Chapter Two - Systems Programming
- The biggest difference between JavaScript and most other programming languages is that many operations in JavaScript are [asynchronous](https://third-bit.com/sdxjs/glossary/#asynchronous). Its designers didn’t want browsers to freeze while waiting for data to arrive or for users to click on things, so operations that might be slow are implemented by describing now what to do later. And since anything that touches the hard drive is slow from a processor’s point of view, [Node](https://nodejs.org/en/) implements [filesystem](https://third-bit.com/sdxjs/glossary/#filesystem) operations the same way.
	- ### How slow is slow?
Table 2.1: Computer operation times at human scale.
|Operation|Actual Time|Would Be…|
|---|---|---|
|1 CPU cycle|0.3 nsec|1 sec|
|Main memory access|120 nsec|6 min|
|Solid-state disk I/O|50-150 μsec|2-6 days|
|Rotational disk I/O|1-10 msec|1-12 months|
|Internet: San Francisco to New York|40 msec|4 years|
|Internet: San Francisco to Australia|183 msec|19 years|
|Physical system reboot|5 min|32,000 years|
- Early JavaScript programs used [callback functions](https://third-bit.com/sdxjs/glossary/#callback) to describe asynchronous operations, but as we’re about to see, callbacks can be hard to understand even in small programs. In 2015, the language’s developers standardized a higher-level tool called promises to make callbacks easier to manage, and more recently they have added new keywords called `async` and `await` to make it easier still. We need to understand all three layers in order to debug things when they go wrong, so this chapter explores callbacks, while [Chapter 3](https://third-bit.com/sdxjs/async-programming/) shows how promises and `async`/`await` work. This chapter also shows how to read and write files and directories with Node’s standard libraries, because we’re going to be doing that a lot.
- ## Section 2.2 - How to list a directory?
	- To start, let’s try listing the contents of a directory the way we would in [Python](https://www.python.org/) or [Java](https://en.wikipedia.org/wiki/Java_(programming_language))
```javascript
// The way we would list a directory in Java or Python

import fs from 'fs'

  

const srcDir = process.argv[2]

const results = fs.readdir(srcDir)

for (const name of results) {

  console.log(name)

}
```
- We use `import _module_ from ‘source’` to load the library `_source_` and assign its contents to `_module_`. After that, we can refer to things in the library using `_module.component_` just as we refer to things in any other object. We can use whatever name we want for the module, which allows us to give short nicknames to libraries with long names; we will take advantage of this in future chapters.
- ### `require` versus `import`
	- In 2015, a new version of JavaScript called ES6 introduced the keyword `import` for importing modules. It improves on the older `require` function in several ways, but Node still uses `require` by default. To tell it to use `import`, we have added `"type": "module"` at the top level of our Node `package.json` file.
- Our little program uses the [`fs`](https://nodejs.org/api/fs.html) library which contains functions to create directories, read or delete files, etc. (Its name is short for “filesystem”.) We tell the program what to list using [command-line arguments](https://third-bit.com/sdxjs/glossary/#command_line_argument), which Node automatically stores in an array called `process.argv`. The name of the program used to run our code is stored `process.argv[0]` (which in this case is `node`), while `process.argv[1]` is the name of our program (in this case `list-dir-wrong.js`). The rest of `process.argv` holds whatever arguments we gave at the command line when we ran the program, so `process.argv[2]` is the first argument after the name of our program ([Figure 2.1](https://third-bit.com/sdxjs/systems-programming/#systems-programming-process-argv)
	- ![Command-line arguments in <code>process.argv</code>](https://third-bit.com/sdxjs/systems-programming/process-argv.svg)
- If we run this program with the name of a directory as its argument, `fs.readdir` returns the names of the things in that directory as an array of strings. The program uses `for (const name of results)` to loop over the contents of that array. We could use `let` instead of `const`, but it’s good practice to declare things as `const` wherever possible so that anyone reading the program knows the variable isn’t actually going to vary—doing this reduces the [cognitive load](https://third-bit.com/sdxjs/glossary/#cognitive_load) on people reading the program. Finally, `console.log` is JavaScript’s equivalent of other languages’ `print` command; its strange name comes from the fact that its original purpose was to create [log messages](https://third-bit.com/sdxjs/glossary/#log_message) in the browser [console](https://third-bit.com/sdxjs/glossary/#console).
- Unfortunately, our program doesn’t work:
```bash
Error: The "cb" argument must be of type function. Received undefined
    at async ModuleLoader.import (https://stackblitzstarterseqdrwr-ibbu.w-credentialless-staticblitz.com/builtins.ddb8d84d.js:154:2688)
    at async loadESM (https://stackblitzstarterseqdrwr-ibbu.w-credentialless-staticblitz.com/builtins.ddb8d84d.js:184:541)
    at async handleMainPromise (https://stackblitzstarterseqdrwr-ibbu.w-credentialless-staticblitz.com/builtins.ddb8d84d.js:165:296) {
  code: 'ERR_INVALID_ARG_TYPE'
}
Node.js v18.20.3
```
- The error message comes from something we didn’t write whose source we would struggle to read. If we look for the name of our file (`list-dir-wrong.js`) we see the error occurred on line 4; everything above that is inside `fs.readdir`, while everything below it is Node loading and running our program.
	- The problem is that `fs.readdir` doesn’t return anything. Instead, its documentation says that it needs a callback function that tells it what to do when data is available, so we need to explore those in order to make our program work.

> [!important] A Theorem
> ### A theorem
> 	1. Every program contains at least one bug.
> 	2. Ever program can be made one line shorter.
> 	3. Therefore, every program can be reduced to a single statement which is wrong.
> - Variously Attributed

- ## Section 2.3 - What is a Callback Function?
	- JavaScript uses a [single-threaded](https://third-bit.com/sdxjs/glossary/#single_threaded) programming model: as the introduction to this lesson said, it splits operations like file I/O into “please do this” and “do this when data is available”. `fs.readdir` is the first part, but we need to write a function that specifies the second part.
	- JavaScript saves a reference to this function and calls with a specific set of parameters when our data is ready ([Figure 2.2](https://third-bit.com/sdxjs/systems-programming/#systems-programming-callbacks)). Those parameters defined a standard [protocol](https://third-bit.com/sdxjs/glossary/#protocol) for connecting to libraries, just like the USB standard allows us to plug hardware devices together.
		- ![[Pasted image 20241001112700.png]]
	- This corrected program gives `fs.readdir` a callback function called `listContents`:
```javascript

```
