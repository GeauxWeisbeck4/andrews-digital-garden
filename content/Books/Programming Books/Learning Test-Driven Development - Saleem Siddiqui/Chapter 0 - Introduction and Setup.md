---
id: 01JBCGRBBDYCB2ST91N76RRPR6
title: Chapter 0 - Introduction and Setup
modified: 2024-10-29T12:11:56-04:00
tags:
  - test-driven-development
  - programming
  - books
  - javascript
  - go-lang
  - python
---
# Chapter 0: Introduction and Setup

> Squeaky clean code is critical to success.
> 
> Ron Jeffries, “Clean Code: A Learning,” Aug 23, 2017, _ronjeffries.com_

Before we start our journey into the demanding and rewarding world of test-driven development, we need to ensure we have a working development environment. This chapter is all about preparing and setting things up.

# Setting Up Your Development Environment

Regardless of which reading pathway you follow (see [Figure P-2](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/preface01.html#figp2)), you need a clean development environment to follow this book. The rest of the book assumes that you have set up the development environment as described in this section.

###### Important

Regardless of which of Go, JavaScript, or Python you start with, set up your development environment as described in this section.

## Common Setup

### Folder structure

Create a folder that will be the root for _all_ the source code we’ll write in this book. Name it something that will be clear and unambiguous to you weeks from now, e.g., _tdd-project_.

Under this folder, create a set of folders as follows:

tdd-project
├── go
├── js
└── py

Create all these folders before you write the first line of code, even if you’re planning to follow this book in multiple passes, one language at a time. Creating this folder structure provides the following benefits:

1. It keeps code in the three languages separate yet in close proximity to each other.
    
2. It ensures that _most_ commands in this book will work without changes.
    
    - Commands that deal with fully qualified file/folder names are an exception—and such commands are rare. One of them is in this section.
        
3. It allows easy adoption of advanced features, such as continuous integration, across all three languages.
    
4. It matches the folder structure in the [accompanying code repository](https://github.com/saleem/tdd-book-code). This can be useful for comparing and contrasting your code as it evolves.
    

Throughout the rest of this book, the term _TDD Project Root_ is used to refer to the root folder containing all the source code—named `tdd-project` above. The folders named `go`, `js`, and `py` are referred to by these very names—the meaning is clear from the context.

###### Important

_TDD Project Root_ is the name used to refer to the folder containing all the source code developed in this book. It’s the parent of three folders named `go`, `js`, and `py`.

Declare an environment variable named `TDD_PROJECT_ROOT` and set its value to the fully qualified name of the _TDD Project Root_ folder. Doing this once in each shell (or better yet, once in your shell initialization script such as the `.bashrc` file) ensures that all subsequent commands work seamlessly.

```
export 
```

For example, on my macOS system, the fully qualified path for the TDD_PROJECT_ROOT is `/Users/saleemsiddiqui/code/github/saleem/tdd-project`.

##### Removing Tedium

You will need to define the environment variable `TDD_PROJECT_ROOT` in every new shell you launch. If you find this cumbersome, you may set it once in the appropriate configuration file. _How_ you set it varies from one operating system to another and from one shell to another. For Bash-like shells that we will use in this book, environment variables can be defined in a configuration file; although the details still vary. For example, on most Linuxes (and on macOS), you can add the `export TDD_PROJECT_ROOT=...` statement in a [file named `.bashrc` in your home folder](https://oreil.ly/SMmFc). If you’re using Git BASH on Windows—as described later in this chapter—you may need to [use the `.bash_profile` file instead](https://oreil.ly/pkgxg).

In short: removing tedium in your work is a good thing. Use an appropriate mechanism to define the environment variables reliably and consistently in all shells you use throughout this book.

### Text editor or IDE

We’ll need a text editor to edit source files. An _integrated development environment_ (_IDE_) can help by providing a single tool within which we can edit, compile, and test code in multiple languages. However, this is a matter of choice and personal preference; choose what works best for _you_.

[Appendix A](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/app01.html#appendix_a) describes IDEs in more detail.

### Shell

We’ll need a shell—a command-line interpreter—to run our tests, examine the output, and carry out other tasks. Like IDEs, shell choices are many and often the subject of exuberant opinion sharing among developers. This book assumes a _Bash-like shell_ for the commands that need to be typed. On most—if not all—Unix-like operating systems (and on macOS), a Bash shell is readily available.

For Windows, shells like [Git BASH](https://gitforwindows.org/) are available. On Windows 10, the [Windows Subsystem for Linux](https://oreil.ly/UZ0KU) provides native support for the Bash shell, among many other “Linux goodies.” Either of these options, or something similar, is sufficient (and necessary) to follow the code examples in this book.

[Figure 0-1](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/introduction01.html#fig0001) shows a Bash-like shell with the results of a command typed in it.

![An iTerm shell on a macOS operating system](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_05.png)

###### Figure 0-1. A Bash-like shell, like the one shown here, is needed to follow the coding examples in this book

### Git

[Chapter 13](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch13.html#chapter_13) introduces the practice of continuous integration (CI) using GitHub Actions. To follow the content of that chapter, we need to create a GitHub project of our own and push code to it.

Git is an open source distributed version control system. GitHub is a collaborative internet hosting platform that allows people to preserve and share the source code of their projects with each other.

###### Important

[Git](https://git-scm.com/) is a free, open source, distributed version control system. [GitHub](https://www.github.com/) is a code-sharing platform that uses Git.

To ensure we can adopt continuous integration, we’ll do some preparation now and defer some work until [Chapter 13](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch13.html#chapter_13). Specifically, we’ll set up the Git version control system on our development environment. We’ll postpone the creation of a GitHub project until [Chapter 13](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch13.html#chapter_13).

First, [download and install the Git version control system](https://git-scm.com/downloads). It is available for macOS, Windows, and Linux/Unix. After you install it, verify that it works by typing `git` `--version` on a terminal window and hitting Enter. You should see the installed version of Git in response, as shown in [Figure 0-2](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/introduction01.html#fig0002).

![Verify that Git is installed on a shell](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_06.png)

###### Figure 0-2. Verify that Git is installed by typing `git --version` and hitting Enter on a shell

Next, we’ll create a new Git project in our `TDD_PROJECT_ROOT`. In a shell window, type the following commands:

```
cd
```

This should produce an output saying `Initialized empty Git repository in` `_/your/fully/qualified/project/path/_.git/`. This creates a shiny new (and currently empty) Git repository in our `TDD_PROJECT_ROOT`. We should have these folders under our `TDD-PROJECT-ROOT` folder now:

tdd-project
├── .git
├── go
├── js
└── py

The `.git` folder is used by Git for bookkeeping. There is no need to make any changes to its contents.

As we write source code in the following chapters, we will periodically commit our changes to this Git repository. We’ll use the Git CLI (command line interface) to do this.

###### Important

We’ll frequently commit our code changes to the Git repository in the rest of this book. To highlight this, we’ll use the ![Git](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/Git-Logo-2Color.png) Git icon.

## Go

We need to install Go version 1.17 to follow this book. This version [is available to download](https://golang.org/dl) for different operating systems.

To verify that Go is correctly installed, type `go version` on a shell and hit Enter. This should print the version number of your Go installation. See [Figure 0-3](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/introduction01.html#fig0003).

![Verify that Go is working on a shell](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_07.png)

###### Figure 0-3. Verify that Go is working by typing `go version` and hitting Enter on a shell

We also need to set a couple of Go-specific environment variables:

1. The `GO111MODULE` environment variable should be set to `on`.
    
2. The `GOPATH` environment variable _should not_ include the `TDD_PROJECT_ROOT` or any folder under in, such as the `go` folder.
    

Execute these two lines of code in the shell:

```
export 
```

We need to create a bare-bones `go.mod` file to get ready to write code. These are the commands to do it:

```
cd
```

This will create a file named `go.mod` whose contents should be:

```
module
```

For all Go development from this point on, make sure that the shell is in the `go` folder under `TDD_PROJECT_ROOT`.

###### Important

For the Go code in this book, make sure to first enter `cd $TDD_PROJECT_ROOT/go` before running any Go commands.

### A quick word on Go package management

Go’s package management is in the midst of a seismic shift. The old style—which used the `GOPATH` environment variable—is being phased out in favor of the newer style using a `go.mod` file. The two styles are largely incompatible with each other.

The two environment variables we defined above, and the bare-bones `go.mod` file we generated, ensure that the Go tools can work correctly with our source code, especially when we create packages. We’ll create Go packages in [Chapter 5](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch05.html#chapter_05).

## JavaScript

We need Node.js v14 (“Fermium”) or v16 to follow this book. Both these versions are [available from the Node.js website](https://nodejs.org/en/download) for different operating systems.

To verify that Node.js is correctly installed, type `node -v` on a shell and hit Enter. The command should print a one-line message, listing the version of Node.js. See [Figure 0-4](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/introduction01.html#fig0004).

![Verify that Node.js is working on a shell](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_08.png)

###### Figure 0-4. Verify that Node.js is working by typing `node -v` and hitting Enter on a shell

### A quick word on testing libraries

There are several unit-testing frameworks in the Node.js ecosystem. By and large, they are excellent for writing tests and doing TDD. However, this book eschews _all_ of them. Its code uses the `assert` NPM package for assertions, and a simple class with methods to organize the tests. The simplicity is to keep our focus on the _practice and semantics_ of TDD instead of the _syntax_ of any one library. [Chapter 6](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch06.html#chapter_06) describes the organization of tests in more detail. [Appendix B](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/app02.html#appendix_b) enumerates the testing frameworks and the detailed reasons for not using any of them.

### Another quick word, on JavaScript package management

Similar to testing frameworks, JavaScript has many ways to define packages and dependencies. This book uses the CommonJS style. In [Chapter 6](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch06.html#chapter_06), there is a discussion of the other styles: the ES6 and UMD styles are shown in detail with source code, and the AMD style more briefly, without source code.

## Python

We need Python 3.10 to follow this book, which is [available from the Python website](https://oreil.ly/xNLPa) for different operating systems.

The Python language underwent significant changes between “Python 2” and “Python 3.” While an older version of Python 3 (e.g., 3.6) may work, _any_ version of Python 2 will be inadequate for the purpose of following this book.

It is possible that you have Python 2 already installed on your computer. For example, many macOS operating systems (including Big Sur) come bundled with Python 2. It is not necessary (or recommended) to _uninstall_ Python 2 to follow this book; however, it _is_ necessary to ensure that Python 3 is the version that’s used.

To prevent ambiguity, this book uses `python3` explicitly as the name of the executable in commands. It is possible—although also unnecessary—to “alias” the `python` command to refer to Python 3.

Here’s a simple way to find out which command you need to type to ensure that Python 3 is used. Type `python --version` on a shell and hit Enter. If you get something starting with `Python 3`, you’re in good shape. If you get something like `Python 2`, you may need to explicitly type in `python3` for all the commands in this book.

[Figure 0-5](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/introduction01.html#fig0005) shows a development environment with _both_ Python 2 and Python 3.

![Verify that Python 3 is working on a shell](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_09.png)

###### Figure 0-5. Verify that Python 3 is installed and the command you need to type to use it (`python3` as shown here)

###### Important

Use Python 3 to follow the code in this book. Do _not_ use Python 2—it won’t work.

[Figure 0-6](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/introduction01.html#fig0006) shows a mnemonic to simplify the preceding Python version rigmarole!

![For all the Python code in this book, remember this easy and cheesy rhyme! Python 2: boo! Python 3: glee!](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098106461/files/assets/ltdd_10.png)

###### Figure 0-6. Simple mnemonic to clarify which version of Python is needed for this book!

# Where We Are

In this preliminary chapter, we got acquainted with the toolchain we’ll need to start writing our code in a test-driven fashion. We also learned how to prepare our development environment and to verify that it is in working condition.

Now that we know what this book is about, what’s in it, how to read it, and most importantly how to set up our working environment to follow it, we are ready to solve our problem, chiseling one feature at time, driven forward by tests. We’ll commence that journey in [Chapter 1](https://learning.oreilly.com/library/view/learning-test-driven-development/9781098106461/ch01.html#chapter_01). Let’s roll!