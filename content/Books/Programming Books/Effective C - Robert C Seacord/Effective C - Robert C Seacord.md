---
title: Effective C
subtitle: An Introduction to Professional C Programming
author: Robert C. Seacord
authors: Robert C. Seacord
category: Computers
categories: Computers
publisher: No Starch Press
publishDate: 2020-08-11
totalPage: 273
coverUrl: http://books.google.com/books/content?id=X5DZDwAAQBAJ&printsec=frontcover&img=1&zoom=1&edge=curl&source=gbs_api
coverSmallUrl: http://books.google.com/books/content?id=X5DZDwAAQBAJ&printsec=frontcover&img=1&zoom=5&edge=curl&source=gbs_api
description: "A detailed introduction to the C programming language for experienced programmers. The world runs on code written in the C programming language, yet most schools begin the curriculum with Python or Java. Effective C bridges this gap and brings C into the modern era--covering the modern C17 Standard as well as potential C2x features. With the aid of this instant classic, you'll soon be writing professional, portable, and secure C programs to power robust systems and solve real-world problems. Robert C. Seacord introduces C and the C Standard Library while addressing best practices, common errors, and open debates in the C community. Developed together with other C Standards committee experts, Effective C will teach you how to debug, test, and analyze C programs. You'll benefit from Seacord's concise explanations of C language constructs and behaviors, and from his 40 years of coding experience. You'll learn: How to identify and handle undefined behavior in a C program The range and representations of integers and floating-point values How dynamic memory allocation works and how to use nonstandard functions How to use character encodings and types How to perform I/O with terminals and filesystems using C Standard streams and POSIX file descriptors How to understand the C compiler's translation phases and the role of the preprocessor How to test, debug, and analyze C programs Effective C will teach you how to write professional, secure, and portable C code that will stand the test of time and help strengthen the foundation of the computing world."
link: https://play.google.com/store/books/details?id=X5DZDwAAQBAJ
previewLink: http://books.google.com/books?id=X5DZDwAAQBAJ&printsec=frontcover&dq=effective+c&hl=&as_pt=BOOKS&cd=1&source=gbs_api
isbn13: 9781718501058
isbn10: 1718501056
id: 01JCH1HATT9ZWV7SX3SB6ZJW8N
modified: 2024-11-12T15:39:32-05:00
---
# Table of Contents
- ## [[Effective C - Robert C Seacord|Effective C]]
	- ## 

# INTRODUCTION

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098182496/files/images/opener.jpg)

C was developed as a system programming language in the 1970s, and even after all this time, it remains incredibly popular. System languages are designed for performance and ease of access to the underlying hardware while providing high-level programming features. While other languages may offer newer language features, their compilers and libraries are typically written in C.

Carl Sagan once said, “If you wish to make an apple pie from scratch, you must first invent the universe.” The inventors of C did not invent the universe; they designed C to work with a variety of computing hardware and architectures that, in turn, were constrained by physics and mathematics. C is layered directly on top of computing hardware, making it more sensitive to evolving hardware features, such as vectorized instructions, than higher-level languages that typically rely on C for their efficiency.

According to the TIOBE index (_[https://www.tiobe.com/tiobe-index/](https://www.tiobe.com/tiobe-index/)_)—whose rankings are based on the number of skilled engineers, courses, and third-party vendors for each language—C has been either the most popular programming language or second most popular since 2001. The popularity of the C programming language can most likely be attributed to several tenets of the language referred to as the _spirit of C_:

- Trust the programmer. The C language assumes you know what you’re doing and lets you. This isn’t always a good thing (for example, if you don’t know what you’re doing).
- Don’t prevent the programmer from doing what needs to be done. Because C is a system programming language, it needs to handle a variety of low-level tasks.
- Keep the language small and simple. The language is designed to be close to the hardware and to have a small footprint.
- Provide only one way to do an operation. Also known as _conservation of mechanism_, the C language tries to limit the introduction of duplicate mechanisms.
- Make it fast, even if it isn’t guaranteed to be portable. Allowing you to write optimally efficient code is the top priority. The responsibility of ensuring that code is portable, safe, and secure is delegated to you, the programmer.

C is used as a target language for compilers to build operating systems, to teach fundamentals of computing, and for embedded and general-purpose programming.

There is a large amount of legacy code written in C. The C standards committee is extremely careful not to break existing code, providing a smooth pass for modernizing this code to take advantage of modern language features.

C is often used in embedded systems because it is a small and efficient language. Embedded systems are small computers that are embedded in other devices, such as cars, appliances, and medical devices.

Your favorite programming language and library are written in C (or were at one time). There are many libraries available for C. This makes it easy to find libraries that can be used to perform common tasks.

Overall, C is a powerful and versatile language that is still widely used today. It is a good choice for programmers who need a fast, efficient, and portable language.

## A Brief History of C

The C programming language was developed in the early 1970s at Bell Labs as a system implementation language for the nascent Unix operating system and remains incredibly popular today (Ritchie 1993). System languages are designed for performance and ease of access to the underlying hardware while providing high-level programming features. While other languages may offer newer language features, their compilers and libraries are typically written in C. It serves as a lingua franca for translating between various systems and languages.

C was first described in 1978 by Kernighan and Ritchie in the book _The C Programming Language_ (Kernighan and Ritchie 1988). It is now defined by revisions of the ISO/IEC 9899 standard (ISO/IEC 2024) and other technical specifications. The C standards committee is the steward of the C programming language, working with the broader community to maintain and evolve the C language. In 1983, the American National Standards Institute (ANSI) formed the X3J11 committee to establish a standard C specification, and in 1989, the C standard was ratified as ANSI X3.159-1989, “Programming Language C.” This 1989 version of the language is referred to as _ANSI C_ or _C89_.

In 1990, the ANSI C standard was adopted (unchanged) by a joint technical committee of the International Organization for Standardization (ISO) and the International Electrotechnical Commission (IEC) and published as the first edition of the C standard, C90 (ISO/IEC 9899:1990). The second edition of the C standard, C99, was published in 1999 (ISO/IEC 9899:1999), and a third edition, C11, in 2011 (ISO/IEC 9899:2011). The fourth version, published in 2018 as C17 (ISO/IEC 9899:2018), repairs defects in C11. The latest version of the C standard (as of this writing) is the fifth version, published in 2024 as C23 (ISO/IEC 9899:2024). As of September 2023, I am the convenor of ISO/IEC JTC1/SC22/WG14, the international standardization working group for the programming language C.

In the 20 years the TIOBE Programming Community index has tracked programming language popularity, C has remained in first or second place (TIOBE Index 2022).

## The C Standard

The C standard (ISO/IEC 9899:2024) defines the language and is the final authority on language behavior. While the standard can be obscure to impenetrable, you need to understand it if you intend to write code that’s portable, safe, and secure. The C standard provides a substantial degree of latitude to implementations to allow them to be optimally efficient on various hardware platforms. _Implementations_ is the term the C standard uses to refer to compilers and is defined as follows:

A particular set of software, running in a particular translation environment under particular control options, that performs translation of programs for, and supports execution of functions in, a particular execution environment.

This definition indicates that each compiler with a particular set of command line flags, along with the C standard library, is considered a separate implementation, and different implementations can have significantly different _implementation-defined behavior_. This is noticeable in GNU Compiler Collection (GCC), which uses the -std= flag to determine the language standard. Possible values for this option include c89, c90, c99, c11, c17, and c23. The default depends on the version of the compiler. If no C language dialect options are given, the default for GCC 13 is -std=gnu17, which provides extensions to the C language. For portability, specify the standard you’re using. For access to new language features, specify a recent standard. C23 features have been available since GCC 11. To enable C23 support, add the compiler option -std=c23 (or possibly -std=c2x). All the examples in this book are written for C23.

Because implementations have such a range of behaviors, and because some of these behaviors are undefined, you can’t understand the C language by just writing simple test programs to examine the behavior. (If you want to try this, the Compiler Explorer is an excellent tool; see _[https://godbolt.org](https://godbolt.org/)_.) The behavior of the code can vary when compiled by a different implementation on different platforms or even the same implementation using a different set of flags or a different C standard library implementation. Code behavior can even vary between _versions_ of a compiler. The C standard specifies which behaviors are guaranteed for all implementations and where you need to plan for variability. This is mostly a concern when developing portable code but can also affect the security and safety of your code.

## The CERT C Coding Standard

_The CERT__®_ _C Coding Standard: 98 Rules for Developing Safe, Reliable, and Secure Systems_, 2nd edition (Addison-Wesley Professional, 2014), is a reference book I wrote while managing the secure coding team at the Software Engineering Institute at Carnegie Mellon University. The book contains examples of common C programming mistakes and how to correct them. Throughout this book, we reference some of those rules as a source for detailed information on specific C language programming topics.

## Common Weakness Enumeration

MITRE’s Common Weakness Enumeration (CWE) is a list of common hardware and software weaknesses that can be used to identify weaknesses in source code and operational systems. The CWE list is maintained by a community project with the goals of understanding flaws in software and hardware and creating automated tools that can be used to identify, fix, and prevent those flaws. Occasionally, we will reference specific CWEs in this book when discussing classes of defects that can lead to security vulnerabilities. For more information on CWE, see _[https://cwe.mitre.org](https://cwe.mitre.org/)_.

## Who This Book Is For

This book is an introduction to the C language. It is written to be as accessible as possible to anyone who wants to learn C programming, without dumbing it down. In other words, we didn’t overly simplify C programming in the way many other introductory books and courses might. These overly simplified references will teach you how to compile and run code, but the code might still be wrong. Developers who learn how to program C from such sources will typically develop substandard, flawed, insecure code that will eventually need to be rewritten (often sooner than later). Hopefully, these developers will eventually benefit from senior developers in their organizations who will help them unlearn these harmful misconceptions about programming in C and help them start developing professional-quality C code. On the other hand, this book will quickly teach you how to develop correct, portable, professional-quality code; build a foundation for developing security- critical and safety-critical systems; and perhaps teach you some things that even the senior developers at your organization don’t know.

_Effective C: An Introduction to Professional C Programming_, 2nd edition, is a concise introduction to essential C language programming that will soon have you writing programs, solving problems, and building working systems. The code examples are idiomatic and straightforward. You’ll also learn about good software engineering practices for developing correct, secure C code.

In this book, you’ll learn about essential programming concepts in C and practice writing high-quality code with exercises for each topic. Code listings from this book and additional materials can be found on GitHub at _[https://github.com/rcseacord/effective-c](https://github.com/rcseacord/effective-c)_. Go to this book’s page at _[https://nostarch.com/effective-c-2nd-edition](https://nostarch.com/effective-c-2nd-edition)_ or to _[http://www.robertseacord.com](http://www.robertseacord.com/)_ to check for updates and additional material, or contact me if you have additional questions or are interested in training.

## What’s in This Book

This book starts with an introductory chapter that covers just enough material to get you programming right from the start. After that, we circle back and examine the basic building blocks of the language. The book culminates with two chapters that will show you how to compose real-world systems from these basic building blocks and how to debug, test, and analyze the code you’ve written. The chapters are as follows:

**[Chapter 1](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter1.xhtml): Getting Started with C** You’ll write a simple C program to become familiar with using the main function. You’ll also look at a few options for editors and compilers.

**[Chapter 2](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter2.xhtml): Objects, Functions, and Types** This chapter explores basics like declaring variables and functions. You’ll also investigate the principles of using basic types.

**[Chapter 3](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter3.xhtml): Arithmetic Types** You’ll learn about the integer and floating-point arithmetic data types.

**[Chapter 4](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter4.xhtml): Expressions and Operators** You’ll learn about operators and how to write simple expressions to perform operations on various object types.

**[Chapter 5](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter5.xhtml): Control Flow** You’ll learn how to control the order in which individual statements are evaluated. We’ll introduce expression and compound statements that define the work to be performed. We’ll then cover the control statements that determine which code blocks are executed and in what order: selection, iteration, and jump statements.

**[Chapter 6](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter6.xhtml): Dynamically Allocated Memory** You’ll learn about dynamically allocated memory, which is allocated from the heap at runtime. Dynamically allocated memory is useful when the exact storage requirements for a program are unknown before runtime.

**[Chapter 7](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter7.xhtml): Characters and Strings** This chapter covers the various character sets, including ASCII and Unicode, that can be used to compose strings. You’ll learn how strings are represented and manipulated using the legacy functions from the C standard library, the bounds-checking interfaces, and POSIX and Windows application programming interfaces (APIs).

**[Chapter 8](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter8.xhtml): Input/Output** This chapter will teach you how to perform input/output (I/O) operations to read data from, or write data to, terminals and filesystems. I/O involves all the ways information enters or exits a program. We’ll cover techniques that make use of C standard streams and POSIX file descriptors.

**[Chapter 9](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter9.xhtml): Preprocessor** You’ll learn how to use the preprocessor to include files, define object- and function-like macros, and conditionally include code based on implementation-specific features.

**[Chapter 10](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter10.xhtml): Program Structure** You’ll learn how to structure your program into multiple translation units consisting of both source and include files. You’ll also learn how to link multiple object files together to create libraries and executable files.

**[Chapter 11](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/chapter11.xhtml): Debugging, Testing, and Analysis** This chapter describes tools and techniques for producing error-free programs, including compile-time and runtime assertions, debugging, testing, static analysis, and dynamic analysis. The chapter also discusses which compiler flags are recommended for use in different phases of the software development process.

**[Appendix](https://learning.oreilly.com/library/view/effective-c-2nd/9781098182496/xhtml/appendix.xhtml): The Fifth Edition of the C Standard (C23)** This appendix enumerates some of the additions and changes in C23. It’s a convenient way to learn what’s new in C and to identify changes from the previous C standard (C17).This book is updated from the previous edition to cover the features and behaviors of C23. According to 2022 polling data from JetBrains (_[https://www.jetbrains.com/lp/devecosystem-2022/c/](https://www.jetbrains.com/lp/devecosystem-2022/c/)_), 44 percent of C programmers use C99, 33 percent use C11, 16 percent use C17, and 15 percent use an embedded version of C.

You’re about to embark on a journey from which you will emerge a newly minted but professional C developer.