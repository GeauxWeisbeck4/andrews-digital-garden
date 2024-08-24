---
id: 01J630QT1HYQB03JMPA7P0718N
modified: 2024-08-24T16:25:20-04:00
title: Chapter 1 - Introduction
description: Chapter 1 of Software design by Example Python
tags:
  - programming
  - python
  - software-design
  - sdbe
---
# Chapter 1 - Introduction
- ## Section 1.1 - The Audience
- _Maya has a master’s degree in genomics. She knows enough Python to analyze data from her experiments, but struggles to write code other people can use. These lessons will show her how to design, build, and test large programs in less time and with less pain._
	- This is who this book is for
- Like Maya, you should be able to:
	- Write Python programs using lists, loops, conditionals, dictionaries, and functions.
	- Puzzle your way through Python programs that use classes and exceptions.
	- Run basic Unix shell commands like `ls` and `mkdir`.
	- Read and write a little bit of HTML.
- Use [Git](https://git-scm.com/) to save and share files. (It’s OK not to know [the more obscure commands](https://git-man-page-generator.lokaltog.net/).)
- These chapters ([Figure 1.1](https://third-bit.com/sdxpy/intro/#intro-syllabus)) are also designed to help another persona:
> _Yim teaches two college courses on software development. They are frustrated that so many books talk about details but not about design and use examples that their students can’t relate to. This book will give them material they can use in class and starting points for course projects._

![[Pasted image 20240824162358.png]]
Our approach to design is based on three big ideas. First, as the number of components in a system grows, the complexity of the system increases rapidly ([Figure 1.2](https://third-bit.com/sdxpy/intro/#intro-complexity)). However, the number of things we can hold in working memory at any time is fixed and fairly small [[Hermans2021](https://third-bit.com/sdxpy/bib/#Hermans2021)]. If we want to build large programs that we can understand, we therefore need to construct them out of pieces that interact in a small number of ways. Figuring out what those pieces and interactions should be is the core of what we call “design”.

![Complexity and size](https://third-bit.com/sdxpy/intro/complexity.svg)

Figure 1.2: How complexity grows with size.

Second, “making sense” depends on who we are. When we use a low-level language, we incur the [cognitive load](https://third-bit.com/sdxpy/glossary/#gl:cognitive_load "The mental effort required to solve a problem.") of assembling micro-steps into something more meaningful. When we use a high-level language, on the other hand, we incur a similar load translating functions of functions into actual operations on actual data.

More experienced programmers are more capable at both ends of the curve, but that’s not the only thing that changes. If a novice’s comprehension curve looks like the lower one in [Figure 1.3](https://third-bit.com/sdxpy/intro/#intro-comprehension), then an expert’s looks like the upper one. Experts don’t just understand more at all levels of abstraction; their _preferred_ level has also shifted so they find �2+�2 easier to read than the Medieval equivalent “the side of the square whose area is the sum of the areas of the two squares whose sides are given by the first part and the second part”. This curve means that for any given task, the code that is quickest for a novice to comprehend will almost certainly be different from the code that an expert can understand most quickly.

![Comprehension curves](https://third-bit.com/sdxpy/intro/comprehension.svg)

Figure 1.3: Novice and expert comprehension curves.

Our third big idea is that programs are just another kind of data. Source code is just text, which we can process like other text files. Likewise, a program in memory is just a data structure that we can inspect and modify like any other. Treating code like data enables us to solve hard problems in elegant ways, but at the cost of increasing the level of abstraction in our programs. Once again, finding the balance is what we mean by “design”.