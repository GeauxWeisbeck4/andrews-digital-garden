---
id: 01JAH06BSJ005PQSR1M45JQXHH
modified: 2024-10-18T19:46:08-04:00
title: Chapter One - Introducing Express
tags:
  - express
  - javascript
  - nodejs
  - books
  - programming
---
# Chapter 1. Introducing Express

# The JavaScript Revolution

Before I introduce the main subject of this book, it is important to provide a little background and historical context, and that means talking about JavaScript and Node. The age of JavaScript is truly upon us. From its humble beginnings as a client-side scripting language, not only has it become completely ubiquitous on the client side, but its use as a server-side language has finally taken off too, thanks to Node.

The promise of an all-JavaScript technology stack is clear: no more context switching! No longer do you have to switch mental gears from JavaScript to PHP, C#, Ruby, or Python (or any other server-side language). Furthermore, it empowers frontend engineers to make the jump to server-side programming. This is not to say that server-side programming is strictly about the language; there’s still a lot to learn. With JavaScript, though, at least the language won’t be a barrier.

This book is for all those who see the promise of the JavaScript technology stack. Perhaps you are a frontend engineer looking to extend your experience into backend development. Perhaps you’re an experienced backend developer like myself who is looking to JavaScript as a viable alternative to entrenched server-side languages.

If you’ve been a software engineer for as long as I have, you have seen many languages, frameworks, and APIs come into vogue. Some have taken off, and some have faded into obsolescence. You probably take pride in your ability to rapidly learn new languages, new systems. Every new language you come across feels a little more familiar: you recognize a bit here from a language you learned in college, a bit there from that job you had a few years ago. It feels good to have that kind of perspective, certainly, but it’s also wearying. Sometimes you want to just _get something done_, without having to learn a whole new technology or dust off skills you haven’t used in months or years.

JavaScript may seem, at first, an unlikely champion. I sympathize, believe me. If you told me in 2007 that I would not only come to think of JavaScript as my language of choice, but also write a book about it, I would have told you you were crazy. I had all the usual prejudices against JavaScript: I thought it was a “toy” language, something for amateurs and dilettantes to mangle and abuse. To be fair, JavaScript did lower the bar for amateurs, and there was a lot of questionable JavaScript out there, which did not help the language’s reputation. To turn a popular saying on its head, “Hate the player, not the game.”

It is unfortunate that people suffer this prejudice against JavaScript; it has prevented people from discovering how powerful, flexible, and elegant the language is. Many people are just now starting to take JavaScript seriously, even though the language as we know it now has been around since 1996 (although many of its more attractive features were added in 2005).

By picking up this book, you are probably free of that prejudice: either because, like me, you have gotten past it or because you never had it in the first place. In either case, you are fortunate, and I look forward to introducing you to Express, a technology made possible by a delightful and surprising language.

In 2009, years after people had started to realize the power and expressiveness of JavaScript as a browser scripting language, Ryan Dahl saw JavaScript’s potential as a server-side language, and Node.js was born. This was a fertile time for internet technology. Ruby (and Ruby on Rails) took some great ideas from academic computer science, combined them with some new ideas of its own, and showed the world a quicker way to build websites and web applications. Microsoft, in a valiant effort to become relevant in the internet age, did amazing things with .NET and learned not only from Ruby and JavaScript but also from Java’s mistakes, while borrowing heavily from the halls of academia.

Today, web developers have the freedom to use the very latest JavaScript language features without fear of alienating users with older browsers, thanks to transcompilation technologies like Babel. Webpack has become the ubiquitous solution for managing dependencies in web applications and ensuring performance, and frameworks such as React, Angular, and Vue are changing the way people approach web development, relegating declarative Document Object Model (DOM) manipulation libraries (such as jQuery) to yesterday’s news.

It is an exciting time to be involved in internet technology. Everywhere there are amazing new ideas (or amazing old ideas revitalized). The spirit of innovation and excitement is greater now than it has been in many years.

# Introducing Express

The Express website describes Express as a “minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.” What does that really mean, though? Let’s break that description down:

Minimal

This is one of the most appealing aspects of Express. Many times, framework developers forget that usually “less is more.” The Express philosophy is to provide the _minimal_ layer between your brain and the server. That doesn’t mean that it’s not robust or that it doesn’t have enough useful features. It means that it gets in your way less, allowing you full expression of your ideas, while at the same time providing something useful. Express provides you a minimal framework, and you can add in different parts of Express functionality as needed, replacing whatever doesn’t meet your needs. This is a breath of fresh air. So many frameworks give you _everything_, leaving you with a bloated, mysterious, and complex project before you’ve even written a single line of code. Often, the first task is to waste time carving off unneeded functionality or replacing the functionality that doesn’t meet requirements. Express takes the opposite approach, allowing you to add what you need when you need it.

Flexible

At the end of the day, what Express does is very simple: it accepts HTTP requests from a client (which can be a browser, a mobile device, another server, a desktop application…anything that speaks HTTP) and returns an HTTP response. This basic pattern describes almost everything connected to the internet, making Express extremely flexible in its applications.

Web application framework

Perhaps a more accurate description would be “server-side part of a web application framework.” Today, when you think of “web application framework,” you generally think of a single-page application framework like React, Angular, or Vue. However, except for a handful of standalone applications, most web applications need to share data and integrate with other services. They generally do so through a web API, which can be considered the server-side component of a web application framework. Note that it’s still possible (and sometimes desirable) to build an entire application with server-side rendering only, in which case Express may very well constitute the entire web application framework!

In addition to the features of Express explicitly mentioned in its own description, I would add two of my own:

Fast

As Express became the go-to web framework for Node.js development, it attracted a lot of attention from big companies that were running high-performance, high-traffic websites. This created pressure on the Express team to focus on performance, and Express now offers leading performance for high-traffic websites.

Unopinionated

One of the hallmarks of the JavaScript ecosystem is its size and diversity. While Express is often at the center of Node.js web development, there are hundreds (if not thousands) of community packages that go into an Express application. The Express team recognized this ecosystem diversity and responded by providing an extremely flexible middleware system that makes it easy to use the components of your choice in creating your application. Over the course of Express’s development, you can see it shedding “built-in” components in favor of configurable middleware.

I mentioned that Express is the “server-side part” of a web application framework…so we should probably consider the relationship between server-side and client-side applications.

# Server-Side and Client-Side Applications

A _server-side application_ is one where the pages in the application are rendered on the server (as HTML, CSS, images and other multimedia assets, and JavaScript) and sent to the client. A _client-side application_, by contrast, renders most of its own user interface from an initial application bundle that is sent only once. That is, once the browser receives the initial (generally very minimal) HTML, it uses JavaScript to modify the DOM dynamically and doesn’t need to rely on the server to display new pages (though raw data usually still comes from the server).

Prior to 1999, server-side applications were the standard. As a matter of fact, the term _web application_ was officially introduced that year. I think of the period roughly between 1999 and 2012 as the Web 2.0 era, during which the technologies and techniques that would eventually become client-side applications were being developed. By 2012, with smartphones firmly entrenched, it was common practice to send as little information as possible over the network, a practice that favored client-side applications.

Server-side applications are often called _server-side rendered_ (SSR), and client-side applications are usually called _single-page applications_ (SPAs). Client-side applications are fully realized in frameworks such as React, Angular, and Vue. I’ve always felt that “single-page” was a bit of a misnomer because—from the user’s perspective—there can indeed be many pages. The only difference is whether the page is shipped from the server or dynamically rendered in the client.

In reality, there are many blurred lines between server-side applications and client-side applications. Many client-side applications have two to three HTML bundles that can be sent to that client (for example, the public interface and the logged-in interface, or a regular interface and an admin interface). Furthermore, SPAs are often combined with SSR to increase first-page-load performance and aid in search engine optimization (SEO).

In general, if the server sends a small number of HTML files (generally one to three), and the user experiences a rich, multiview experience based on dynamic DOM manipulation, we consider that client-side rendering. The data (usually in the form of JSON) and multimedia assets for different views generally still come from the network.

Express, of course, doesn’t really care much if you’re making a server-side or client-side application; it is happy to fill either role. It makes no difference to Express if you are serving one HTML bundle or a hundred.

While SPAs have definitively “won” as the predominant web application architecture, this book begins with examples consistent with server-side applications. They are still relevant, and the conceptual difference between serving one HTML bundle or many is small. There is an SPA example in [Chapter 16](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch16.html#ch_single_page_applications).

# A Brief History of Express

The creator of Express, TJ Holowaychuk, describes Express as a web framework inspired by Sinatra, which is a web framework based on Ruby. It is no surprise that Express borrows from a framework built on Ruby: Ruby spawned a wealth of great approaches to web development, aimed at making web development faster, more efficient, and more maintainable.

As much as Express was inspired by Sinatra, it was also deeply intertwined with Connect, a “plug-in” library for Node. Connect coined the term _middleware_ to describe pluggable Node modules that can handle web requests to varying degrees. In 2014, in version 4.0, Express removed its dependency on Connect, but it still owes its concept of middleware to Connect.

###### Note

Express underwent a fairly substantial rewrite between 2.x and 3.0, then again between 3.x and 4.0. This book focuses on version 4.0.

# Node: A New Kind of Web Server

In a way, Node has a lot in common with other popular web servers, like Microsoft’s Internet Information Services (IIS) or Apache. What is more interesting, though, is how it differs, so let’s start there.

Much like Express, Node’s approach to web servers is very minimal. Unlike IIS or Apache, which a person can spend many years mastering, Node is easy to set up and configure. That is not to say that tuning Node servers for maximum performance in a production setting is a trivial matter; it’s just that the configuration options are simpler and more straightforward.

Another major difference between Node and more traditional web servers is that Node is single threaded. At first blush, this may seem like a step backward. As it turns out, it is a stroke of genius. Single threading vastly simplifies the business of writing web apps, and if you need the performance of a multithreaded app, you can simply spin up more instances of Node, and you will effectively have the performance benefits of multithreading. The astute reader is probably thinking this sounds like smoke and mirrors. After all, isn’t multithreading through server parallelism (as opposed to app parallelism) simply moving the complexity around, not eliminating it? Perhaps, but in my experience, it has moved the complexity to exactly where it should be. Furthermore, with the growing popularity of cloud computing and treating servers as generic commodities, this approach makes a lot more sense. IIS and Apache are powerful indeed, and they are designed to squeeze the very last drop of performance out of today’s powerful hardware. That comes at a cost, though: they require considerable expertise to set up and tune to achieve that performance.

In terms of the way apps are written, Node apps have more in common with PHP or Ruby apps than .NET or Java apps. While the JavaScript engine that Node uses (Google’s V8) does compile JavaScript to native machine code (much like C or C++), it does so transparently,[1](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch01.html#idm45053609101112) so from the user’s perspective, it behaves like a purely interpreted language. Not having a separate compile step reduces maintenance and deployment hassles: all you have to do is update a JavaScript file, and your changes will automatically be available.

Another compelling benefit of Node apps is that Node is incredibly platform independent. It’s not the first or only platform-independent server technology, but platform independence is really more of a spectrum than a binary proposition. For example, you can run .NET apps on a Linux server thanks to Mono, but it’s a painful endeavor thanks to spotty documentation and system incompatibilities. Likewise, you can run PHP apps on a Windows server, but it is not generally as easy to set up as it is on a Linux machine. Node, on the other hand, is a snap to set up on all the major operating systems (Windows, macOS, and Linux) and enables easy collaboration. Among website design teams, a mix of PCs and Macs is quite common. Certain platforms, like .NET, introduce challenges for frontend developers and designers, who often use Macs, which has a huge impact on collaboration and efficiency. The idea of being able to spin up a functioning server on any operating system in a matter of minutes (or even seconds!) is a dream come true.

# The Node Ecosystem

Node, of course, lies at the heart of the stack. It’s the software that enables JavaScript to run on the server, uncoupled from a browser, which in turn allows frameworks written in JavaScript (like Express) to be used. Another important component is the database, which will be covered in more depth in [Chapter 13](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch13.html#ch_persistence). All but the simplest of web apps will need a database, and there are databases that are more at home in the Node ecosystem than others.

It is unsurprising that database interfaces are available for all the major relational databases (MySQL, MariaDB, PostgreSQL, Oracle, SQL Server); it would be foolish to neglect those established behemoths. However, the advent of Node development has revitalized a new approach to database storage: the so-called NoSQL databases. It’s not always helpful to define something as what it’s _not_, so we’ll add that these NoSQL databases might be more properly called “document databases” or “key/value pair databases.” They provide a conceptually simpler approach to data storage. There are many, but MongoDB is one of the front-runners, and it’s the NoSQL database we will be using in this book.

Because building a functional website depends on multiple pieces of technology, acronyms have been spawned to describe the “stack” that a website is built on. For example, the combination of Linux, Apache, MySQL, and PHP is referred to as the _LAMP_ stack. Valeri Karpov, an engineer at MongoDB, coined the acronym _MEAN_: Mongo, Express, Angular, and Node. While it’s certainly catchy, it is limiting: there are so many choices for databases and application frameworks that “MEAN” doesn’t capture the diversity of the ecosystem (it also leaves out what I believe is an important component: rendering engines).

Coining an inclusive acronym is an interesting exercise. The indispensable component, of course, is Node. While there are other server-side JavaScript containers, Node is emerging as the dominant one. Express, also, is not the only web app framework available, though it is close to Node in its dominance. The two other components that are usually essential for web app development are a database server and a rendering engine (either a templating engine like Handlebars or an SPA framework like React). For these last two components, there aren’t as many clear front-runners, and this is where I believe it’s a disservice to be restrictive.

What ties all these technologies together is JavaScript, so in an effort to be inclusive, I will be referring to the _JavaScript stack_. For the purposes of this book, that means Node, Express, and MongoDB (there is also a relational database example in [Chapter 13](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch13.html#ch_persistence)).

# Licensing

When developing Node applications, you may find yourself having to pay more attention to licensing than you ever have before (I certainly have). One of the beauties of the Node ecosystem is the vast array of packages available to you. However, each of those packages carries its own licensing, and worse, each package may depend on other packages, meaning that understanding the licensing of the various parts of the app you’ve written can be tricky.

However, there is some good news. One of the most popular licenses for Node packages is the MIT license, which is painlessly permissive, allowing you to do _almost_ anything you want, including use the package in closed source software. However, you shouldn’t just assume every package you use is MIT licensed.

###### Tip

There are several packages available in npm that will try to figure out the licenses of each dependency in your project. Search npm for `nlf` or `license-report`.

While MIT is the most common license you will encounter, you may also see the following licenses:

GNU General Public License (GPL)

The GPL is a popular open source license that has been cleverly crafted to keep software free. That means if you use GPL-licensed code in your project, your project must _also_ be GPL licensed. Naturally, this means your project can’t be closed source.

Apache 2.0

This license, like MIT, allows you to use a different license for your project, including a closed source license. You must, however, include notice of components that use the Apache 2.0 license.

Berkeley Software Distribution (BSD)

Similar to Apache, this license allows you to use whatever license you wish for your project, as long as you include notice of the BSD-licensed components.

###### Note

Software is sometimes _dual licensed_ (licensed under two different licenses). A common reason for doing this is to allow the software to be used in both GPL projects and projects with more permissive licensing. (For a component to be used in GPL software, the component must be GPL licensed.) This is a licensing scheme I often employ with my own projects: dual licensing with GPL and MIT.

Lastly, if you find yourself writing your own packages, you should be a good citizen and pick a license for your package, and document it correctly. There is nothing more frustrating to a developer than using someone’s package and having to dig around in the source to determine the licensing or, worse, find that it isn’t licensed at all.

# Conclusion

I hope this chapter has given you some more insight into what Express is and how it fits into the larger Node and JavaScript ecosystem, as well some clarity on the relationship between server-side and client-side web applications.

If you’re still feeling confused about what Express actually _is_, don’t worry: sometimes it’s much easier to just start using something to understand what it is, and this book will get you started building web applications with Express. Before we start using Express, however, we’re going to take a tour of Node in the next chapter, which is important background information to understanding how Express works.

[1](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch01.html#idm45053609101112-marker) Often called _just in time_ (JIT) compilation.