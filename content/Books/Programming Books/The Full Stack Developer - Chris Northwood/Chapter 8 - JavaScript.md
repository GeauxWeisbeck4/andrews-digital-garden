---
id: 01JB58FGP72PH2RK6TD4R84QTM
title: Chapter 8 - JavaScript
modified: 2024-10-26T16:36:59-04:00
tags:
  - full-stack
  - books
---
# 8. JavaScript

Chris Northwood[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_8_Chapter.xhtml#Aff2) 

(1)

Manchester, UK

Famously designed in 10 days, JavaScript has quite a mixed reputation among software developers. However, if you’re doing anything on the web, you will need to have some JavaScript skills.

JavaScript was originally designed as a language for manipulating web pages using an API known as the Document Object Model (DOM). Originally created at Netscape, it was named JavaScript as an attempt to ride the wave of hype behind the increasingly popular Java language, a decision that has caused confusion for new developers for years, as the language has little in common with Java. The early days of the web didn’t help, with Microsoft developing its own variant, known simply as JScript which added new features, such as XMLHttpRequest (now part of the main language, and commonly abbreviated to XHR), which allowed developers to make requests to external servers to refresh the information on a web page. The European Computer Manufacturers Association (ECMA) finally decided to attempt to merge these competing implementations into a new standard, called ECMAScript. Therefore, what is called JavaScript nowadays is actually various implementations of ECMAScript, which is why you might hear terms like “ES6” or “ES2018” used in reference to JavaScript (previously it was versioned by number, but is now by year—ES2018 is the ninth edition of JavaScript). These refer to the different specifications published by ECMA, which different browsers are compliant with. In this chapter, I will reference the ES2018 version of the language. For developers who have worked with JavaScript before, ES6 introduced some significant language changes, such as the introduction of arrow function, with the yearly iterations beyond that introducing smaller, but still significant, changes.

JavaScript may have come from a rushed past, but as the language is constantly refined, some of the biggest thorns in the sides of developers (especially those who come from other languages) are disappearing. Some still remain, however, and I’ll call out those issues below.

For a long time, JavaScript on the browser was only used to add extra enhancements to a page, such as inline form validation, or simple animations. This was partly because the JavaScript engines in the browser simply weren’t fast enough to support complicated operations; although there were some early apps that did, they were highly tuned for performance and were therefore complicated to build. Google changed all this with the introduction of the V8 engine, which significantly sped up JavaScript and allowed for much more complex code to be introduced to the page. Later on, the Node project was launched, which allowed JavaScript to run on a server, along with a new standard library to support server-side requirements. In addition to enabling the same language on both the client and server sides, this also enabled code-sharing between the two, introducing a new style of app known as isomorphic—or universal—JavaScript, where code is agnostic as to where it’s run.

## Asynchronicity

This book isn’t the place to cover the huge range of events and APIs available in a browser, as they are rapidly evolving and there are many fantastic resources out there already. However, there are a few important things to understand. The main thing to understand about JavaScript is that it is single-threaded and relies on the concept of asynchronicity extensively. The downside is that, while JavaScript is executing, the runtime is “blocked.” In the browser, this means that the whole browser will not react to any interaction from the user while a JavaScript function is running. On the server, it means no other connections will be accepted or handled.

Many JavaScript applications are IO-bound, rather than CPU-bound—that is, most of this time is spent waiting for an external event to occur, rather than doing heavy number crunching. In these kinds of environments, this single threading approach makes a lot of sense, because Node does not need to actively wait for these external events to occur. It can just execute a function, then when that function ends, it can use callbacks and event listeners for various asynchronous activities. When the events those handlers were set up for occur, those functions can then execute. In between those two instances, any other callbacks or event listeners can execute in response to events that have either occurred while the original function was running, or that happen while there is no active method.

This makes life simpler, because concepts of thread safety do not need to be considered, which eliminates an entire class of errors and complexity. If you are handling a callback, you do not need to worry that suddenly another function may start in the middle of your context and leave some shared state in an inconsistent or unexpected mode.

For a more concrete example of what this means, take, for example, some theoretical code for making a web request:

function fetchAndLog(url) {

    const request = new XMLHttpRequest();

    request.open('GET', url, false);

    request.send();

    console.log(request.responseText);

}

The issue here is that it might take several seconds for the request to complete, and during that time the user can’t interact with the page—to them, it almost appears to have crashed. Even when the request is quicker, it can seem to make the page slow.

An asynchronous approach might look like this:

function fetchAndLog(url) {

    const request = new XMLHttpRequest();

    request.open('GET', url, false);

    request.onreadystatechange = () => {

        if (request.readyState === 4) {

            console.log(request.responseText);

        }

    }

    request.send();

}

While the user is waiting for the HTTP request to occur, they can continue to interact with the page as appropriate. This can cause complications with the user interface. If the user has performed an action that requires something asynchronous to occur, it may not always be obvious that their action has worked, so they click on it again. Updating the UI to include an intermittent “pending” or “in progress” state in response to an action is needed to avoid this.

### CALLBACKS, PROMISES, ASYNC & AWAIT

For complicated functions, nesting callbacks to such an extent can result in “callback hell,” where there’s a deep level of nested callbacks. Take the following example for downloading a video file, transcoding it using ffmpeg, and then deleting the original file.

function downloadAndTranscode(url, outputFilename, callback) {

    downloadFile(url, (err, downloadedFilename) => {

        if (err) { callback(err); }

        ffmpegTranscode(downloadedFilename, outputFilename, (err) => {

            if (err) { callback(err); }

            deleteFile(downloadedFilename, (err) => {

                if (err) {callback(err); }

                callback(null);

            });

        });

    });

}

Every time a new asynchronous operation occurs, a further level of nesting is added, which can make the code hard to follow. Promises are a technique that can avoid this by building a chain of individual promises that each respond to the one before it. In the example above, we could rewrite this using promises to instead look like so:

function downloadAndTranscode(url, outputFilename) {

    return downloadFile(url)

        .then(downloadedFilename =>

            ffmpegTranscode(downloadedFilename, outputFilename))

                .then(() => deleteFile(downloadedFilename))

        )

}

In the example above, assuming that the functions themselves also return promises, a chain is set up. A promise can be thought of as a placeholder for an asynchronous function that will either resolve to some value or reject if an error has occurred. The promise object itself does not actually change, but instead handlers can be added to it by calling .then() with a callback. The return value of .then() is then a new promise, which can then be chained further. Promises are a very rich subject, and the Mozilla Developer Network provides a more in-depth introduction to promises and how to use them. Promise.all() has particular advantage over the previous callback style of asynchronous handling, allowing multiple asynchronous actions to happen in parallel and then succeed or fail as one, with the results being made available to the following then() in the chain.

async and await are also language features introduced to JavaScript that work behind the scenes to handle the construction of promise chains and can further simplify them. Using async and await , the chain above could be further simplified to:

async function downloadAndTranscode(url, outputFilename) {

    let downloadedFilename = await downloadFile(url);

    await ffmpegTranscode(downloadedFilename, outputFilename);

    await deleteFile(downloadedFilename);

}

The main issue with asynchronous actions is that it is often hard to refactor an API that previously worked synchronously into an asynchronous one. This is sometimes necessary when the way the API works under the hood has changed to an asynchronous one. However, refactoring an asynchronous API to a synchronous one is much simpler, as the callback can simply be called synchronously. With this in mind, it can often be easier when designing an interface in JavaScript to make it behave asynchronously, if there’s any chance whatsoever that it will need to become asynchronous in the future.

This approach can create subtle bugs with regards to ordering and race conditions. Even if you can evaluate an asynchronous function immediately (for example, calling an error callback immediately on a method that makes an HTTP request if the parameters are invalid, whereas sometimes the callback is called asynchronously if it has to wait for a response), it can make sense to force it to always behave asynchronously, using mechanisms like setImmediate() or Node’s process.nextTick().

## JavaScript in the Browser

JavaScript was designed to run in a web browser, and for a long time, that was the only place it did run. But JavaScript was never a widely loved language, and for a long time, people have used alternate languages for the Web. This was partly because of perceived shortcomings in JavaScript, and a desire to standardize on one tech stack. This caused the rise of browser plugins like Flash (which was supposed to address the former) and Java (to address the latter), but these plugins have grown out of favor as the tension between closed source, proprietary plugins and the open nature of the Web intensified. JavaScript also evolved to address many of the issues that made it unappealing to become the powerful language it is today.

Another driving force for alternative languages was that many bits of functionality had to be implemented twice—once on the server (to validate form requests, for example), and again on the client (to provide a better user experience and faster responses). Some frameworks responded by automatically generating JavaScript for the page based on the server-side logic, but this created issues. The automatically generated code was inflexible and hard to debug, and it often performed poorly. With the rise of Node, the holy grail of writing DRY (“don’t repeat yourself”) code was in sight, as modules could be shared between the client and server. This has had varying levels of success, as the contexts of execution on a server and execution in a browser are very different and required careful engineering to get right.

An alternative approach was languages that compile to JavaScript. Google’s GMail service has its front end written in Java, with a compiler that outputs JavaScript. Other languages, such as TypeScript and CoffeeScript, have taken the same approach, and occupy significant niches, but most web development is still done in JavaScript itself. This is partly because these alternative approaches can make debugging hard, but also because mixing libraries between different languages and the additional build work needed can often add more complication than saved effort. A feature known as asm.js was created to ease the ability to compile to JavaScript, allowing a simple subset of JavaScript that these compilers can target that, in theory, offers a high level of performance. This original idea has evolved into a standard called WebAssembly, a kind of low-level assembly language for the Web, which can be high performance but lacks access to the DOM, so cannot completely replace JavaScript.

One notable exception where compilation has taken off is in the case of JavaScript being compiled into another form of JavaScript, a process known as transpilation. For a long time, JavaScript didn’t support modules or packages. Asynchronous Module Definitions (AMDs) were the first attempt to solve this, and ran natively in the browser simply by the addition of a library, but Node used an alternative method for module loading known as CommonJS. CommonJS wasn’t natively compatible with the browser, so tools were needed to package CommonJS modules into browser-compatible scripts (Browserify is the most popular tool used to do this). This meant there was no longer a direct mapping between code written in an IDE and code running in the browser. Even before this, minification tools were used to boost performance, applying optimizations to the code and providing a single downloadable file. As with other languages that compiled to JavaScript, this made debugging more difficult. Fortunately, a technique known as source maps was introduced that allowed in-browser debuggers to help map the minified or transformed JavaScript back to the original.

With this in place, then came the next step in JavaScript transpilation. Although many people were using the latest browsers and were, thanks to auto-updating, remaining up-to-date, there was still a considerable chunk of people using older browsers, and either couldn’t or wouldn’t update. For some individuals, this can be a result of using old and unsupported devices, but for organizations, policy often dictates a conservative approach to rolling out new software until it has been thoroughly tested, meaning browsers can lag behind. As the pace of evolution of the web increased, developers wanted to use these new features, but were constrained by older browsers. At first, a technique known as polyfilling was used. Polyfills are scripts that provide pure-JavaScript versions of new APIs that are available in JavaScript, and were effective in letting people use new JavaScript APIs in old versions of JavaScript. However, they can’t cover all the holes—for example, adding something like the Geolocation API to a browser that simply doesn’t support it is impossible. Another limitation of polyfills is that they can’t actually change the underlying language. As newer versions of JavaScript evolved, the syntax changed, introducing new ways of expressing classes and defining anonymous functions. The introduction of transpilers allowed for developers to use these new features in the code they wrote, but for them to be translated into a less readable, but backwards-compatible variant to allow for execution in older browsers. This can be thought of as a more advanced version of the autoprefixers used in CSS.

However, with these transpilers, an increasing number of technologies have arisen that are closer to the traditional compiler approach. Facebook’s React extends JavaScript to “JSX,” which allows you to specify the HTML structure of a React component using an HTML-like syntax. Carefully considered usage of such a technology can be beneficial—in the case of React, as you can only use JSX to write React components, then you aren’t introducing a new dependency beyond using React. However, extensive use of similar technologies for other purposes can lead to vulnerable situations, since you are then heavily dependent on a non-standard approach for vast portions of your application, rather than only in a carefully scoped area.

Before the introduction of polyfills and JavaScript, a different approach was taken for backwards compatibility. This was partly because of a lack of standardization between the dialects of JavaScript, which did not emerge until much later. Some of these differences were fundamental, such as Mozilla and Internet Explorer having very different models for dealing with JavaScript events. To cope with these, it was much more common to use a library to abstract away the differences, so that library became your interface to the browser. By far, the most common library used to do this was JQuery, which became ubiquitous across the web, present on the majority of web sites at its peak.

As the Web has developed, this practice has become problematic. In modern browsers, the layer of abstraction offered by JQuery slows things down considerably, and many features that JQuery offered are now offered directly by JavaScript itself. Additionally, JQuery works in a global namespace, which can cause tension when you’re trying to develop loosely coupled components. JQuery is still a useful tool and shouldn’t be discounted, but in simple cases, it is not uncommon to see modules that interact with the DOM directly, rather than developers relying on JQuery. Use JQuery when needed, but know that it is no longer necessary to default to it.

As people are building richer and richer JavaScript applications, the original design decision to only have one thread, and to use an asynchronous model to keep the UI responsive, has started to cause issues. Front-end developers often refer to this problem as “jank.” A site is janky when it isn’t smooth to respond (for example, slow animations, or unsmooth scrolling, often caused by a “scroll” event handler). Fortunately, new browsers support “web workers,” which are a way of bringing a thread-type structure to JavaScript. Web workers can be thought of more as background processes than as threads. They run in a more constrained environment, without access to the full set of browser APIs. Most importantly, the web worker does not have access to the DOM to alter the page directly. There is also no shared state, though messages can be passed back and forth between the main JavaScript context and the workers.

## Offline-First Development

The Web is an inherently connected medium. If you have no network connection, you can’t load web pages. Native applications have always had this advantage over web applications, as the code and content are held locally on a device. As increasingly complex applications are now being delivered over the Web, and with the rise of smartphones that may have patchy connectivity, a solution to this gap was required.

HTML5 introduced the concept of application caches, where manifest files determined which resources a browser should cache, which were “online only,” and whether any offline placeholder content should be used when the network is unavailable. This approach proved hard to work with and was deprecated from the standards. A much more flexible alternative has now sprung up, known as service workers. A service worker is a JavaScript application that runs in the background of the browser, without access to the DOM. It behaves like a proxy server, in that all requests for the page or domain it was registered for go through it, meaning that the service worker can decide how to handle caching for requests.

Service workers are inherently asynchronous. They receive events from the browser, and then call respondWith() on the event with promises that either resolve with a response to the request that event signifies, or reject if there’s an error. For example, the install event happens when a service worker is first loaded. In this case, it is common for a service worker to download all the static assets and add them to the browser cache.

Often the most important event, once a service worker is installed, is the fetch event. This is called when the browser wants to make a network request. It is up to you to then handle this appropriately. You could choose to return from the cache instantly if it exists, resulting in performance improvements but potentially presenting stale content to the user, or to instead make a network request to get fresh content, only falling back on the cache if the browser is offline.

Caching is a complicated area, with many different approaches depending on your exact goals. Caching in general is discussed elsewhere in this book, but specifically for browsers, a Cache API exists. The Cache API in browsers does not handle expiry of items in the cache, which means you will often need to write some code yourself (or use a framework) to handle this for you, in addition to implementing appropriate patterns such as “stale-while-revalidate” if desirable (see the Systems chapter for more information on caching techniques). You should also be careful about what you cache. A photo-sharing web site could quite easily fill up a user’s disk if everything is cached indefinitely. To avoid this, browsers limit how much can be cached for an individual site. If you cache too much, browsers will prune the cache of important elements and your site may not work appropriately offline.

Offline-first goes beyond just using service workers to handle caching. Much like mobile-first web applications are designed to work with the most constrained screen sizes and interaction style first, web applications that are offline-first are designed to work without a network connection as their primary mode (usually by synchronizing the local caches with a server and then only accessing those local caches), rather than having an offline-only mode added on at a later date. This can add complexity to some types of apps, but will ultimately result in better performance and a more robust experience for the user. In the case of content web sites, the complexity of being offline-first may not matter; having the entire archive of a site synchronized to a device would be overkill. There are always exceptions, however. A travel guide being used out and about on a mobile phone without an internet connection would be a good fit. A news web site would not.

For read-only web sites, offline-first can be quite simple, requiring periodic downloads of certain data. This data can then be stored using the IndexedDB API from the service worker and accessed in the client. The Background Sync API was designed for exactly this use case, but at the time of writing, it is an extremely new API without widespread support.

For web sites where a user is expected to contribute data, the complexity can increase significantly. For example, in a hospital, it could be useful for doctors to use tablets for issuing prescriptions. However, wi-fi coverage at a hospital may not be consistent, so although the medicine database may be synced locally, the actual prescription could be stored until the device has once again connected, and then issued, similar to the way e-mail applications work. This kind of application could be handled fairly straight-forwardly by the Background Sync API by behaving opposite to the sync of server-side content. When a sync occurs, the service worker would check for messages in a pending or outbox table in IndexedDB and then fire them, marking them as successful when completed. User experience is important here, as the doctor would need to know if they’ve successfully issued a prescription, or if it has failed.

Other use cases become even more complicated. Take, for example, collaborative document editing, where multiple people are editing a document and want offline access (perhaps to edit an important presentation on a train or plane). Every software developer who has worked on a team is familiar with merge conflicts, when two people have worked on the same code and their changes have clashed with each other. The appropriate way to deal with this depends on your use case. Sometimes, “most recent wins” is a simple way to solve this, especially when combined with versioning to allow for mistakes to be corrected. A technique dating from the 1980s but popularized more recently by Google Drive, known as operational transformation, is a much more complex but effective way of dealing with this sync, as it can deal with any asynchronous operation, not just offline ones.

Regardless of the complexity, if you are building an application that is expected to be robust on mobile devices where connectivity may not be guaranteed, offline-first will allow you to tackle that complexity head on, rather than deferring it to another time.

## Document Object Model

If you’re working in browser-side JavaScript, you will at some point have to interact with the Document Object Model. The DOM is basically a tree of JavaScript objects that is exposed on a global object called window that represents the HTML structure of the page (starting at window.document for the <html> tag) and other HTML APIs. You manipulate the page by altering this tree—for example, inserting a new node as a child of another one to create new content, or by grabbing a node on the tree and altering its properties (for example, innerText on a <p> tag). Each HTML element has a corresponding class in JavaScript that offers an API, as well as some common APIs that apply to all elements (such as querying to find a child element, either on document.body to query the whole page, or inside an element for more scoped queries).

Another important concept related to the DOM is events. Until fairly recently, there were fundamental differences between how events were handled in browsers, but fortunately nowadays modern browsers implement the standard. DOM events occur in response to user interactions, such as clicking on part of the page or scrolling through it. You can bind functions to events that happen on particular DOM elements by calling addEventListener() on the DOM element you want to listen to. An older approach is to assign a function to an attribute of the DOM element you want to react to—for example, target.onclick = myFunction—but this has the downside of only having one handler per event.

DOM events differ from other types of events that require passing a specific callback to an API, as there is no DOM element to listen to. For example, calling setInterval() or requestAnimationFrame() both accept a callback with the result of the operation. Another difference in DOM events is the concept of event capturing and bubbling. When a user clicks on a button, the event is first “captured” through all of the parent nodes until it reaches the target, and then “bubbles” up, so that the event listeners on the button, then on the parent node of the button, then on its parent, and so on, are called, all the way back to the root of the DOM tree. You can use event.target to see which DOM element actually triggered the event. For example, if a click handler is on a <p> that has a <span> inside it, perhaps the user actually clicked on the <span>. Most event handlers run in the bubbling phase, and it is rare to see capture phase handlers in actual code. You can use addEventListener to specify whether a handler runs in the capture or bubble phase. Event handlers can also prevent further bubbling or capturing by calling stopPropagation() on the event object the handler is called with, but this can have unexpected consequences. preventDefault() is a more useful alternative that serves a similar purpose, by stopping the browser from perform its default action while still letting the event propagate to other listeners.

The number of HTML elements, events, and their interfaces is changing rapidly, although the most common of these are by-and-large stable and form a core of the HTML standard. Even covering these in detail would require a whole book, and I recommend the Mozilla Developer Network and many other resources that offer an extensive and up-to-date reference for all of these.

## Server-Side JavaScript

Like JavaScript in the browser, server-side JavaScript is asynchronous, but the side-effects of this manifest themselves slightly differently. If the JavaScript in the browser is busy, the user can’t interact with the page, but if the JavaScript on the server is busy, then the server will not process any other connections at all—users will have to wait until that one operation has completed. Although this asynchronicity can at first seem hard to understand (especially for a back-end developer used to dealing with threads and other techniques for parallelizing work), it is in practice a much simpler, and safer, approach than alternatives in other languages such as threading, when the work needed to be done by the application is IO-bound (that is, waiting for responses from other systems such as disks, databases, or API servers) rather than CPU-bound. Many web applications follow this pattern.

The major downside to JavaScript’s single-threaded nature is that on a multi-CPU server, it is limited to using only one CPU , and if you want to utilize all the CPUs on a multi-core system, you will need to start up multiple processes of the app (fortunately, Node’s cluster module allows you to share a port between all of these processes). This does mean that anything kept in-memory will not necessarily be available in response to future requests from a user, as a different process on the same box may serve that request. For storing any information that may need to persist beyond multiple requests, an external caching server such as Memcached or Redis is used. Using this approach is good practice, because if your application becomes popular, you may want to scale it onto multiple servers behind a load balancer to handle all of the requests. With an external caching server in place, it is as easy to scale to multiple boxes as it is to scale to different processes. This approach is called horizontal scaling, and is discussed further in the Systems chapter.

The biggest difference between JavaScript in the browser and JavaScript on the server is the DOM. There simply isn’t one on the server (nor would it make sense for there to be one!). Instead, Node has a completely separate standard library that provides functions that make sense on the server, but not on the browser, such as opening raw TCP sockets and accessing files. The language and syntax are otherwise the same, but for front-end developers new to server-side coding, it is sometimes surprising to realize that things such as a global window object actually aren’t part of the JavaScript language, but just the DOM.

Of course, you can still render HTML, but this is done using templating techniques (as discussed in the Front End chapter) rather than by building a DOM, or by using a virtual DOM library that behaves like a DOM but renders out a string of HTML. The libraries and frameworks used by JavaScript are in a high level of flux that almost certainly means anything you read here will be out of date by the time this book is printed. However, server-side JavaScript seems to have developed a culture of small, loosely coupled libraries, as opposed to other server-side languages where there are much larger all-encompassing frameworks (such as Java’s Spring or PHP’s Symfony). It is not uncommon to have a large number of dependencies on your application. This can make a quick-start slightly harder, since you have to wire together a number of things before you start, but also gives you a great deal of flexibility to build exactly what you need.

## JavaScript Modules

For a long time, JavaScript didn’t employ the concept of modules—simply different files that all acted in the same way in the same environment. JavaScript doesn’t have a standard library like other languages—all functions are available at all times—and the DOM was a single global object called window that new APIs kept getting added to over browser releases. If you wanted to bring in other functions that weren’t built into the language, you would normally add another <script> tag to your HTML before your code was loaded, and then those libraries would at best add something else to the window object (like JQuery), and at worst leak a bunch of internal functions everywhere, and you would just have to hope they didn’t clash with anything else.

The very first solution to that was to take advantage of JavaScript’s scoping rules to only expose what you wanted to leak. One way to do this is a mechanism known as an immediately-invoked function expression, or IIFE. When something is defined inside of a function, it only remains visible inside of that function, so to avoid putting everything onto window, modules were instead wrapped in a function that then became immediately invoked:

(function() {

        function somethingPrivate() { ... }

        function externallyUsableFunction() { ... }

        window.MyLibrary = externallyUsableFunction;

})();

This defines a function and immediately calls it, which means that when a file containing it is dropped onto the page, it gets run straight away and window.MyLibrary is made available, but none of the private functions are available.

However, there are flaws here. What happens if two libraries accidentally pick the same name, or you want two different versions of the same library? You also have to hope that if you make a distributable library that has other dependencies, any HTML page that includes your code correctly adds the other <script> tags before you are called. This relies on users following some documentation, as opposed to the library declaring its own dependencies in code.

Following this, a new mechanism known as asynchronous module definitions (AMDs) was used. This was an agreed syntax for defining a module and then importing it into another module.

define('MyLibrary', ['dependency-1', 'dependency-2'], function(dep1, dep2) {

        function MyLibrary() { ... }

        ...

        return MyLibrary;

})

require(['MyLibrary'], function(MyLibrary) {

        MyLibrary();

});

The first example above defines a library called MyLibrary, which exports a single function (called MyLibrary) and has access to two dependencies. This can be thought of as a step up from IIFE; functions are still used to ensure that the scope can be controlled, but instead of attaching things to the window object directly, they are returned from the function and stored until the library itself is depended upon. The definition also allows you to add the names of any dependencies you need, which are given as arguments to your defining function, so they are made accessible only to the modules that explicitly ask for them.

What the function actually returns could be anything. It could be a function, or it could be an object with many things on them, or perhaps even a simple string (if being used to define concepts like config settings or translations).

A library is then needed to provide a way of managing these dependencies, the most popular of which is called RequireJS. When you specify a dependency, RequireJS will then load it from the network as needed, which can reduce pre-load time, and is why these module definitions are asynchronous. A common optimization was to bundle all the dependencies together, so only one file needed to be downloaded at runtime.

NodeJS introduced another method of introducing modules, known as a CommonJS module. Instead of requiring IIFE, NodeJS by default ran each file in its own environment to avoid polluting the global namespace. A file therefore becomes a module by default, and to make something available to other modules, you added module.exports = MyLibrary to the end. Another module could then important that using syntax similar to const MyLibrary = require('./my-library.js'), as a path to the file that defines it (or if a non-relative path, like a path relative to a special folder called node_modules where all your dependencies live, often managed by a dependency management tool). This require() function, unlike the one in AMDs, is synchronous, therefore returns immediately, rather than requiring a callback. This is fine on a server where all files are local (although it can cause blocking), but gives you reduced flexibility on the desktop, as all possible dependencies must now be delivered as one bundle in advance, rather than loading in part later (there are now some tools that try to work around this restriction).

In an attempt to bridge the gap between AMDs and CommonJS modules, another approach known as a UMD (Universal Module Definition) is also used, although most often as the output of a build tool for a library, which is compatible with both.

The final type of module you will come across is the ES6 module, which was introduced in the JavaScript ES6 spec. An ES6 module is similar to a CommonJS module, except the syntax for importing and exporting modules is now language keywords, rather than special variables and functions.

In ES6 modules, you can have a default export defined like this:

export default function() { ... }

And then you can import this in another module:

import MyLibrary from "./my-library.js"

This makes the function available in the second module as MyLibrary.

ES6 modules also add the ability to export multiple things from a module, rather than a single thing, as with CommonJS. In this case, we can expand the first module as follows:

export default function() { ... }

export function utility() { ... }

export let configurationKey = '...';

and now in the second module:

import MyLibrary, { configurationKey } from "./my-library.js"

which makes MyLibrary and the configurationKey string available, but not the utility function. We can also avoid the default import completely:

import { utility } from "./my-library.js"

If the name of an export clashes, then it is possible to give it a different name when you import it:

import MyLibrary as AnotherLibrary from "./my-library.js"

The main drawback of ES6 modules is that, because they are a feature of the language, the module name must be a string literal. The following example is not allowed:

let libraryName = "./my-library.js";

import MyLibrary as AnotherLibrary from libraryName

This is because all imports in ES6 modules are done before the code executes, unlike the other module loading approaches, where the module loading happens at runtime. This is because loading a module might require downloading code in a browser, so all the referenced modules are downloaded at once, rather than having to be preloaded.

## Structuring Your JavaScript

Before the mainstream adoption of JavaScript modules, it was common to have a small number of files containing large segments of functionality, or sometimes JavaScript inline in the HTML. This worked when JavaScript was mostly used for small functionality improvements, but modern JavaScript applications tend to be larger, and using a single file for your entire application is hard to manage. Nowadays, using modules to structure your application is normal.

Knowing how to structure your modules can be quite hard. There are two leading methods for dealing with this. One is to have folder structures for each type of component—for example, grouping any UI modules in one place, and modules for managing state and business logic in another, perhaps with further separations based on models, views and controllers for MVC apps, or actions, reducers and state for Redux applications. This method has been criticized for being an arbitrary separation that requires looking in lots of different places when changing a single feature. Another option is to have a directory per feature or functional component with all the different parts of that feature (models, views and controllers, or similar) in that folder. One issue with this method is that often models and other common concepts will be shared across many features.

As with many things, balance is key. It is widely considered good practice to have the components relating to your views be free of any non-view logic. Taking state out of a view component also makes it easier to reuse it, potentially across other modules. In React with Redux, it’s also useful to keep a separation between the two types of view components: the component itself, and the component bound to the state.

In this case, all your views can be grouped together, perhaps with subgroupings for complex view components that might themselves have subcomponents, and business logic areas grouped based on functional domains. This means a view might take from many different areas of the state to render a UI component, but for the state, all the logic for manipulating that part of the state are kept together .

## JavaScript Types

For many developers who are experienced with languages such as Java or C#, JavaScript’s approach to data types can be surprising. JavaScript is known as a weakly typed language, in that it does have types, but these are implicit, and if something is in the wrong type, the JavaScript run time will try to implicitly cast something to the expected type. This behavior can be surprising, and can throw off a new JavaScript developer, and is the butt of many jokes about JavaScript.

For example: 1 + "2" will result in "12" as + is overloaded to mean string concatenation as well as addition, and the integer 1 will be coerced to a string to make the types match. +[] will end up being 0, as the + operator attempts to make something positive, but a list has no sensible value and ends up being 0.

The biggest issue this causes arises when checking for equality. When using == (and !=), JavaScript will try to coerce both sides of the equality operator to match. For example, "1" == 1 will evaluate as true, which is normally unexpected and undesirable. When using the identity operator, this type coercion does not happen, so both the types and the value of both sides must match. This can result in an idiosyncratic look to JavaScript code, where the identity operator of === (and its counterpart !==) appear frequently.

Fortunately, with the correct use of equality and identity checks, you can avoid hitting most of the pitfalls of this loose typing. Despite the fears of developers who are used to working in strongly typed languages like Java, it is rare to uncover bugs that result from this automatic type coercion. That said, you should also aim to avoid writing code that uses this loose typing, as that can introduce subtle bugs. It is sensible to call .toString() when concatenating strings when there is any doubt over the type of a variable, or run input through parseInt to ensure it is indeed an integer. The exception to this is often in if statements. As null, undefined, or empty strings become false when cast to a Boolean (these values are referred to as “falsey”), it can be a quick shortcut to write an if check such as if (!input) { return; } at the start of a function. Conversely, a number of values evaluate as true when considered as a Boolean, and these are called “truthy.” You may see syntax such as !!input used, which negates a value twice and, as a result, converts a truthy or falsey variable into a literal true or false.

JavaScript, as of ECMAScript 6, has seven data types. Six of these are considered to be primitives, and they are Boolean, null, undefined, number, string, and symbol.

Booleans are true or false, and they behave as you would expect. There is only one value that has the type null, and that is null. Undefined is similar, having only one value: undefined. The difference between null and undefined can be confusing, but the main difference is that undefined is used to represent a variable that has never had a value assigned to it, whereas null is used to indicate that something has been deliberately given no value. Null also behaves confusingly when used with the typeof operator. Despite null being a type, typeof null === 'object' is true. This is a result of a bug in the initial implementation of JavaScript that was left too long, and now cannot be fixed without breaking legacy code.

Also, unlike other languages, JavaScript only has one type for numerical values: number. This can simplify code, although if you are used to languages that do separate the two types, you must remember that the way elements such as integer division behave will differ from the way you are used to—it will become a float.

Strings should also be a familiar concept. In JavaScript, strings are immutable, meaning any operation you perform on a string does not change the original string. For example:

        const url = 'https://www.example.com';

        console.log(url.slice(0, 8));

This will print out the first eight characters of the variable url, but does not actually modify it. Instead, a new string is returned.

In implementation terms, JavaScript strings consist of UTF-16 characters (known as code points), which can act strangely when working with Unicode characters. In Unicode, some characters have IDs that are too long to fit into the space of UTF-16, so two UTF-16 characters are used, known as surrogate pairs, to represent a single character. This can manifest itself as strings appearing longer than they are. "![../images/471976_1_En_8_Chapter/471976_1_En_8_Figa_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_8_Chapter/471976_1_En_8_Figa_HTML.jpg)".length would return 2, rather than 1 as expected, as emoji exist in the area of Unicode known as the “astral plane,” so require two UTF-16 characters. Other operations also require caution. For example, if you wanted to reverse a string, a naive approach would be to iterate over it backwards, but this would result in an invalid string, as the individual UTF-16 code points would be inverted, in this case referencing invalid Unicode characters. There are libraries that can help you work with these kinds of strings, but further detail is beyond the scope of this introduction.

The final primitive is the latest addition to JavaScript. Symbols were added in ES6 as a way of uniquely identifying keys on objects. The main difference between strings and symbols is that two symbols are not identical even if they have the same value. Although "example" === "example" evaluates to true, Symbol("example") !== Symbol("example") requires that the comparison is a not-equals in order to be true, unless the symbols on both sides are the same instances of the objects.

Objects are the workhorses of JavaScript, and can do many different things. Objects essentially map keys to values, behaving like maps or dictionaries in other languages. Functions are also objects, which means that objects can be executed. Objects are also used to implement other higher-level types, such as linked lists (known as arrays in JavaScript) or sets. A JavaScript array is an object with numerical keys and other helper methods.

Different objects always have different values, even if their contents are the same. For example { foo: 'bar' } !== { foo: 'bar' } will evaluate to true, as in this case, the two objects that were created are different instances of the object. As arrays are also objects, this means that [1, 2] !== [1, 2] too is true. JavaScript does not provide a built-in way to check for equality in these cases, but there are simple approaches to implementing these kind of equality checks. For arrays (or objects where the order of keys is guaranteed), using JSON.stringify() to translate something into a JSON string and then doing a string equality check is a quick way to check this. For more complicated checks, such as deeply nested objects, there are many utility libraries available.

### JAVASCRIPT OBJECT NOTATION (JSON)

When data is transferred between systems, it often needs to be converted into a string, or binary file, to allow it to be transferred across a network. This process is known as serialization, and often this can be used to serialize arbitrary objects. For example, in Python, this is known as pickling. JSON is JavaScript’s equivalent, although it only supports serializing fundamental data types, not classes or similar, unlike other languages.

JSON was proposed in the mid-2000s by Douglas Crockford as a lighter-weight alternative to the XML-based serialization format that JavaScript originally used. JSON is a limited set of the syntax JavaScript uses to declare Boolean, object, string, number, and array literals. For example, it only supports double-quotes (") for strings, and does not support trailing commas in objects or array items. This was intended to make the language simpler and easier to use—to parse, you could use JavaScript’s eval function to run it as JavaScript, and the result would be the parsed object. However, as eval allows running arbitrary JavaScript, this opened up a security hole if there was any chance the JavaScript was malformed or untrusted, so later versions of the language introduced JSON.parse() and JSON.stringify() as methods to convert JavaScript objects to and from JSON strings.

As a result of its relative simplicity and support for types that are also fundamental in other languages, JSON has become a common serialisation format for other languages and systems too, and there are many libraries for other languages to serialize and parse JSON—even those where neither server nor client is JavaScript.

As a result of JSON’s lack of support for more complex types, both server and client must agree on how to convert anything more complex into JSON and then back into the more complex type, so JSON by itself cannot completely self-describe data. It must be paired with some additional type information. This is a source of criticism for JSON, especially from the XML community, where XML schemas can be used to more strictly define the form and meaning of the encoded data. There are tools available for JSON that help define the meaning and check the validity of serialized JSON—JSON Schema is one of the more commonly used options.

The biggest advantage of JavaScript’s weak typing is that duck typing can be used when passing objects around (i.e., if an object behaves in an expected way, even if it isn’t defined in terms of the expected type, such as if it had the same keys or similar). For the most part though, weak typing is often seen as a negative to be avoided. For a long time, other languages have existed that compile to JavaScript, and as mentioned earlier in this chapter, there are JavaScript transpilers that take new JavaScript and make it backwards-compatible with old browsers. To this end, Microsoft has developed a language called TypeScript that looks and feels like JavaScript, but includes type annotations that are then checked in the transpilation process. An alternative approach has been taken by Facebook with their Flow type checker. Instead of using a new language that’s derived from JavaScript, Flow allows you to add annotations to existing JavaScript code, which are then stripped out during the transpilation process. A separate type checker runs over the code to check that all the types are as expected.

        function fetchTransactionValue(id: string): Promise<number> {

            ...

        }

The preceding code is an example of a type annotation in Flow, indicating that the function takes a single parameter of type string and then returns a promise, which resolves to a number (often, the return type is inferred, so does not need to be explicitly specified).

TypeScript and Flow are popular methods for helping development teams write JavaScript and avoid bugs, and at the time of writing, it appears that adoption of these tools will only increase.

## Object-Oriented Programming

Object-oriented programming has become the predominant methodology in software engineering over the last few decades, replacing structural programming. Although JavaScript supports object-orientation, it doesn’t do so in the way most people are used to. Most programmers are trained on object-orientation in the classical sense (here, classical means “pertaining to classes,” rather than “traditional”), where objects are instantiations of classes, and classes can have hierarchy. Until ES6, JavaScript differed in that JavaScript classes were prototypical. Rather than an object being an instance of a class, it’s an instance of a prototype. As such, it is missing many features that a Java or PHP developer may take for granted, such as interfaces, inheritance, or even method visibility and defined fields.

ES6 has changed all that by introducing classes into JavaScript, which are much more familiar to those coming from other languages. Under the hood, objects are still prototype-based, but you can now treat them the way you would otherwise treat classes. Libraries also exist for older variants of JavaScript to make prototypes feel more like classes, but they never became a core part of the language. Other techniques give support for private fields, such as keeping the private fields in a different scope and enforcing access through getters/setters, but these often have a performance or memory overhead.

JavaScript isn’t the only language to not have private methods/fields. Python also doesn’t, taking a “gentleman’s agreement” approach. “Private” methods and fields are prefixed with an underscore and not documented, and although this does not prevent them from being accessed directly, they indicate to a developer that they are doing something wrong and should not be relied upon. For many JavaScript applications, this approach is also sufficient, even if you’re distributing a library for wider consumption.

Prototypical objects in JavaScript start by defining a constructor as a normal function, and then adding functions to an object of that function called prototype. It looks a bit like this:

function ShoppingBasket() {

    this._items = [];

}

ShoppingBasket.prototype.addItem = function(item) {

    this._items.push(item);

}

Running new ShoppingBasket() will create a new object from the prototype, set this to be that object, and then run the constructor.

ES6 classes simplify this syntax to make it look more familiar to developers coming from other languages, meaning you could instead specify the above in a more familiar sense:

class ShoppingBasket {

    constructor() {

        this._items = [];

    }

    addItem(item) {

        this._items.push(item);

    }

}

Now, recall that in our first example, ShoppingBasket is just a function. That means that if we call ShoppingBasket(), forgetting to add new at the start, then the code will execute, but will do the wrong thing. A new object will not be created, so this will be the window object, and nothing will be returned from it. JavaScript has a mode called strict mode to help prevent this from occurring. Strict mode can be enabled by adding "use strict"; to the top of a file or function, and is designed to make common mistakes actually throw errors, rather than be silently ignored. In this case, it stops unknown variables from being set on this unless they are actual objects.

To avoid this, there is a pattern that prefers not to use new at all, but instead use functions for everything. For example, the previous code could be expressed as:

function ShoppingBasket() {

    const items = [];

    return {

        addItems(item) {

            items.push(item);

        },

    };

}

This code is a function that returns an object with functions. It also makes items private by not actually putting it onto an object that is returned. This has the downside of increasing memory, as each method on the object is a new instance rather than a shared prototype. However, for modern JavaScript, ES6 classes are far and away the preferred way of expressing classes, with the other approaches common only in older code. There are also various libraries that can help create different types of JS classes, but these mostly predate ES6 classes.

One of the biggest benefits of ES6 classes is that they make class inheritance behave as many would expect it to. Take for example:

class Animal {

    ...

}

class Dog extends Animal {

    constructor() {

        super();

        this.kennel = null;

    }

}

In this, the class Dog extends Animal, so when new Dog() is called, it has both the methods of Dog and the methods of Animal (assuming that Dog has not overridden them). The keyword super is also made available, and can call methods on the parent (or further away, if the inheritance chain is several layers deep). When using ES6 classes, this hides a lot of the complexity required to specify prototypes directly (this is why there were several helpers in JavaScript to do inheritance prior to ES6).

The best way to understand how inheritance works in JavaScript is to know about prototype chains. An instance of a JavaScript class is an object that has a prototype. However, this prototype is itself just an object, so it can itself have a prototype. This is called the prototype chain, and when you access a JavaScript object, the prototype chain is followed until the first thing with the name you’re asking for is found. Ultimately, the last object you define in a chain will have a prototype of JavaScript’s built-in Object, which itself has a prototype of null, indicating the end of the chain. This shows that there’s never actually a thing called a class in JavaScript—just objects and prototypes—but this implementation detail is mostly irrelevant for most common day-to-day uses of JavaScript.

A final word of warning regarding objects in JavaScript, involving the keyword this. For developers coming from almost any other object-oriented language, this always refers to the instance of the object that the method belongs to. In JavaScript, it refers to the context the function is operating in. For example, when a function is added as a click event handler, when it is executed, this will be the element that fired the event, as that is the context the callback is executed in. JavaScript doesn’t differentiate between methods and functions, so passing a method to a click callback will divorce it from the object to which it otherwise belongs.

A common way of getting around this was to create a variable called that or _this or similar, and create a function that had that in scope (this is called a closure—more on that in the Functional Programming section), which gave it access to the object. In ES6, there are two alternative mechanisms in use: using .bind(this), which creates a new function with the this variable correctly referencing this (the same “this” that is passed in as an argument to bind), or using the arrow shorthand for defining a function (_() => { ... }_), which does that automatically.

## Functional Programming

Functional programming isn’t new —it predates the rise of object-oriented programming—but has seen a resurgence, and some of JavaScript’s early design decisions have made it suitable for applying these functional programming techniques, even though it does not enforce pure functional programming.

One advantage of JavaScript in this context is that it allows you to switch between class-based, procedural and functional styles with ease, and even use them in combination with one another. Some consider pure functional programming unsuitable for certain kinds of tasks, and certainly for many developers it requires a change in thinking about how to structure programs. This has held back the rise of functional programming for a long time.

In essence, functional programming has risen from the world of mathematics. It can sometimes be intimidating to a newcomer, as many of the terms (functor, monad, etc.) are very unfamiliar, and the concepts are often not easily described without relying on a base knowledge of category theory. But you can use functional programming techniques without developing any deep knowledge. At the core of functional programming is the idea of a function. A function is something that, for a given input, always returns the same output. Contrast this to an object-oriented approach, such as this date object:

class Date {

  ...

  addDays(daysToAdd) {

    this.timestamp += daysToAdd * 24 * 60 * 60;

  }

}

if we run addDays multiple times, the result we get is different each time. We could write a similar function that performs the same each time, but the function stores no state. It returns a new result, and doesn’t change the original one.

function addDays(date, daysToAdd) {

  return {

    timestamp: date.timestamp + (daysToAdd * 24 * 60 * 60),

    timezone: date.timezone,

  };

}

This is very powerful, as it eliminates a whole series of bugs, and it also allows us to chain together methods in new and interesting ways. Of course, it is possible to code a method this way too, such that it returns a new instance rather than modifying this one.

If you have worked extensively with JavaScript, there’s a good chance you’re using functional techniques already, without knowing it. Prior to .bind becoming a feature of the language, you may have seen this common technique:

Widget.prototype.setupEventListeners = function() {

    var that = this;

    foo.addEventListener('click', function(ev) {

        that.handleClick(ev);

    });

}

This employs two techniques that are core to functional programming: closures and functions as first-class objects.

A closure is simply a function that has access to the scope of where it is defined. In the setupEventListeners function , we declare a variable that, and the click callback handler. Even though the callback handler is a different function, and will potentially run long after setupEventListeners() has finished executing, it still can use that, as it has visibility to the scope (and also any nested scopes where it was declared).

### WHAT IS SCOPE?

Scope determines which functions can “see” a particular variable or function. Take the example:

function getCircleArea(radius) {

    const PI = 3.1415926535;

    return PI * radius * radius;

}

(in reality you would use Math.PI)

The variable PI is in scope within this function, but no other functions or areas of code can access it. Because JavaScript allows us to nest functions and blocks, scopes can also be nested within each other to create different levels of scope. Many module-loading frameworks will also create a scope at a file level, so you can’t arbitrarily use a variable in another file unless it’s declared as an export, and you import it. Other languages also employ the idea of “global” scope, where something is automatically available without having to be imported. JavaScript does not quite use the same concept, but instead has a global object (window in browser-based environments) that you can add things to.

Prior to ES6, the only way to declare variables was with the keywords var or function (for functions), but these were subject to something known as variable hoisting. In variable hoisting, all variables are defined before any code runs. For example, in strict mode, you would expect this to fail, as a is declared when it is set:

a = 6;

var a = 5;

console.log(a);

Because of variable hoisting of var a, when a = 6 runs, var has been declared. a = 5 overrides the value, as the initial values are not hoisted, just the definition. This can be useful for recursive functions; for example:

function recurseDeep(items) {

    return [].concat(recurseWide(items[0]), recurseWide(items.splice(1));

}

function recurseWide(items) {

    if (items[0].length > 1) {

        return recurseDeep(items);

    } else {

         return [].concat([items[0][0]], recurseWide(items.splice(1));

    }

}

You want recurseWide to be in scope for recurseDeep, despite it having been declared before.

ES6 has introduced two new keywords that can help protect you from errors that can be accidentally introduced by var. These are let and const, and they are what is known as block-scoped. Instead of automatically being declared at the top of the function that contains the definition, they are declared at the point they are actually used, and are only in scope within the block (set of braces) where that definition occurs.

let is used for variables that can be reassigned, whereas const is for variables that are not reassigned. const is not necessarily constant; for example, a list or object defined as const can have new things added to or removed from it, because it is always pointing to the same list. The best practice is to always use const, unless you need to reassign, in which case you should instead use let.

A slightly contrived example of using let and const is:

function calculate(a, b, callback) {

  let wrappedCallback;

  if (callback === null) {

    wrappedCallback = () => {

      console.log('done');

    };

  } else {

    wrappedCallback = (result) => {

      callback(result);

      console.log('done');

    };

  }

  if (a > b) {

    const result = a + b;

    wrappedCallback(result);

  }

  wrappedCallback(null);

}

In this example (which calls back the result of adding a and b if a is greater than b, and then null to indicate the calculation is finished), wrappedCallback must be let because the value is assigned to it after the fact, whereas result is const because it is never reassigned. In the final line of the function, we would not be able to access result (it would give an error as an undeclared variable) because the scope of result is limited to the body of the if function where it is declared.

Functions as first-class objects simply means that functions are objects that can be passed around like any other object, such as a string or a number. One important concept this enables is that functions can be passed to other functions as callbacks. Contrast this to languages like Java and PHP, where a technique known as reflection is needed in order to access these functions, which gives a large amount of overhead for passing around things like callbacks.

This allows JavaScript to have many asynchronous features, compared to languages like PHP, which must wait for a function to execute (known as blocking) before continuing on to the next line of code, or Java, which uses threads to avoid blocking, although an individual thread can still block.

There are several techniques that functional programming makes use of that are not common in predominantly object-oriented approaches. One such technique is that of a partial function. A partial function is a version of another function that has some of its arguments “pre-filled.” Take, for example, the following snippet:

function foo(a, b, c, d) { ... }

function makePartialFoo(a, b) {

    return function(c, d) { foo(a, b, c, d); };

}

This makePartialFoo is given the arguments a and b and returns a function that only needs c and d passed to run. Every time that new function is called, a and b are already supplied. This can be useful if a function takes some dependencies (through dependency injection) or other state into its first few arguments and then uses those as common arguments for a number of invocations.

bind() allows us to make partial functions much easier. We could instead make a partial version of the function foo above by calling: foo.bind(this, a, b) instead of needing to have a function to make the partial of foo (which can get unwieldy if you want multiple versions of foo). In pure functional languages, all functions can be made partial just by calling it with a subset of its arguments, a process known as currying.

In functional programming, functions should not hold some state. Instead, state is represented in a data structure that is passed to or returned from the function. Similarly, a pure function should not manipulate what it has been given. For example, if you had a function that added money to an account, and it was called as addMoneyToAccount(account, money), then it should return a new account object with the money added to it, and the previous instance of the account object remains in the previous state. This is called immutability, in that once an object or thing is created, it should never change as a side effect or something else. Instead, a new object is created as the result of the operation.

This is useful because it allows you to chain together operations by calling another function on the result. In JavaScript this can get quite unwieldy—imagine baz(bar(foo(thing))), but potentially even deeper. The code can be hard to follow, as the order of operations is backwards (i.e., foo executes first, then bar and finally baz). As a result, functional programming languages offer different ways of chaining functions like this (in Haskell, the syntax would be foo . bar . baz thing), and many functional programming frameworks in JavaScript offer ways to mimic this chaining. It may look unfamiliar, but remember that they are still function calls.

As a language that is neither purely object oriented nor functional, but allows you to mix both styles, it is hard to determine what is idiomatic JavaScript. When should classes be used, and when should functions be used? Often, it comes down to what you prefer, what already exists in a codebase, or the style used by any external libraries or frameworks you are building on top of. It is common to mix and match styles within a codebase, depending on the task at hand.

A reasonable rule of thumb is to use a functional style when you want to make more generic functions, which act on more generic data structures such as lists or dictionaries, and there are many ways you might want to manipulate that data. Classes are useful if you are working with data and want the methods of operating on it to be very tightly bound. Dates and times are good examples of things that can be classes, as the logic for manipulating them is tightly connected to the nature of the data. In this case, you might want to consider making these kinds of classes immutable, so the methods actually return new instances of the class rather than modify the one in place, to avoid some of the issues that come with classes and shared state. Other times classes might be useful is when you need wrappers around a particular resource, and multiple operations can operate on the same resource. An example might be an HTTP client that takes some configuration options, or a database connection. It is possible to use these functionally, but this can often be painful, as it requires passing around configuration or handles in many places (partial functions can help, but can also require a large number of functions to be passed around, whereas a class can simplify the process). In this case, state is often not modified once the class is constructed, and it acts simply as a collection of functions over some shared configuration or connection. Classes in this case can also correctly handle logic, such as connection pooling, reconnection, or circuit breakers, while presenting a simple interface to the user.

## Communicating Between Components

Early JavaScript often had very simple requirements, and didn’t require much code to be written. A single JavaScript file with little nesting or modularization, and global variables, may have sufficed for those needs, but as JavaScript applications have become larger and more complex, that is no longer the case. Instead, we structure JavaScript as modules, but those modules need to interact with one another. Coupling is the term used to describe the interaction between two modules, and how closely bound the modules are with each other.

It is usually desirable to have each module loosely coupled—that is, to minimize the amount of interaction between the two—through well-defined interfaces. By contrast, strong coupling would mean that two modules were highly linked together, interacting with each other very frequently in many different ways. Strong coupling can mean that the two modules become hard to separate and use independently. The concept of loose coupling is prevalent in object-oriented design, but applies to functional programming too.

The loosest coupling is one where there is no interaction whatsoever, but components often have to at least communicate state or changes to each other. One way to do that is to call methods directly on any component that needs to know when an event change happens. This, however, introduces a flaw in your component design—a component has to know every possible other component that could change based on that state, which can introduce a large amount of coupling. An alternate way to solve the problem of communicating state changes is by solving the problem the other way around. Rather than something that can generate a state change sending messages to those that need to respond to that change, you could set up the components that depend on that to register a callback when an event happens. However, this introduces a different form of coupling, as now these components have to know exactly which other components can generate those state changes.

A common way of getting around this is to use an event bus. In this scenario, things that cause state changes send messages to the event bus when an event occurs, and then others subscribe with callbacks to know when that has happened. In JavaScript, events are the predominant way the DOM indicates changes to interested code, but custom events can also be fired on the global window object, which allows a straightforward event bus to be created, where events get fired in the same way as DOM and subscribers can subscribe to that as needed.

There are many different architectural patterns that can be used with event buses. One pattern is known as a hexagonal architecture. In a hexagonal architecture, your app is split up into different layers (similar to model-view-controller). For example, at the very core is usually where domain models live, then this is surrounded by a layer that implements business logic, and eventually a layer that implements interaction with the outside world—not just UI, but also database and other persistence stores. Each layer communicates with the one above it through ports and adapters. A port is essentially an interface into a layer, and an adapter is what a layer implements to communicate with a particular port to transition from one layer to another. This hexagonal architecture often works well with another design pattern known as CQRS: command query responsibility segregation. In CQRS, the methods and data structures you use to interact with data and those you use to read data are different, with one corresponding to the adapters and the other to the ports.

This particular architecture can get very complicated when your application is closer to a CRUD (create-read-update-delete) use case. In CRUD, you are normally just editing underlying data structures, but CQRS is better for triggering particular events where the UI action is not directly linked to the underlying data structure changes.

For CRUD apps, a particular UI component may only be directly interested in one part of the state (or data model), so can set up event handlers on that data model directly, without necessarily using an event bus. The component in question can update the data model, and then set up events as needed. This is known as two-way binding, as communication happens in two ways. For apps that are more complicated and task-based, then separating the concerns through an event bus is often easier to manage, and instead a one-way binding to a data model is then used. In a one-way binding, a UI component fires an event into an event bus, and then receives a new/updated data model based on the changes. This separation means you do not need to know the detail of the whole structure of the underlying data model or how it needs to be manipulated—that can be left to a business logic layer. You only need to couple to the bits that give you the information you need to read, rather than what you need to change. With one-way binding, your application is closer to the CQRS application pattern, even if it is still performing CRUD-like applications.

Most event bus systems in JavaScript do not directly fire events through the DOM, but instead have their own event system outside of the DOM, which allows intermediate layers such as business logic to be better inserted.

One common JavaScript library for managing events with one-way flow is Redux. In Redux, events are triggered by dispatching an action. Redux provides a function called dispatch, which is bound to a particular instance of an event bus, and actions are defined by the developer as a particular thing that can be triggered. These are often linked to UI actions, but can be linked to other asynchronous events. Actions have a type, but can also carry other data. For example, you might have an action called SEND_MESSAGE that has the body of some text as an argument. A UI component dispatches this action when the user clicks “enter” in a chat window. For asynchronous actions, you may have a timer that triggers the background refresh of a user’s state, which may then fire an event called REQUEST_USER_STATUS to update away indicators of a friend’s list. Then, the action causes the business logic to fire a request to get a new list, and then, once it completes, calls another action, UPDATE_USER_STATUS, with the response of the request.

Dispatching an action causes an event to be created, which is then passed to the business logic. In Redux, these bits of business logic are called reducers, but they essentially take a current state and the event, and then return a new system state that has been manipulated according to the needs of that action. The components are then notified of a change of state, and Redux provides mapping functions that allow a component to only be bound to the parts of the state that it actually needs to give us the loose coupling we desire. This is what is meant by uni-directional flow; the state changes may be started by an action being dispatched in a component, but the actual state changes only happen in one direction, rather than a component changing the state or data model directly, as happens in a two-way binding system.

Redux works especially well with React. React is a JavaScript library developed by Facebook that re-renders UI components when the data model they are bound to changes, which can greatly simplify logic. The full details of React are outside the scope of this book, but essentially, React allows you to define UI components, often using a language known as JSX, which allows you to express these UI components in an HTML-like way. React then takes a tree of these components and renders them out to HTML. Each component can maintain its own state, and is passed properties (props) that can be used to determine how a particular component behaves. A component cannot directly change the props of its parent—only its direct children. If any part of the state of a system changes, which results in changes to the props, React re-evaluates the components where the props have changed to see if they have resulted in any change to the resulting HTML. If it did, React makes only the changes it needs. This can be more performant than potentially re-rendering the entire DOM, or making a series of incremental updates as the state changes. With React and Redux, the coupling for a component becomes the actions it needs to dispatch into the store, and the specific parts of the state it needs to read out, which are done by mapping actions and state to props of a component using connect. The business logic is completely decoupled from logic, allowing React to figure out how the state maps to the attributes the UI needs, while Redux handles events.

Using this kind of event bus system also allows us to implement state machines to capture the particular state of a system, instead of simply mapping directly to values of particular data models. A state machine is used to map flow through a system—often, a higher-level component organizes other components based on the state, and then the actions that the other components can trigger may cause transitions into other states. Figure [8-1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_8_Chapter.xhtml#Fig1) demonstrates one way of depicting a state machine, where the circles represent the states and the arrows are annotated with the actions that can cause a transition between those states.

![../images/471976_1_En_8_Chapter/471976_1_En_8_Fig1_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_8_Chapter/471976_1_En_8_Fig1_HTML.png)

Figure 8-1

A state machine for editing and saving an item

In the above state machine, we can see a process for editing an item in a web app. We start in the state of “List View”, and selecting “Edit Item” will cause the state to transition to “Update item view”, with the item that is being edited set to the state. In the “Update item view state” pressing the save item causes the state to transition back to the list view (normally with a side-effect of saving the changes back to a database), but you may also have a “cancel” transition that also transitions back to “List View”, but without the side-effect. Most systems will have a state machine somewhere, and although this can often be determined through the value of particular parts of the data model, it can be useful to directly model the current state as part of your data store, rather than trying to derive it from the values of the models in the data store, and being explicit in your business logic of the transitions, rather than leaving them implicit.

In state machines, though, some transitions are invalid, so care must be taken to only trigger an action that is valid from a particular state by a component. This is often done by making components that can cause a transition inactive during particular states.

## Connecting Components Together

Although it’s possible to reduce the coupling between components, there will often still be cases where you will need access to another component or library in your code, and there are two ways of doing this.

The first is to simply import the components or libraries that you depend on directly, and the second is a technique known as dependency injection, where a component is given specific objects that it depends on, rather than importing them itself. Considering the event bus approach with Redux discussed previously, this case also includes functions such as the dispatcher, which can be used to dispatch actions, and the bound values from the store, in addition to other utilities that might be needed.

For utility libraries, which are not configurable, the first approach often makes sense. Let’s say we need to generate unique identifiers for something in our code. We might want to do the following:

const uuid = require('uuid');

function buildCommentModel(commentText, threadId) {

  if (commentText.length === 0) {

    throw new Error('Comment is empty');

  }

  return {

    id: uuid.v4(),

    body: commentText,

    thread: threadId,

  };

}

Often, the libraries and functions you import this way will be functional—they will not have any state, just things you can configure after importing. Similarly, importing a class from a file, the state will live in the instance, not the class.

This approach of pulling in dependencies can be thought of as a module asking for what it needs, rather than being given it. However, there are many cases when importing things in this way is not helpful—for example, if the module needs access to a shared resource, or an external library takes some configuration, but you want to configure it once, rather than every time it’s used.

A tempting solution might be to create a module that exports an instance of your class, or an already-configured set of options for your functions. For example, many times in NodeJS we need to talk to other back-end services, and we import an HTTP client. However, especially in enterprise environments, it is not unusual for machines to live behind a proxy, or to require specific configuration—perhaps we want detailed logging in the development environment, but not in production. If every time we made an HTTP request we had to configure that HTTP client in every piece of code that imports the class, it would generate a lot of repeated code and require many changes every time we needed to change its behavior. A similar approach could be connections to databases—it’d be much more convenient for modules to simply get an active database connection than to create a new one from the raw classes or functions. It is tempting just to import an instance of an HTTP client that’s configured.

However, this breaks the rule of coupling. That module is now specifically linked to that particular instance in that module. This may seem fine, but when it comes to unit testing your code, you may not want to use a real HTTP client, as this can result in very slow tests, and makes setting up test data much harder. We want to replace that HTTP client with a mock, but when a module asks for a dependency, there’s no easy way to actually give it a different dependency than the one it asks for. There are tools, such as proxyquire, that replace the CommonJS require() with one that can return alternate options for the purposes of unit testing, but this can lead to non-obvious issues and spaghetti code, especially if, at a later time, we want to change which HTTP client we use in production (perhaps development and production environments use different proxies, or different TLS certificate authorities).

Furthermore, this approach actually reduces the flexibility of the component (a side effect of increasing the coupling). If we needed an HTTP client that was configured in a different way, then we’d have to create a new module that configures it differently, causing duplication.

A better solution is that of “dependency injection,” where a module is instead given what it is needed to do the job, rather than importing them directly. For libraries and components that need configuration to be useful, this is a much better approach that can lead to cleaner code. For a developer coming from a Java background, dependency injection instantly evokes ideas of large frameworks for managing these, but this does not need to be the case, and many JavaScript apps do not need a framework to manage these for you. Often, having a “main” function that sets up all your dependencies and then wires it up is sufficient.

For classes, it is typical to set these dependencies through the constructor, and then reference them as fields when needed:

class ProductCatalogueApi {

  constructor(httpClient, catalogueApiUrl) {

    this._httpClient = httpClient;

    this._catalogueApiUrl = catalogueApiUrl

  }

  fetchCatalogueItem(id) {

    return this._httpClient

      .get(`—{this._catalogueApiUrl}/product/—{id}`)

      .json();

  }

}

In the example above, the ProductCatalogueApi class could then be instantiated in the main function and passed through to any controllers that need access to the product catalogue, again using dependency injection.

For modules that aim to be more functional, a closure can be used instead, where the module exports a function that takes in the dependencies and then returns the function that does the work of that module, which now has access to those dependencies in the scope of the closure.

function fetchCatalogueItemFactory(httpClient, catalogueApiUrl) {

  function fetchCatalogueItem(id) {

    return this._httpClient

      .get(`—{this._catalogueApiUrl}/product/—{id}`)

      .json();

  }

  return fetchCatalogueItem;

}

With this approach, when getting testing your class or function, you can set them up in the same way as your main application’s code, but pass in compatible stubs with dummy data as the dependencies.

## Testing

There are many options for running tests in JavaScript, with the most popular ones supporting the arrange-act-assert mechanism discussed in the Testing chapter. Specific styles tend to vary, with libraries like Jasmine allowing you to write the tests, and tools like Karma allowing you to run them from the command line and reporting on their success.

Earlier unit testing tools, like QUnit, run by opening a browser and visualizing the result. JSTestDriver was another popular early testing tool for JavaScript. Written in Java, it allows you to write tests very similar to the JUnit style, and manages reporting and execution in browsers too. These tools are rarely used for new projects, although you may come across them. The rise of headless browsers like PhantomJS, and then server-side runtimes like NodeJS, have simplified JavaScript test running. I would recommend Jest as the best tool for test running; it is an all-in-one test running and specification library that is compatible with Jasmine.

In Jasmine and Jest, you define a test by calling the function it, then passing a human-readable name of the test and the function to be run when the tests are run.

const ShoppingBasket = require('../lib/shopping-basket');

const Product = require('../lib/product');

describe('shopping basket', () => {

  it('can be added to', () => {

    const shoppingBasket = new ShoppingBasket();

    shoppingBasket.addItem(new Product('t-shirt'));

    expect(shoppingBasket.items).toBe(1);

  });

});

The describe() method allows for grouping, and a block of tests inside a describe() can also include another describe() nested inside of it. The method names and descriptions are supposed to encourage you to write natural language–inspired names, because when the test runs, the description of all the describes together, and that of the test itself, is used as the overall test name in the output. In the example above, this would be “shopping basket can be added to.”

When writing many tests for a module, it is common to execute the same setup for each test, perhaps having to construct an instance of a class, or tear it down at the end. Most tools allow you to specify a common bit of code to run before and after each test (regardless of whether or not it passed) to avoid this repetition. In Jest/Jasmine, this is done by invoking beforeEach() and afterEach() with a callback

describe('shopping basket', () => {

  let shoppingBasket;

  beforeEach(() => {

    shoppingBasket = new ShoppingBasket();

  });

  it('can be added to', () => {

    shoppingBasket.addItem(new Product('t-shirt'));

    expect(shoppingBasket.items).toBe(1);

  });

  afterEach(() => {

    shoppingBasket.empty();

  });

});

By default, each test will be executed synchronously. If you are testing asynchronous code (for example, if the method you’re testing returns a promise), then you need to have some way of telling the test that it should wait for all the asynchronous code to have completed. In Jest, you can return a promise to do this, and then the test will pass if the promise resolves, or fail if the promise rejects. Let’s assume adding an item to the shopping basket is now an asynchronous operation that returns a promise:

describe('shopping basket', () => {

  let shoppingBasket;

  beforeEach(() => {

    shoppingBasket = new ShoppingBasket();

  });

  it('can be added to', () => {

    return shoppingBasket.addItem(new Product('t-shirt'))

      .then(() => expect(shoppingBasket.items).toBe(1));

  });

  afterEach(() => {

    shoppingBasket.empty();

  });

});

Another way of doing this is by taking in a single argument—normally called done—to your test function. You then call done() at the end of your tests, and done.fail() if you need to fail the tests.

describe('shopping basket', () => {

  let shoppingBasket;

  beforeEach(() => {

    shoppingBasket = new ShoppingBasket();

  });

  it('can be added to', (done) => {

    shoppingBasket.addItem(new Product('t-shirt'))

      .then(() => {

        expect(shoppingBasket.items).toBe(1);

        done();

      });

  });

  afterEach(() => {

    shoppingBasket.empty();

  });

});

If there is a bug in your asynchronous code such that the callback or promise never gets called, Jest/Jasmine will time out waiting for a test to complete (although the error messages might not be useful!)

Testing simple logic with JavaScript tests can be easy, as the output depends purely on the input, sometimes with some side effects on other members of a class or other dependencies that can be passed in and replaced with stubs or mocks (fake versions that simplify the external system’s behavior—you can read more about these later in the chapter). JavaScript components that operate on the DOM can be hard to test. The JavaScript might assume that the DOM is in a particular shape when instantiated (for example, provided by the HTML), and the DOM is a form of global state. One test might leave it to be manipulated in a way that can cause the next test to fail, even if that test would otherwise be fine.

Using virtual DOM libraries like React can make this easier, as the virtual DOM can be replaced by a mocked-out version, and assertions can be made on that. There are libraries (such as Enzyme for React) that can help with this. For traditional “vanilla” JavaScript, a popular technique is to pass in the DOM element that the component operates on in as a dependency, rather than to let it find it in the global DOM. For example, when using a function to add a click handler to something, instead of the function using document.querySelector, it would instead be passed an argument that is the HTMLElement object to add the click handler to. This is an application of dependency injection.

With this approach, a new layer is needed above the individual components that does all the lookups from the real document and passes them through, but this allows for greater reuse of an individual component if need be. This is sometimes called the wiring, as it connects the individual components together. When we want to test a component, then our testing code can take on the role of this wiring layer. We can then construct fake snippets of the DOM in the test, perhaps by loading in a “fixture” file (an HTML file that contains a small snippet of the site that is needed to test this component), and then pass this constructed element through, without actually attaching it to the global document. This allows us to isolate each test from each other, although there is still a risk of leaks to global states, especially if things such as timers are set up. You will often end up writing ways of clearing any manipulation of global state up—for example, having a “destroy()” method on a class that uses clearTimeout(), or using lifecycle hooks such as componentWillUnmount in React, even if the destroy code never needs to be run in the running application. The test code will need to call these, and it can also be a good habit for writing real code, as it can help you avoid performance issues. Even if at first on the real page only one instance of a component will ever be created, this may not always be true, and the habit will help you there.

The kinds of tests written in this way are normally called unit tests. You can read more about these later in the Testing chapter, but in general they allow you to test one bit of your code in isolation, in many different configurations (including ones where it should error). There are many benefits of unit testing. In addition to regression suites and ensuring that your code is functionally correct, unit testing helps with the design of the interface of your modules. Applying test-driven development can help focus the role and responsibility of the module, but your module now automatically has two things that use it: the test code and the production code. If your code is hard to test, the design of the interface may not be optimal, or it may be too heavily coupled to another module. This may not cause you immediate pain, but can down the road, as code that has these characteristics is often hard to change, and being able to react quickly to changing business needs is one of the real benefits of agile software development. The chapter on testing has a full introduction to applying test-driven development.

Once you have written your tests, you will need a way to run them. Although server-side tests can be run as code in NodeJS, tests for client-side code must run in an environment similar to a browser in order for the code under test to behave appropriately. When JQuery was prominent, it came with a unit testing framework known as QUnit. With QUnit, it was common to run your tests manually in a browser window for each browser you needed to target, as a browser is, of course, the most browser-like environment out there.

This integrated poorly into continuous integration workflows, and also added maintenance overhead. It wasn’t long before automation tools such as Karma came about, which ran tests programmatically by booting and remotely controlling browsers. Still, this caused problems for automated test running on a CI server, which ran in a headless mode or on Linux systems rather than the OS the browsers needed. Opening a browser to run the tests was slow, especially compared to the workflow of server-side code with frameworks like JUnit, which could run tests in milliseconds from within an editor. PhantomJS was a “headless” browser, based on WebKit, that worked in a cross-platform way without a UI and became a popular way to run unit tests, as it significantly sped up the load time for the browser, and therefore the tests themselves.

More recent developments have allowed server-side and client-side code to merge. Instead of trying to force a browser to run programmatically, libraries such as JSDOM instead make the NodeJS runtime behave like a browser. Although it might seem dangerous not to test the code in the actual environment it will be running in, the risks are actually minimal, and the benefits of quick test runs outweigh them. In modern browsers, most bugs are a result of logic errors in your code, rather than browser incompatibilities.

There are options for those who want to run their code in actual browsers, though. End-to-end tests, as opposed to unit tests, are commonly run in real browsers. There tend to be fewer end-to-end tests than unit tests, but each test will touch more parts of the system and can give you a higher degree of confidence in your system than a single unit test. End-to-end tests are also slower to run, and combined with the overhead of starting up browsers, this can make these tests especially slow to run. Running these tests also tends to happen in a different way than your unit tests—instead of your test code running in the same browser context as the code it tests, it instead spins up a different browser and remotely controls that browser (or possibly multiple browsers in parallel). This means you can actually write your integration tests in other languages (Java and Ruby are two common ones), although JavaScript itself is still common.

A library is used to support this, and by far the most mature library is known as Selenium. Selenium is a Java library, but defines an API known as WebDriver, which runs in the browsers. Other languages then have Selenium-like libraries, which interact with the WebDriver API to control the browsers remotely to run tests. With WebDriver, you can create tests within your normal testing framework, but these tests then use that library to control a browser remotely and make expectations/assertions through that library as per usual.

describe('shopping basket', () => {

  it('increases the number of items shown on the page', () => {

    browser.url('http://localhost:3000/catalogue/t-shirt');

    expect(browser.getText('#cart_size')).toEqual("Empty");

    browser.click('#add_to_cart');

    expect(browser.getText('#cart_size')).toEqual("1 item");

  });

});

The main difference between Selenium and mocking tools such as JSDOM is that you need to get a real browser instance to run on. When running these tests on a desktop, it’s not unusual to see copies of browsers start up and physically watch the remotely controlled interactions occur. In CI environments, a GUI environment must be provisioned to allow this to happen. This is not always straightforward, and maintaining a suite of different OS’s and browsers to provide full coverage can be a considerable overhead for an organization. Several SaaS providers offer pay-as-you-go, “cloud-based” browsers on infrastructure they control, which can be helpful in some circumstances (but not others—for example, if you want to check an intranet or lock your development environments down to specific IPs).

A common issue with end-to-end test suites is reliability, especially across browsers. In browsers, almost all actions should be treated as asynchronous, which can make writing tests tricky. A common way around this, especially in languages that do not have as much support for asynchronous functions as JS, is to add delays to the test. Say, for example, opening a dialogue box has a 200ms animation; it is common to see code such as:

describe('control panel', () => {

  it('opens when clicked', (done) => {

    browser.url('http://localhost:3000/')

    browser.click('#open_control_panel')

    setTimeout(() => {

      expect(browser.isVisible('#control_panel')).toBeTruthy();

      done();

    }, 250);

  });

});

The wait above is actually for 250ms, not 200, to allow for some overhead or variance in the browser’s rendering engine. Where this approach really falls down is for actions where the delay is not exactly determined, such as for AJAX calls. Often, a delay is a number based on the worst-case scenario, which might still not be 100% reliable, and may introduce slowness if the call does complete more quickly. An alternative approach is known as “waiting,” where you periodically check whether an action has completed, and then continue as soon as it has. Otherwise, once a timeout has been reached, the action fails.

describe('control panel', () => {

  it('opens when clicked', (done) => {

    browser.url('http://localhost:3000/')

    browser.click('#open_control_panel')

    browser.waitForVisible('#control_panel');

    expect(browser.isVisible('#control_panel')).toBeTruthy();

  });

});

This allows you to write faster and more resilient tests, and also helps decouple your code from the implementation. For example, if your animation speed becomes slightly slower, you would not necessarily need to update your test code, which you would have to do if you had used a sleep with a hard-coded interval.

Another approach to decoupling your test code from the specific implementation details even further is known as the page object model. Selenium uses CSS classes (or IDs) to find items on a page, but sometimes those classes need to change (if a major restyle is underway), but the actual behavior and structure of the components stays the same. Instead of writing several tests that refer to the same component by class name, you can instead encapsulate all the logic inside a helper class, and then make assertions about that class. If any information about an object on the page changes, then the definition in the class gets updated and all the classes use it. This is a good application of the DRY (don’t repeat yourself) pattern.

When running your integration tests, you also have to consider how to control the state of the application under the hood. In unit tests, techniques like mocking can be used, but with these kind of tests, you are often running your app as a standalone server, which you do not have access to at runtime (although starting an instance of the server within the test code is also a possibility, and does allow you to directly manipulate the state of the server-side code). One way to solve this is to point your app at a fake database or API that you can control, rather than a real one, which makes it easier to integration test your app in several ways. Another interesting approach is to actually run these tests against your real web site. Some people prefer this because it offers a high degree of confidence in the version of the web site your audience will actually see, but it is not always appropriate. For example, making a real order on an e-commerce web site can be hard to test. Sometimes a pre-staging web site is used instead. Normally, the real web site must be provisioned with some test data that can cause confusion if a real user stumbles across it (see YouTube’s “Webdriver Torso” tests), or depends on the existence of specific bits of content that, if changed, can break your tests.

## Build Tools

As mentioned previously, it is rare to write JavaScript that is then executed by a browser with no interim transformation. Even when you do, it’s still good to have some automated tooling that can help you execute tests or apply other quality checks to your codebase.

At the time of writing, there is no standard build tool for JavaScript, with several popular frameworks for doing so. However, most build toolchains have commonalities, and tend to help out in three ways: managing dependencies, checking code quality, and producing the end product (a “bundle”), which the browsers execute. It is common to put together different tools to fulfill all of these functions in a toolchain, although some do more than others. When different tools need to be combined a coordination layer is needed. This can be as simple as a shell script that runs different commands in sequence, or something more advanced (such as Gulp or Grunt) that supports parallelization and more complex chains, as well as watching for changes and rerunning builds during development.

The usual purposes of a build tool are to bundle multiple JavaScript files together into one, run any transpilation needed, and resolve dependencies from imported code—either other modules in the same system or third-party code installed from a repository. A single file can then be delivered to the browser in one request, which can offer some performance enhancements (although HTTP/2 offers alternative ways of negating the performance impact of making multiple requests from a browser). This file is often also minified, especially in production, meaning that any human niceties, such as white space, are stripped out and the variable made smaller to further reduce the size of the file. The downside of this is that it can make debugging harder, as the browser only sees the minified code. Source maps were invented to counteract this, and they provide a way for browser developer tools to relate the minified, transpiled code back to the original. It is common for build tools to have separate “development” and “production” modes for this process, as source maps can significantly increase the size of a file, so they are only included on local development builds.

Although often not part of the build tool itself, the test runner, dependency manager, and code quality checker and often bundled together into the same toolchain. Early JavaScript projects would either simply save any dependencies in the code repository alongside the code for this application (a process known as “vendoring,” as the common location for these files was in a “vendor” subdirectory), or sometimes used features like SVN externals or Git submodules. It was also common (and remains so today) to include a remotely hosted version of common libraries such as JQuery as a script tag on your page and assume those dependencies were there.

In 2012, Twitter introduced a tool known as “Bower,” which added tooling over the population of the vendor directory, by specifying which dependencies and versions you required in a config file and automatically downloading them. As NodeJS became popular, a tool known as Node Package Manager (NPM) became popular alongside it, which managed JavaScript dependencies for server-side code. It wasn’t long before this became the most common way to distribute JavaScript code intended for execution in a browser too. NPM (and related tools, such as Yarn) is now considered the standard way to include dependencies into your JavaScript. The common build tools will understand how to resolve dependencies that have been installed using NPM, either using ES6 import or the CommonJS require syntax.

Code style and quality checkers have been long established as a powerful friend in writing maintainable code. The first style checker, called “lint,” was created for the C language, and a “linter” is now the term used to refer to the same tools in other languages. There are several linters for JavaScript, with ESLint being one of the more modern and common. Over time, linters became responsible for more than just checking for style consistency, but also for potential mistakes in logic (such as using a variable that could have possibly not had a value assigned to it). Type checking tools such as Flow, as discussed previously, are also often integrated into a build toolchain here.

Code style is also the source of many heated debates within a development team, which can be reflected in the vast array of configuration options available for linting tools. For many other languages, the body responsible for publishing that language will also publish recommended style guides, such as PHP’s PSR-2, or Python’s PEP-8. For JavaScript, there is no one standard, but several popular styles. I would recommend just picking one of those styles and stick with it, as a simpler approach than defining your own internal style. Doing so will save many heated, and often unnecessary, discussions among your team members over personal preferences!

Some of this will seem similar to the roles of the build tools used for other parts of the front end, and in these case they are often merged into one tool or task runner that handles both.

## Summary

Despite originating as a language to manipulate web pages, JavaScript is now often used server-side as well. For now, JavaScript remains the dominant language of the Web for client-side code, so must be a sharpened tool for any full stack web developer.

JavaScript is a rapidly evolving language. Following a long period of stagnation, with some browsers adding incompatible extensions that became ad-hoc parts of the language, 2015 saw the introduction of the sixth edition of the language specification, called ES6, and also a shift to a yearly model (so ES6 is ES2015), with smaller iterations constantly being added. Beyond being used for simple enhancements to a page, JavaScript can now be used for extremely rich and complex web applications.

As a language, JavaScript has some distinct features. One of those is its asynchronous nature. JavaScript runtimes run the code in a single thread, with I/O operations happening in the background, and functions registered as event handlers or callbacks that occur when a UI event or background I/O process happens. JavaScript has several features to help simplify the use of callbacks, and especially chained callbacks: promises and, more recently, the async and await keywords.

In the browser, JavaScript interacts with the UI using the Document Object Model (DOM) API. As JavaScript code is run by a potentially wide range and age of browsers, this can limit you to a lowest common denominator of the language, but there are tools that work around this by either making new language features available in older browsers (polyfills), or by transforming newer syntax into a backwards-compatible form (transpilation). Some libraries, such as JQuery, have also historically been used to provide a common API over multiple incompatible implementations of the same functionality.

On the server, the NodeJS runtime is used to run code, but with a different standard library, and particularly no DOM. NodeJS also has a technique that allows for JavaScript modules to be included to manage dependencies, using a technique known as CommonJS. On the browser, asynchronous module definitions (AMDs) were common for defining and loading other modules and namespaces, but CommonJS code can be transpiled into a form compatible with modern browsers. Becoming available is a native JavaScript module type, which is usable without any additional tool support.

JavaScript supports a small set of basic types, with more complex functionality developed by combining those. A common “gotcha” with JavaScript is the equality operator (==), which will coerce the two sides into a similar type to compare them, yielding surprising results. You will almost always want to use the identity operator (===), which behaves in a way that is less surprising. JavaScript’s object-orientation approach differs from other languages too, using prototype-based, rather than class-based, inheritance, although ES6 allows you to work in a more familiar syntax. JavaScript also supports some basic primitives for functional programming—most importantly, anonymous functions that can be passed as first-class objects. JavaScript’s scoping rules are often surprising too, where variables defined using var are subject to “hoisting,” which happens when the definitions are evaluated before the code. The modern alternatives of let and const are preferred for newer code.

The other biggest pitfall of JavaScript is the this keyword. In other object-oriented languages, this refers to the object that the method exists on, but in JavaScript it refers to the context in which the method was called (potentially as a method, but also perhaps as an event callback). Closures or binding can be used to ensure that you have access to the this keyword or an equivalent when adding event handlers or callbacks. Event handlers and callbacks are a common way to communicate between components, and there are many libraries and frameworks to help structure event processing frameworks, as well as binding between models and UI code.

As a language, JavaScript has good tool support for writing and running automated tests. These include unit tests, which test a distinct part of an application, and integration tests, which run in the full browser environment. JavaScript unit testing frameworks borrow extensively from the common principles of unit testing, although they are often structured using describe and it statements, rather than as methods on classes. This way of structuring tests reflects JavaScript’s nature as neither a pure OO, procedural nor functional language.

Although it is possible to write JavaScript directly for execution by a browser, it is more common to use tools to translate JavaScript into a backwards-compatible and optimized form for use in a web browser. These build tools also work with dependency management tools such as NPM, where third-party JavaScript libraries can be utilized and bundled into your final app.

JavaScript is an often imperfect language. But JavaScript is the language of the web, and as a web developer, having a good understanding of how to develop software in JavaScript will see you most of the way to satisfying the technical requirements of being a full stack developer.