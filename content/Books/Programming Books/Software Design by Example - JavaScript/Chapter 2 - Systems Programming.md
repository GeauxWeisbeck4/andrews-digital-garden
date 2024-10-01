---
id: 01J6CQY19Z145V6BN002CDD2G1
title: Chapter 2 - Systems Programming
modified: 2024-09-30T20:32:38-04:00
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
	- To start, let’s try listing the contents of a directory the way we would in [Python](https://www.python.org/) or [Java](https://en.wikipedia.org/wiki/Java_(programming_language)):