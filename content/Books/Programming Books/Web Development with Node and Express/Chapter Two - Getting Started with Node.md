---
id: 01JAH06BSJ005PQSR1M45JQXHH
modified: 2024-10-18T19:45:42-04:00
title: Chapter Two - Getting Started with Node
tags:
  - express
  - javascript
  - nodejs
  - books
  - programming
---
# Chapter 2. Getting Started with Node

If you don’t have any experience with Node, this chapter is for you. Understanding Express and its usefulness requires a basic understanding of Node. If you already have experience building web apps with Node, feel free to skip this chapter. In this chapter, we will be building a very minimal web server with Node; in the next chapter, we will see how to do the same thing with Express.

# Getting Node

Getting Node installed on your system couldn’t be easier. The Node team has gone to great lengths to make sure the installation process is simple and straightforward on all major platforms.

Go to the [Node home page](http://nodejs.org/). Click the big green button that has a version number followed by “LTS (Recommended for Most Users).” LTS stands for _Long-Term Support_, and is somewhat more stable than the version labeled Current, which contains more recent features and performance improvements.

For Windows and macOS, an installer will be downloaded that walks you through the process. For Linux, you will probably be up and running more quickly if you [use a package manager](http://bit.ly/36UYMxI).

###### Caution

If you’re a Linux user and you do want to use a package manager, make sure you follow the instructions in the aforementioned web page. Many Linux distributions will install an extremely old version of Node if you don’t add the appropriate package repository.

You can also download [a standalone installer](https://nodejs.org/en/download), which can be helpful if you are distributing Node to your organization.

# Using the Terminal

I’m an unrepentant fan of the power and productivity of using a terminal (also called a _console_ or _command prompt_). Throughout this book, all examples will assume you’re using a terminal. If you’re not friends with your terminal, I highly recommend you spend some time familiarizing yourself with your terminal of choice. Many of the utilities in this book have corresponding GUI interfaces, so if you’re dead set against using a terminal, you have options, but you will have to find your own way.

If you’re on macOS or Linux, you have a wealth of venerable shells (the terminal command interpreter) to choose from. The most popular by far is bash, though zsh has its adherents. The main reason I gravitate toward bash (other than long familiarity) is ubiquity. Sit down in front of any Unix-based computer, and 99% of the time, the default shell will be bash.

If you’re a Windows user, things aren’t quite so rosy. Microsoft has never been particularly interested in providing a pleasant terminal experience, so you’ll have to do a little more work. Git helpfully includes a “Git bash” shell, which provides a Unix-like terminal experience (it has only a small subset of the normally available Unix command-line utilities, but it’s a useful subset). While Git bash provides you with a minimal bash shell, it’s still using the built-in Windows console application, which leads to an exercise in frustration (even simple functionality like resizing a console window, selecting text, cutting, and pasting is unintuitive and awkward). For this reason, I recommend installing a more sophisticated terminal such as [ConsoleZ](https://github.com/cbucher/console) or [ConEmu](https://conemu.github.io/). For Windows power users—especially for .NET developers or for hardcore Windows systems or network administrators—there is another option: Microsoft’s own PowerShell. PowerShell lives up to its name: people do remarkable things with it, and a skilled PowerShell user could give a Unix command-line guru a run for their money. However, if you move between macOS/Linux and Windows, I still recommend sticking with Git bash for the consistency it provides.

If you’re using Windows 10 or later, you can now install Ubuntu Linux directly on Windows! This is not dual-boot or virtualization but some great work on behalf of Microsoft’s open source team to bring the Linux experience to Windows. You can install Ubuntu on Windows through the [Microsoft App Store](http://bit.ly/2KcSfEI).

A final option for Windows users is virtualization. With the power and architecture of modern computers, the performance of virtual machines (VMs) is practically indistinguishable from actual machines. I’ve had great luck with Oracle’s free [VirtualBox](https://www.virtualbox.org/).

Finally, no matter what system you’re on, there are excellent cloud-based development environments, such as [Cloud9](https://aws.amazon.com/cloud9/) (now an AWS product). Cloud9 will spin up a new Node development environment that makes it extremely easy to get started quickly with Node.

Once you’ve settled on a shell that makes you happy, I recommend you spend some time getting to know the basics. There are many wonderful tutorials on the internet ([The Bash Guide](https://guide.bash.academy/) is a great place to start), and you’ll save yourself a lot of headaches later by learning a little now. At minimum, you should know how to navigate directories; copy, move, and delete files; and break out of a command-line program (usually Ctrl-C). If you want to become a terminal ninja, I encourage you to learn how to search for text in files, search for files and directories, chain commands together (the old “Unix philosophy”), and redirect output.

###### Caution

On many Unix-like systems, Ctrl-S has a special meaning: it will “freeze” the terminal (this was once used to pause output quickly scrolling past). Since this is such a common shortcut for Save, it’s easy to unthinkingly press, which leads to a confusing situation for most people (this happens to me more often than I care to admit). To unfreeze the terminal, simply hit Ctrl-Q. So if you’re ever confounded by a terminal that seems to have suddenly frozen, try pressing Ctrl-Q and see if that releases it.

# Editors

Few topics inspire such heated debate among programmers as the choice of editors, and for good reason: the editor is your primary tool. My editor of choice is vi (or an editor that has a vi mode).[1](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch02.html#idm45053605785656) vi isn’t for everyone (my coworkers constantly roll their eyes at me when I tell them how easy it would be to do what they’re doing in vi), but finding a powerful editor and learning to use it will significantly increase your productivity and, dare I say it, enjoyment. One of the reasons I particularly like vi (though hardly the most important reason) is that, like bash, it is ubiquitous. If you have access to a Unix system, vi is there for you. Most popular editors have a “vi mode” that allows you to use vi keyboard commands. Once you get used to it, it’s hard to imagine using anything else. vi is a hard road at first, but the payoff is worth it.

If, like me, you see the value in being familiar with an editor that’s available anywhere, your other option is Emacs. Emacs and I have never quite gotten on (and usually you’re either an Emacs person or a vi person), but I absolutely respect the power and flexibility that Emacs provides. If vi’s modal editing approach isn’t for you, I would encourage you to look into Emacs.

While knowing a console editor (like vi or Emacs) can come in incredibly handy, you may still want a more modern editor. A popular choice is [Visual Studio Code](https://code.visualstudio.com/) (not to be confused with Visual Studio without the “Code”). I can heartily endorse Visual Studio Code; it is a well-designed, fast, efficient editor that is perfectly suited for Node and JavaScript development. Another popular choice is [Atom](https://atom.io/), which is also popular in the JavaScript community. Both of these editors are available for free on Windows, macOS, and Linux (and both have vi modes!).

Now that we have a good tool to edit code, let’s turn our attention to npm, which will help us get packages that other people have written so we can take advantage of the large and active JavaScript community.

# npm

npm is the ubiquitous package manager for Node packages (and is how we’ll get and install Express). In the wry tradition of PHP, GNU, WINE, and others, _npm_ is not an acronym (which is why it isn’t capitalized); rather, it is a recursive abbreviation for “npm is not an acronym.”

Broadly speaking, a package manager’s two primary responsibilities are installing packages and managing dependencies. npm is a fast, capable, and painless package manager, which I feel is in large part responsible for the rapid growth and diversity of the Node ecosystem.

###### Note

There is a popular competing package manager called Yarn that uses the same package database that npm does; we’ll be using Yarn in [Chapter 16](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch16.html#ch_single_page_applications).

npm is installed when you install Node, so if you followed the steps listed earlier, you’ve already got it. So let’s get to work!

The primary command you’ll be using with npm (unsurprisingly) is `install`. For example, to install nodemon (a popular utility to automatically restart a Node program when you make changes to the source code), you would issue the following command (on the console):

npm install -g nodemon

The `-g` flag tells npm to install the package _globally_, meaning it’s available globally on the system. This distinction will become clearer when we cover the _package.json_ files. For now, the rule of thumb is that JavaScript utilities (like nodemon) will generally be installed globally, whereas packages that are specific to your web app or project will not.

###### Note

Unlike languages like Python—which underwent a major language change from 2.0 to 3.0, necessitating a way to easily switch between different environments—the Node platform is new enough that it is likely that you should always be running the latest version of Node. However, if you do find yourself needing to support multiple versions of Node, check out [nvm](https://github.com/creationix/nvm) or [n](https://github.com/tj/n), which allow you to switch environments. You can find out what version of Node is installed on your computer by typing `node --version`.

# A Simple Web Server with Node

If you’ve ever built a static HTML website before or are coming from a PHP or ASP background, you’re probably used to the idea of the web server (Apache or IIS, for example) serving your static files so that a browser can view them over the network. For example, if you create the file _about.html_ and put it in the proper directory, you can then navigate to _http://localhost/about.html_. Depending on your web server configuration, you might even be able to omit the _.html_, but the relationship between URL and filename is clear: the web server simply knows where the file is on the computer and serves it to the browser.

###### Note

_localhost_, as the name implies, refers to the computer you’re on. This is a common alias for the IPv4 loopback address 127.0.0.1 or the IPv6 loopback address ::1. You will often see 127.0.0.1 used instead, but I will be using _localhost_ in this book. If you’re using a remote computer (using SSH, for example), keep in mind that browsing to _localhost_ will not connect to that computer.

Node offers a different paradigm than that of a traditional web server: the app that you write _is_ the web server. Node simply provides the framework for you to build a web server.

“But I don’t want to write a web server,” you might be saying! It’s a natural response: you want to be writing an app, not a web server. However, Node makes the business of writing this web server a simple affair (just a few lines, even), and the control you gain over your application in return is more than worth it.

So let’s get to it. You’ve installed Node, you’ve made friends with the terminal, and now you’re ready to go.

## Hello World

I’ve always found it unfortunate that the canonical introductory programming example is the uninspired message “Hello world.” However, it seems almost sacrilegious at this point to fly in the face of such ponderous tradition, so we’ll start there and then move on to something more interesting.

In your favorite editor, create a file called _helloworld.js_ (_ch02/00-helloworld.js_ in the companion repo):

```
const
```

###### Note

Depending on when and where you learned JavaScript, you may be disconcerted by the lack of semicolons in this example. I used to be a die-hard semicolon promoter, and I grudgingly stopped using them as I did more React development, where it is conventional to omit them. After a while, the fog lifted from my eyes, and I wondered why I was ever so excited about semicolons! I’m now firmly on team “no-semicolon,” and the examples in this book will reflect that. It’s a personal choice, and you are welcome to use semicolons if you wish.

Make sure you are in the same directory as _helloworld.js_, and type `node helloworld.js`. Then open a browser and navigate to _http://localhost:3000_, and _voilà_! Your first web server. This particular one doesn’t serve HTML; rather, it just displays the message “Hello world!” in plain text to your browser. If you want, you can experiment with sending HTML instead: just change `text/plain` to `text/html` and change `'Hello world!'` to a string containing valid HTML. I didn’t demonstrate that, because I try to avoid writing HTML inside JavaScript for reasons that will be discussed in more detail in [Chapter 7](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch07.html#ch_templating).

## Event-Driven Programming

The core philosophy behind Node is that of _event-driven programming_. What that means for you, the programmer, is that you have to understand what events are available to you and how to respond to them. Many people are introduced to event-driven programming by implementing a user interface: the user clicks something, and you handle the _click event_. It’s a good metaphor, because it’s understood that the programmer has no control over when, or if, the user is going to click something, so event-driven programming is really quite intuitive. It can be a little harder to make the conceptual leap to responding to events on the server, but the principle is the same.

In the previous code example, the event is implicit: the event that’s being handled is an HTTP request. The `http.createServer` method takes a function as an argument; that function will be invoked every time an HTTP request is made. Our simple program just sets the content type to plain text and sends the string “Hello world!”

Once you start thinking in terms of event-driven programming, you start seeing events everywhere. One such event is when a user navigates from one page or area of your application to another. How your application responds to that navigation event is referred to as _routing_.

## Routing

Routing refers to the mechanism for serving the client the content it has asked for. For web-based client/server applications, the client specifies the desired content in the URL; specifically, the path and querystring (the parts of a URL will be discussed in more detail in [Chapter 6](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch06.html#ch_the_request_and_response_objects)).

###### Note

Server routing traditionally hinges on the path and the querystring, but there is other information available: headers, the domain, the IP address, and more. This allows servers to take into consideration, for example, the approximate physical location of the user or the preferred language of the user.

Let’s expand our “Hello world!” example to do something more interesting. Let’s serve a really minimal website consisting of a home page, an About page, and a Not Found page. For now, we’ll stick with our previous example and just serve plaintext instead of HTML (_ch02/01-helloworld.js_ in the companion repo):

```
const
```

If you run this, you’ll find you can now browse to the home page (_http://localhost:3000_) and the About page (_http://localhost:3000/about_). Any querystrings will be ignored (so _http://localhost:3000/?foo=bar_ will serve the home page), and any other URL (_http://localhost:3000/foo_) will serve the Not Found page.

## Serving Static Resources

Now that we’ve got some simple routing working, let’s serve some real HTML and a logo image. These are called _static_ resources because they generally don’t change (as opposed to, for example, a stock ticker: every time you reload the page, the stock prices may change).

###### Tip

Serving static resources with Node is suitable for development and small projects, but for larger projects, you will probably want to use a proxy server such as NGINX or a CDN to serve static resources. See [Chapter 17](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch17.html#ch_static_content) for more information.

If you’ve worked with Apache or IIS, you’re probably used to just creating an HTML file, navigating to it, and having it delivered to the browser automatically. Node doesn’t work like that: we’re going to have to do the work of opening the file, reading it, and then sending its contents along to the browser. So let’s create a directory in our project called _public_ (why we don’t call it _static_ will become evident in the next chapter). In that directory, we’ll create _home.html_, _about.html_, _404.html_, a subdirectory called _img_, and an image called _img/logo.png_. I’ll leave that up to you; if you’re reading this book, you probably know how to write an HTML file and find an image. In your HTML files, reference the logo thusly: `<img src="/img/logo.png" alt="logo">`.

Now modify _helloworld.js_ (_ch02/02-helloworld.js_ in the companion repo):

```
const
```

###### Note

In this example, we’re being pretty unimaginative with our routing. If you navigate to _http://localhost:3000/about_, the _public/about.html_ file is served. You could change the route to be anything you want, and change the file to be anything you want. For example, if you had a different About page for each day of the week, you could have files _public/about_mon.html_, _public/about_tue.html_, and so on, and provide logic in your routing to serve the appropriate page when the user navigates to _http://localhost:3000/about_.

Note we’ve created a helper function, `serveStaticFile`, that’s doing the bulk of the work. `fs.readFile` is an asynchronous method for reading files. There is a synchronous version of that function, `fs.readFileSync`, but the sooner you start thinking asynchronously, the better. The `fs.readFile` function uses a pattern called _callbacks_. You provide a function called a _callback function_, and when the work has been done, that callback function is invoked (“called back,” so to speak). In this case, `fs.readFile` reads the contents of the specified file and executes the callback function when the file has been read; if the file didn’t exist or there were permissions issues reading the file, the `err` variable is set, and the function returns an HTTP status code of 500 indicating a server error. If the file is read successfully, the file is sent to the client with the specified response code and content type. Response codes will be discussed in more detail in [Chapter 6](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch06.html#ch_the_request_and_response_objects).

###### Tip

`__dirname` will resolve to the directory the executing script resides in. So if your script resides in _/home/sites/app.js_, `__dirname` will resolve to _/home/sites_. It’s a good idea to use this handy global whenever possible. Failing to do so can cause hard-to-diagnose errors if you run your app from a different directory.

# Onward to Express

So far, Node probably doesn’t seem that impressive to you. We’ve basically replicated what Apache or IIS do for you automatically, but now you have some insight into how Node does things and how much control you have. We haven’t done anything particularly impressive, but you can see how we could use this as a jumping-off point to do more sophisticated things. If we continued down this road, writing more and more sophisticated Node applications, you might very well end up with something that resembles Express….

Fortunately, we don’t have to: Express already exists, and it saves you from implementing a lot of time-consuming infrastructure. So now that we’ve gotten a little Node experience under our belt, we’re ready to jump into learning Express.

[1](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch02.html#idm45053605785656-marker) These days, vi is essentially synonymous with vim (vi improved). On most systems, vi is aliased to vim, but I usually type `vim` to make sure I’m using vim.