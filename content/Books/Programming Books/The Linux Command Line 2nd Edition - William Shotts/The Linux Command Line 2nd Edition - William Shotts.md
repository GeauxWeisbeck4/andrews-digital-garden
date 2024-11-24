---
title: The Linux Command Line, 2nd Edition
subtitle: A Complete Introduction
author: William Shotts
authors: William Shotts
category: Computers
categories: Computers
publisher: No Starch Press
publishDate: 2019-03-05
totalPage: 504
coverUrl: http://books.google.com/books/content?id=dIBxDwAAQBAJ&printsec=frontcover&img=1&zoom=1&edge=curl&source=gbs_api
coverSmallUrl: http://books.google.com/books/content?id=dIBxDwAAQBAJ&printsec=frontcover&img=1&zoom=5&edge=curl&source=gbs_api
description: "You've experienced the shiny, point-and-click surface of your Linux computer--now dive below and explore its depths with the power of the command line. The Linux Command Line takes you from your very first terminal keystrokes to writing full programs in Bash, the most popular Linux shell (or command line). Along the way you'll learn the timeless skills handed down by generations of experienced, mouse-shunning gurus: file navigation, environment configuration, command chaining, pattern matching with regular expressions, and more. In addition to that practical knowledge, author William Shotts reveals the philosophy behind these tools and the rich heritage that your desktop Linux machine has inherited from Unix supercomputers of yore. As you make your way through the book's short, easily-digestible chapters, you'll learn how to: • Create and delete files, directories, and symlinks • Administer your system, including networking, package installation, and process management • Use standard input and output, redirection, and pipelines • Edit files with Vi, the world's most popular text editor • Write shell scripts to automate common or boring tasks • Slice and dice text files with cut, paste, grep, patch, and sed Once you overcome your initial &quot;shell shock,&quot; you'll find that the command line is a natural and expressive way to communicate with your computer. Just don't be surprised if your mouse starts to gather dust."
link: https://play.google.com/store/books/details?id=dIBxDwAAQBAJ
previewLink: http://books.google.com/books?id=dIBxDwAAQBAJ&printsec=frontcover&dq=the+linux+command+line&hl=&as_pt=BOOKS&cd=1&source=gbs_api
isbn13: 9781593279530
isbn10: 1593279531
id: 01JCH5HRP8WQ093NTC1BJ5PRJV
modified: 2024-11-12T16:50:43-05:00
---
# Table of Contents
- 

- ## Introduction

![image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781492071235/files/images/common01.jpg)

I want to tell you a story. No, not the story of how, in 1991, Linus Torvalds wrote the first version of the Linux kernel. You can read that story in lots of Linux books. Nor am I going to tell you the story of how, some years earlier, Richard Stallman began the GNU Project to create a free Unix-like operating system. That’s an important story too, but most other Linux books have that one, as well.

No, I want to tell you the story of how you take back control of your computer.

When I began working with computers as a college student in the late 1970s, there was a revolution going on. The invention of the microprocessor had made it possible for ordinary people like you and me to actually own a computer. It’s hard for many people today to imagine what the world was like when only big business and big government ran all the computers. Let’s just say, you couldn’t get much done.

Today, the world is very different. Computers are everywhere, from tiny wristwatches to giant data centers to everything in between. In addition to ubiquitous computers, we also have a ubiquitous network connecting them together. This has created a wondrous new age of personal empowerment and creative freedom, but over the last couple of decades something else has been happening. A few giant corporations have been imposing their control over most of the world’s computers and deciding what you can and cannot do with them. Fortunately, people from all over the world are doing something about it. They are fighting to maintain control of their computers by writing their own software. They are building Linux.

Many people speak of “freedom” with regard to Linux, but I don’t think most people know what this freedom really means. Freedom is the power to decide what your computer does, and the only way to have this freedom is to know what your computer is doing. Freedom is a computer that is without secrets, one where everything can be known if you care enough to find out.

### **Why Use the Command Line?**

Have you ever noticed in the movies when the “superhacker”—you know, the guy who can break into the ultra-secure military computer in less than 30 seconds—sits down at the computer, he never touches a mouse? It’s because filmmakers realize that we, as human beings, instinctively know the only way to really get anything done on a computer is by typing on a keyboard!

Most computer users today are familiar only with the _graphical user interface_ (_GUI_) and have been taught by vendors and pundits that the _command line interface_ (_CLI_) is a terrifying thing of the past. This is unfortunate because a good command line interface is a marvelously expressive way of communicating with a computer in much the same way the written word is for human beings. It’s been said that “graphical user interfaces make easy tasks easy, while command line interfaces make difficult tasks possible,” and this is still very true today.

Since Linux is modeled after the Unix family of operating systems, it shares the same rich heritage of command line tools as Unix. Unix came into prominence during the early 1980s (although it was first developed a decade earlier), before the widespread adoption of the graphical user interface and, as a result, developed an extensive command line interface instead. In fact, one of the strongest reasons early adopters of Linux chose it over, say, Windows NT was the powerful command line interface that made the “difficult tasks possible.”

### **What This Book Is About**

This book is a broad overview of “living” on the Linux command line. Unlike some books that concentrate on just a single program, such as the shell program bash, this book will try to convey how to get along with the command line interface in a larger sense. How does it all work? What can it do? What’s the best way to use it?

**This is not a book about Linux system administration.** While any serious discussion of the command line will invariably lead to system administration topics, this book touches on only a few administration issues. It will, however, prepare the reader for additional study by providing a solid foundation in the use of the command line, an essential tool for any serious system administration task.

**This book is Linux-centric.** Many other books try to broaden their appeal by including other platforms such as generic Unix and macOS. In doing so, they “water down” their content to feature only general topics. This book, on the other hand, covers only contemporary Linux distributions. Ninety-five percent of the content is useful for users of other Unix-like systems, but this book is highly targeted at the modern Linux command line user.

### **Who Should Read This Book**

This book is for new Linux users who have migrated from other platforms. Most likely you are a “power user” of some version of Microsoft Windows. Perhaps your boss has told you to administer a Linux server, or you’re entering the exciting new world of single board computers (SBC) such as the Raspberry Pi. You may just be a desktop user who is tired of all the security problems and wants to give Linux a try. That’s fine. All are welcome here.

That being said, there is no shortcut to Linux enlightenment. Learning the command line is challenging and takes real effort. It’s not that it’s so hard, but rather it’s so _vast_. The average Linux system has literally _thousands_ of programs you can employ on the command line. Consider yourself warned; learning the command line is not a casual endeavor.

On the other hand, learning the Linux command line is extremely rewarding. If you think you’re a “power user” now, just wait. You don’t know what real power is—yet. And, unlike many other computer skills, knowledge of the command line is long-lasting. The skills learned today will still be useful 10 years from now. The command line has survived the test of time.

It is also assumed that you have no programming experience, but don’t worry, we’ll start you down that path as well.

### **What’s in This Book**

This material is presented in a carefully chosen sequence, much like a tutor sitting next to you guiding you along. Many authors treat this material in a “systematic” fashion, exhaustively covering each topic in order. This makes sense from a writer’s perspective but can be very confusing to new users.

Another goal is to acquaint you with the Unix way of thinking, which is different from the Windows way of thinking. Along the way, we’ll go on a few side trips to help you understand why certain things work the way they do and how they got that way. Linux is not just a piece of software; it’s also a small part of the larger Unix culture, which has its own language and history. I might throw in a rant or two, as well.

This book is divided into four parts, each covering some aspect of the command line experience.

- **[Part 1](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part01.xhtml#part01), “[Learning the Shell](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part01.xhtml#part01),”** starts our exploration of the basic language of the command line including such things as the structure of commands, file system navigation, command line editing, and finding help and documentation for commands.
- **[Part 2](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part02.xhtml#part02), “[Configuration and the Environment](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part02.xhtml#part02),”** covers editing configuration files that control the computer’s operation from the command line.
- **[Part 3](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part03.xhtml#part03), “[Common Tasks and Essential Tools](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part03.xhtml#part03),”** explores many of the ordinary tasks that are commonly performed from the command line. Unix-like operating systems, such as Linux, contain many “classic” command line programs that are used to perform powerful operations on data.
- **[Part 4](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part04.xhtml#part04), “[Writing Shell Scripts](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part04.xhtml#part04),”** introduces shell programming, an admittedly rudimentary but easy-to-learn technique for automating many common computing tasks. By learning shell programming, you will become familiar with concepts that can be applied to many other programming languages.

### **How to Read This Book**

Start at the beginning of the book and follow it to the end. It isn’t written as a reference work; it’s really more like a story with a beginning, middle, and end.

#### **_Prerequisites_**

To use this book, all you will need is a working Linux installation. You can get this in one of two ways:

**Install Linux on a (not so new) computer.** It doesn’t matter which distribution you choose, though most people today start out with either Ubuntu, Fedora, or OpenSUSE. If in doubt, try Ubuntu first. Installing a modern Linux distribution can be ridiculously easy or ridiculously difficult depending on your hardware. I suggest a desktop computer that is a couple of years old and has at least 2GB of RAM and 6GB of free hard disk space. Avoid laptops and wireless networks if at all possible, as these are often more difficult to get working.

**Use a “live CD” or USB flash drive.** One of the cool things you can do with many Linux distributions is run them directly from a CD-ROM or USB flash drive without installing them at all. Just go into your BIOS setup and set your computer to boot from a CD-ROM drive or USB device and reboot. Using this method is a great way to test a computer for Linux compatibility prior to installation. The disadvantage is that it may be slow compared to having Linux installed on your hard drive. Both Ubuntu and Fedora (among others) have live versions.

Regardless of how you install Linux, you’ll need to have occasional superuser (i.e., administrative) privileges to carry out the lessons in this book.

After you have a working installation, start reading and follow along with your own computer. Most of the material in this book is “hands on,” so sit down and get typing!

**WHY I DON’T CALL IT “GNU/LINUX”**

In some quarters, it’s politically correct to call the Linux operating system the “GNU/Linux operating system.” The problem with “Linux” is that there is no completely correct way to name it because it was written by many different people in a vast, distributed development effort. Technically speaking, Linux is the name of the operating system’s kernel, nothing more. The kernel is very important, of course, since it makes the operating system go, but it’s not enough to form a complete operating system.

Enter Richard Stallman, the genius-philosopher who founded the Free Software movement, started the Free Software Foundation, formed the GNU Project, wrote the first version of the GNU C Compiler (gcc), created the GNU General Public License (the GPL), etc., etc., etc. He _insists_ that you call it “GNU/Linux” to properly reflect the contributions of the GNU Project. While the GNU Project predates the Linux kernel and the project’s contributions are extremely deserving of recognition, placing them in the name is unfair to everyone else who made significant contributions. Besides, I think “Linux/GNU” would be more technically accurate since the kernel boots first and everything else runs on top of it.

In popular usage, Linux refers to the kernel and all the other free and open source software found in the typical Linux distribution, that is, the entire Linux ecosystem, not just the GNU components. The operating system marketplace seems to prefer one-word names such as DOS, Windows, macOS, Solaris, Irix, and AIX. I have chosen to use the popular format. If, however, you prefer to use “GNU/Linux” instead, please perform a mental search-and-replace while reading this book. I won’t mind.

### **What’s New in the Second Edition**

While the basic structure and content remain the same, this edition of _The Linux Command Line_ is peppered with various refinements, clarifications, and modernizations, many of which are based on reader feedback. In addition, two particular improvements stand out. First, the book now assumes bash version 4._x_, which was not in wide use at the time of the original manuscript. This fourth major version of bash added several useful new features now covered in this edition. Second, [Part 4](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part04.xhtml#part04), “[Shell Scripting](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part04.xhtml#part04),” has been improved to provide better examples of good scripting practice. The scripts included in [Part 4](https://learning.oreilly.com/library/view/the-linux-command/9781492071235/xhtml/part04.xhtml#part04) have been revised to make them more robust, and I also fixed a few bugs ;-).

### **Your Feedback Is Needed!**

This book is an ongoing project, like many open source software projects. If you find a technical error, drop me a line at _[bshotts@users.sourceforge.net](mailto:bshotts@users.sourceforge.net)_.

Be sure to indicate the exact edition of the book you are reading. Your changes and suggestions may get into future releases.
- ## Chapter 2