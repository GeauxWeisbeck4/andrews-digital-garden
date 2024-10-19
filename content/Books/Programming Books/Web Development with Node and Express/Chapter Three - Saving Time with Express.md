---
id: 01JAH06BSJ005PQSR1M45JQXHH
modified: 2024-10-18T19:47:15-04:00
title: Chapter Three - Saving Time with Express
tags:
  - express
  - javascript
  - nodejs
  - books
  - programming
---
# Chapter 3. Saving Time with Express

In [Chapter 2](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch02.html#ch_getting_started_with_node), you learned how to create a simple web server using only Node. In this chapter, we will re-create that server using Express. This will provide a jumping-off point for the rest of the content of this book and introduce you to the basics of Express.

# Scaffolding

_Scaffolding_ is not a new idea, but many people (myself included) were introduced to the concept by Ruby. The idea is simple: most projects require a certain amount of so-called _boilerplate_ code, and who wants to re-create that code every time you begin a new project? A simple way is to create a rough skeleton of a project, and every time you need a new project, you just copy this skeleton, or template.

Ruby on Rails took this concept one step further by providing a program that would automatically generate scaffolding for you. The advantage of this approach is that it could generate a more sophisticated framework than just selecting from a collection of templates.

Express has taken a page from Ruby on Rails and provided a utility to generate scaffolding to start your Express project.

While the Express scaffolding utility is useful, I think it’s valuable to learn how to set up Express from scratch. In addition to learning more, you have more control over what gets installed and the structure of your project. Also, the Express scaffolding utility is geared toward server-side HTML generation and is less relevant for APIs and single-page applications.

While we won’t be using the scaffolding utility, I encourage you to take a look at it once you’ve finished the book: by then you’ll be armed with everything you need to know to evaluate whether the scaffolding it generates is useful for you. For more information, see the [`express-generator` documentation](http://bit.ly/2CyvvLr).

# The Meadowlark Travel Website

Throughout this book, we’ll be using a running example: a fictional website for Meadowlark Travel, a company offering services for people visiting the great state of Oregon. If you’re more interested in creating an API, have no fear: the Meadowlark Travel website will expose an API in addition to serving a functional website.

# Initial Steps

Start by creating a new directory: this will be the root directory for your project. In this book, whenever we refer to the project directory, app directory, or project root, we’re referring to this directory.

###### Tip

You’ll probably want to keep your web app files separate from all the other files that usually accompany a project, such as meeting notes, documentation, etc. For that reason, I recommend making your project root a subdirectory of your project directory. For example, for the Meadowlark Travel website, I might keep the project in _~/projects/meadowlark_, and the project root in _~/projects/meadowlark/site_.

npm manages project dependencies—as well as metadata about the project—in a file called _package.json_. The easiest way to create this file is to run `npm init`: it will ask you a series of questions and generate a _package.json_ file to get you started (for the “entry point” question, use _meadowlark.js_ for the name of your project).

###### Tip

Every time you run npm, you may get warnings about a missing description or repository field. It’s safe to ignore these warnings, but if you want to eliminate them, edit the _package.json_ file and provide values for the fields npm is complaining about. For more information about the fields in this file, see the [npm _package.json_ documentation](http://bit.ly/2O8HrbW).

The first step will be installing Express. Run the following npm command:

npm install express

Running `npm install` will install the named package(s) in the _node_modules_ directory and update the _package.json_ file. Since the _node_modules_ directory can be regenerated at any time with npm, we will not save it in our repository. To ensure we don’t accidentally add it to our repository, we create a file called _.gitignore_:

# ignore packages installed by npm
node_modules

# put any other files you don't want to check in here, such as .DS_Store
# (OSX), *.bak, etc.

Now create a file called _meadowlark.js_. This will be our project’s entry point. Throughout the book, we will simply be referring to this file as the _app file_ (_ch03/00-meadowlark.js_ in the companion repo):

```
const
```

###### Tip

Many tutorials, as well as the Express scaffolding generator, encourage you to name your primary file _app.js_ (or sometimes _index.js_ or _server.js_). Unless you’re using a hosting service or deployment system that requires your main application file to have a specific name, I don’t feel there’s a compelling reason to do this, and I prefer to name the primary file after the project. Anyone who’s ever stared at a bunch of editor tabs that all say “index.html” will immediately see the wisdom of this. `npm init` will default to _index.js_; if you use a different name for your application file, make sure to update the `main` property in _package.json_.

You now have a minimal Express server. You can start the server (`node meadowlark.js`) and navigate to _http://localhost:3000_. The result will be disappointing: you haven’t provided Express with any routes, so it will simply give you a generic 404 message indicating that the page doesn’t exist.

###### Note

Note how we choose the port that we want our application to run on: `const port = process.env.PORT || 3000`. This allows us to override the port by setting an environment variable before you start the server. If your app isn’t running on port 3000 when you run this example, check to see whether your `PORT` environment variable is set.

Let’s add some routes for the home page and an About page. Before the 404 handler, we’ll add two new routes (_ch03/01-meadowlark.js_ in the companion repo):

```
app
```

`app.get` is the method by which we’re adding routes. In the Express documentation, you will see `app.METHOD`. This doesn’t mean that there’s literally a method called `METHOD`; it’s just a placeholder for your (lowercased) HTTP verbs (`get` and `post` being the most common). This method takes two parameters: a path and a function.

The _path_ is what defines the route. Note that `app.METHOD` does the heavy lifting for you: by default, it doesn’t care about the case or trailing slash, and it doesn’t consider the querystring when performing the match. So the route for the About page will work for _/about_, _/About_, _/about/_, _/about?foo=bar_, _/about/?foo=bar_, etc.

The _function_ you provide will get invoked when the route is matched. The parameters passed to that function are the request and response objects, which we’ll learn more about in [Chapter 6](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch06.html#ch_the_request_and_response_objects). For now, we’re just returning plain text with a status code of 200 (Express defaults to a status code of 200—you don’t have to specify it explicitly).

###### Tip

I highly recommend getting a browser plug-in that shows you the status code of the HTTP request as well as any redirects that took place. It will make it easier to spot redirect issues in your code or incorrect status codes, which are often overlooked. For Chrome, Ayima’s Redirect Path works wonderfully. In most browsers, you can see the status code in the Network section of the developer tools.

Instead of using Node’s low-level `res.end`, we’re switching to using Express’s extension, `res.send`. We are also replacing Node’s `res.writeHead` with `res.set` and `res.status`. Express is also providing us a convenience method, `res.type`, which sets the `Content-Type` header. While it’s still possible to use `res.writeHead` and `res.end`, it isn’t necessary or recommended.

Note that our custom 404 and 500 pages must be handled slightly differently. Instead of using `app.get`, we are using `app.use`. `app.use` is the method by which Express adds _middleware_. We’ll be covering middleware in more depth in [Chapter 10](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch10.html#ch_middleware), but for now, you can think of this as a catchall handler for anything that didn’t get matched by a route. This brings us to an important point: _in Express, the order in which routes and middleware are added is significant_. If we put the 404 handler above the routes, the home page and About page would stop working; instead, those URLs would result in a 404. Right now, our routes are pretty simple, but they also support wildcards, which can lead to problems with ordering. For example, what if we wanted to add subpages to About, such as _/about/contact_ and _/about/directions_? The following will not work as expected:

```
app
```

In this example, the `/about/contact` and `/about/directions` handlers will never be matched because the first handler uses a wildcard in its path: `/about*`.

Express can distinguish between the 404 and 500 handlers by the number of arguments their callback functions take. Error routes will be covered in depth in [Chapter 10](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch10.html#ch_middleware) and [Chapter 12](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch12.html#ch_production_concerns).

Now you can start the server again and see that there’s a functioning home page and About page.

So far, we haven’t done anything that couldn’t be done just as easily without Express, but already Express is providing us some functionality that isn’t immediately obvious. Remember in the previous chapter how we had to normalize `req.url` to determine what resource was being requested? We had to manually strip off the querystring and the trailing slash and convert to lowercase. Express’s router is now handling those details for us automatically. While it may not seem like a large thing now, it’s only scratching the surface of what Express’s router is capable of.

## Views and Layouts

If you’re familiar with the “model-view-controller” paradigm, then the concept of a view will be no stranger to you. Essentially, a _view_ is what gets delivered to the user. In the case of a website, that usually means HTML, though you could also deliver a PNG or a PDF or anything that can be rendered by the client. For our purposes, we will consider views to be HTML.

A view differs from a static resource (like an image or CSS file) in that a view doesn’t necessarily have to be static: the HTML can be constructed on the fly to provide a customized page for each request.

Express supports many different view engines that provide different levels of abstraction. Express gives some preference to a view engine called _Pug_ (which is no surprise, because it is also the brainchild of TJ Holowaychuk). The approach Pug takes is minimal: what you write doesn’t resemble HTML at all, which certainly represents a lot less typing (no more angle brackets or closing tags). The Pug engine then takes that and converts it to HTML.

###### Note

Pug was originally called Jade, and the name changed with the release of version 2 because of a trademark issue.

Pug is appealing, but that level of abstraction comes at a cost. If you’re a frontend developer, you have to understand HTML and understand it well, even if you’re actually writing your views in Pug. Most frontend developers I know are uncomfortable with the idea of their primary markup language being abstracted away. For this reason, I am recommending the use of another, less abstract templating framework called _Handlebars_.

Handlebars (which is based on the popular language-independent templating language Mustache) doesn’t attempt to abstract away HTML for you: you write HTML with special tags that allow Handlebars to inject content.

###### Note

In the years following the original release of this book, React has taken the world by storm…which abstracts HTML away from frontend developers! Viewed through that lens, my prediction that frontend developers didn’t want HTML abstracted away hasn’t stood the test of time. However, JSX (the JavaScript language extension that most React developers use) is (almost) identical to writing HTML, so I wasn’t entirely wrong.

To provide Handlebars support, we’ll use Eric Ferraiuolo’s `express-handlebars` package. In your project directory, execute the following:

npm install express-handlebars

Then in _meadowlark.js_, modify the first few lines (_ch03/02-meadowlark.js_ in the companion repo):

```
const
```

This creates a view engine and configures Express to use it by default. Now create a directory called _views_ that has a subdirectory called _layouts_. If you’re an experienced web developer, you’re probably already comfortable with the concepts of _layouts_ (sometimes called _master pages_). When you build a website, there’s a certain amount of HTML that’s the same—or very close to the same—on every page. It not only becomes tedious to rewrite all that repetitive code for every page, but also creates a potential maintenance nightmare: if you want to change something on every page, you have to change _all_ the files. Layouts free you from this, providing a common framework for all the pages on your site.

So let’s create a template for our site. Create a file called _views/layouts/main.handlebars_:

```
<!doctype html>
```

The only thing that you probably haven’t seen before is this: `{{{body}}}`. This expression will be replaced with the HTML for each view. When we created the Handlebars instance, note we specified the default layout (`defaultLayout: \'main'`). That means that unless you specify otherwise, this is the layout that will be used for any view.

Now let’s create view pages for our home page, _views/home.handlebars_:

```
<h1>
```

Then our About page, _views/about.handlebars_:

```
<h1>
```

Then our Not Found page, _views/404.handlebars_:

```
<h1>
```

And finally our Server Error page, _views/500.handlebars_:

```
<h1>
```

###### Tip

You probably want your editor to associate _.handlebars_ and _.hbs_ (another common extension for Handlebars files) with HTML to enable syntax highlighting and other editor features. For vim, you can add the line `au BufNewFile,BufRead *.handlebars set filetype=html` to your _~/.vimrc_ file. For other editors, consult your documentation.

Now that we have some views set up, we have to replace our old routes with new ones that use these views (_ch03/02-meadowlark.js_ in the companion repo):

```
app
```

Note that we no longer have to specify the content type or status code: the view engine will return a content type of `text/html` and a status code of 200 by default. In the catchall handler, which provides our custom 404 page, and the 500 handler, we have to set the status code explicitly.

If you start your server and check out the home or About page, you’ll see that the views have been rendered. If you examine the source, you’ll see that the boilerplate HTML from _views/layouts/main.handlebars_ is there.

Even though every time you visit the home page, you get the same HTML, these routes are considered _dynamic content_, because we could make a different decision each time the route gets called (which we’ll see plenty of later in this book). However, content that really never changes, in other words, static content, is common and important, so we’ll consider static content next.

## Static Files and Views

Express relies on _middleware_ to handle static files and views. Middleware is a concept that will be covered in more detail in [Chapter 10](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch10.html#ch_middleware). For now, it’s sufficient to know that middleware provides modularization, making it easier to handle requests.

The `static` middleware allows you to designate one or more directories as containing static resources that are simply to be delivered to the client without any special handling. This is where you would put things such as images, CSS files, and client-side JavaScript files.

In your project directory, create a subdirectory called _public_ (we call it _public_ because anything in this directory will be served to the client without question). Then, before you declare any routes, you’ll add the `static` middleware (_ch03/02-meadowlark.js_ in the companion repo):

```
app
```

The `static` middleware has the same effect as creating a route for each static file you want to deliver that renders a file and returns it to the client. So let’s create an _img_ subdirectory inside _public_ and put our _logo.png_ file in there.

Now we can simply reference _/img/logo.png_ (note, we do not specify `public`; that directory is invisible to the client), and the `static` middleware will serve that file, setting the content type appropriately. Now let’s modify our layout so that our logo appears on every page:

```
<body>
```

###### Note

Remember that middleware is processed in order, and static middleware—which is usually declared first or at least very early—will override other routes. For example, if you put an _index.html_ file in the _public_ directory (try it!), you’ll find that the contents of that file get served instead of the route you configured! So if you’re getting confusing results, check your static files and make sure there’s nothing unexpected matching the route.

## Dynamic Content in Views

Views aren’t simply a complicated way to deliver static HTML (though they can certainly do that as well). The real power of views is that they can contain dynamic information.

Let’s say that on the About page, we want to deliver a “virtual fortune cookie.” In our _meadowlark.js_ file, we define an array of fortune cookies:

```
const
```

Modify the view (_/views/about.handlebars_) to display a fortune:

```
<h1>
```

Now modify the route _/about_ to deliver the random fortune cookie:

```
app
```

Now if you restart the server and load the _/about_ page, you’ll see a random fortune, and you’ll get a new one every time you reload the page. Templating is incredibly useful, and we will be covering it in depth in [Chapter 7](https://learning.oreilly.com/library/view/web-development-with/9781492053507/ch07.html#ch_templating).

## Conclusion

We’ve created a basic website with Express. Even though it’s simple, it contains all the seeds we need for a full-featured website. In the next chapter, we’ll be crossing our _t_s and dotting our _i_s in preparation for adding more advanced functionality.