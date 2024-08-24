---
id: 01J63449FS1VFF7PX03GQ1TSY4
modified: 2024-08-24T19:34:16-04:00
---
# Chapter One - Building a Blog Application with Django 5

## Install Stuff here


# Django overview

Django is a framework consisting of a set of components that solve common web development problems. Django components are loosely coupled, which means they can be managed independently. This helps separate the responsibilities of the different layers of the framework; the database layer knows nothing about how the data is displayed, the template system knows nothing about web requests, and so on.

Django offers maximum code reusability by following the **DRY** (**don’t repeat yourself**) principle. Django also fosters rapid development and allows you to use less code by taking advantage of Python’s dynamic capabilities, such as introspection.

You can read more about Django’s design philosophies at [https://docs.djangoproject.com/en/5.0/misc/design-philosophies/](https://docs.djangoproject.com/en/5.0/misc/design-philosophies/).

## Main framework components

Django follows the **MTV** (**Model-Template-View**) pattern. It is a slightly similar pattern to the well-known **MVC** (**Model-View-Controller**) pattern, where the template acts as the view and the framework itself acts as the controller.

The responsibilities in the Django MTV pattern are divided as follows:

- **Model**: This defines the logical data structure and is the data handler between the database and the view.
- **Template**: This is the presentation layer. Django uses a plain-text template system that keeps everything that the browser renders.
- **View**: This communicates with the database via the model and transfers the data to the template for viewing.

The framework itself acts as the controller. It sends a request to the appropriate view, according to the Django **URL** configuration.

When developing any Django project, you will always work with models, views, templates, and URLs. In this chapter, you will learn how they fit together.
## The Django architecture

_Figure 1.1_ shows how Django processes requests and how the request/response cycle is managed with the different main Django components – URLs, views, models, and templates:
![[Pasted image 20240824193346.png]]

This is how Django handles HTTP requests and generates responses:

1. A web browser requests a page by its URL and the web server passes the HTTP request to Django.
2. Django runs through its configured URL patterns and stops at the first one that matches the requested URL.
3. Django executes the view that corresponds to the matched URL pattern.
4. The view potentially uses data models to retrieve information from the database.
5. Data models provide data definitions and behaviors. They are used to query the database.
6. The view renders a template (usually HTML) to display the data and returns it with an HTTP response.

We will get back to the Django request/response cycle at the end of this chapter in the _The request/response cycle_ section.

Django also includes hooks in the request/response process, which are called middleware. Middleware has been intentionally left out of this diagram for the sake of simplicity. You will use middleware in different examples of this book, and you will learn how to create custom middleware in _Chapter 17_, _Going Live_.