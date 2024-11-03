---
id: 01JBMASX1409RMB46MTT50H7T5
modified: 2024-11-01T13:01:51-04:00
---
# 3. Introduction to POSIX Standards and System-Level APIs

Sri Manikanta Palakollu[1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_3_Chapter.xhtml#Aff2) 

(1)

freelance, Hanuman Junction, Hanuman Junction, 521105, Andhra Pradesh, India

POSIX standards help maintain compatibility between operating systems. System-level APIs help to efficiently develop applications very within a short development period. In this chapter, you learn about the following introductory topics.

- The POSIX standard
    
- POSIX support
    
- Introduction to APIs
    
- Importance of APIs
    
- Built-in C Standard APIs
    

## Understanding POSIX Standards

POSIX is the acronym for _Portable Operating System Interface_ on Unix-based operating systems. They are IEEE standards to formalize certain common standards in all operating systems in the enterprise market.

In the olden days, programmers struggled to develop an application for computer systems. Before POSIX standards, there were no common standards for developing a computer operating system model, so developers needed to develop their applications for every model—from scratch—to be compatible with all systems. This increased development time and the cost of an application. Debugging was also very difficult because of new bugs and issues in every new computer model, which caused a lot of problems for developers.

To avoid these issues, IEEE introduced standard rules to practice when developing new computer models. These standards helped develop all kinds of applications. Developers no longer need to develop new code for new system models. These standard rules are classified into four main categories.

- POSIX.1
    
- POSIX.1b
    
- POSIX.1c
    
- POSIX.2
    

### POSIX.1 Standards

POSIX.1 standards deal with the core services of all operating system models. The following are features included in this standard.

- Process creation and control
    
- Process triggers
    
- Files and directory operations
    
- Segmentation faults
    
- Memory faults
    
- Floating-point exceptions
    
- Pipes
    
- Signals
    
- Standard C library implementation
    
- Standard I/O interface and control
    

These are some of the core features that the IEEE addressed to improve the interface of all operating systems.

### POSIX.1b Standards

Along with the POSIX.1 standards, there are additional core features specifically related to real-time application development. These POSIX.1b rules include the following topics.

- CPU scheduling algorithms
    
- Message passing
    
- Shared memory
    
- Semaphore
    
- Memory-locking interfaces
    
- Synchronous and asynchronous data transfer interfaces
    

All the core features are covered in upcoming chapters.

### POSIX.1c Standards

This standard category includes core features related to multithreading.

- Thread creation
    
- Thread control
    
- Thread deletion
    
- Thread synchronization
    
- Thread scheduling
    

### POSIX.2 Standards

POSIX.2 standards address the core functionality features of an operating system.

- uname
    
- tty
    
- cd
    
- ls
    
- mkdir
    
- echo
    
- cp
    
- rm
    
- mv
    

This standard list includes all common utilities and the tools that are commonly used by the users.

## POSIX Support

All OS models do not use POSIX standards. macOS uses the complete POSIX standards for its operating system, but most Linux distros use the Linux Standard Base (LSB), which includes more powerful features than POSIX. It is a superset of POSIX standards but also independent of POSIX standards. The Windows 10 operating system uses POSIX as a subsystem with the same standard features.

## Introduction to APIs

API stands for _application programming interface_ . It is a collection of protocols and subroutines that communicate between various systems and subsystems. APIs make developers’ lives a lot easier. There are several types of APIs.

- Public API or open API
    
- Private API
    
- Partner API
    
- Composite API
    

All the APIs are normal web services standard APIs, but this book concentrates on system-level APIs. System-level APIs are normal programs written by developers to improve the core functionality of a programming language. A system-level API has two different modes: user mode and supervisor mode (see Figure [3-1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_3_Chapter.xhtml#Fig1)).

![../images/497677_1_En_3_Chapter/497677_1_En_3_Fig1_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484263211/files/images/497677_1_En_3_Chapter/497677_1_En_3_Fig1_HTML.jpg)

Figure 3-1

Working API

### User Mode

In user mode, developers typically develop programs to incorporate or manipulate custom activities in the system. The activities performed at the user level is done with the help of System-level API. The activities include file creation, directory creation, and similar basic activities.

## Supervisor Mode

In supervisor mode, system calls perform the actions written by the developers. These built-in functions and libraries are very helpful when it comes to performing system-level tasks. The system call in supervisor mode executes the calls that are made from the user mode. Programs are written to perform some action(s).

### The Importance of System-Level APIs

Built-in system APIs improve the performance of the developed system. There is no need to write the code from scratch. Built-in functions and libraries are not required to perform testing because there are tested rigorously before release. The following are some of the benefits of using these functions.

- **Performance**: These libraries are under active development, and developers continuously try to improve the performance of existing functions. Also, they use standard algorithms, such as standard sorting and searching algorithms, to get the best performance.
    
- **Reliable code**: There are fewer errors because most of the activities are done with built-in functions, and because the libraries are under active development.
    
- **Reduces development time**: Most of the code is written by developers to perform a particular activity, which reduces development time because there is no need to write code from scratch.
    
- **System-independent**: C-program compiled binaries are system-dependent, but these built-in libraries are system-independent, which means that they don’t depend on the system. All the built-in functions work the same on all operating systems.
    

### Built-in APIs in C

Standard libraries are used in the rest of this book. To perform certain system core activities with a specific programming language, you need to check whether the language has a standard library that allows you to interact with your OS or not. Luckily, the C language provides support for a system’s core functionality. Table [3-1](https://learning.oreilly.com/library/view/practical-system-programming/9781484263211/html/497677_1_En_3_Chapter.xhtml#Tab1) lists the most commonly used libraries.

Table 3-1

Most Common Libraries

|Library|Functionality|
|---|---|
|<stdio.h>|This library contains all the standard input and output operation functions: 10 macros and 41 functions. The most popular functions are printf and scanf.|
|<stdlib.h>|This is a standard library in C that is mainly used for general-purpose programming. It contains the memory allocation and deallocation functions that perform dynamic activities.|
|<unistd.h>|This library provides the standard interface for the POSIX API.|
|<sys/types.h>|This library contains standard derived data types, which are helpful in system-level programming.|
|<signal.h>|This library handles the signal activities in an operating system.|
|<time.h>|This library provides support for time and date activities in a standard manner.|
|<sys/stat.h>|This library determines the file system status and activity.|
|<fcntl.h>|This library is a part of the POSIX API that manipulates files, such as changing permissions.|
|<sys/ipc.h>|This library deals with three major core tasks that include interprocess communication activity (i.e., message queues, semaphores, and shared memory).|
|<sys/msg.h>|This library works with the <sys/ipc.h> library to deal with IPC activity.|
|<semaphore.h>|This library performs the semaphore activity in an operating system. It is also a part of the POSIX library.|
|<sys/shm.h>|This library performs shared memory activities.|
|<sys/wait.h>|This library places a process into a waiting state.|
|<stdargs.h>|This library handles the variable argument activity that takes input directly from the command-line.|

## Summary

This chapter discussed topics related to the POSIX environment and the various C built-in libraries. You were also introduced to system-level APIs.