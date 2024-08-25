---
id: 01J63449FS1VFF7PX03GQ1TSY4
modified: 2024-08-25T12:28:27-04:00
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

## New features in Django 5

Django 5 introduces several key features that you will use in the examples of this book. This version also deprecates certain features and eliminates previously deprecated functionalities. Django 5.0 presents the following new major features:

- **Facet filters in the administration site**: Facet filters can be added now to the administration site. When enabled, facet counts are displayed for applied filters in the admin object list. This feature is presented in the _Added facet counts to filters_ section of this chapter.
- **Simplified templates for form field rendering**: Form field rendering has been simplified with the capability to define field groups with associated templates. This aims to make the process of rendering related elements of a Django form field, such as labels, widgets, help text, and errors, more streamlined. An example of using field groups can be found in the _Creating templates for the comment form_ section of _Chapter 2, Enhancing Your Blog and Adding Social Features_.
- **Database-computed default values**: Django adds database-computed default values. An example of this feature is presented in the _Adding datetime fields_ section of this chapter.
- **Database-generated model fields**: This is a new type of field that enables you to create database-generated columns. An expression is used to automatically set the field value each time the model is changed. The field value is set using the `GENERATED` `ALWAYS` SQL syntax.
- **More options for declaring model field choices**: Fields that support choices no longer require accessing the `.choices` attribute to access enumeration types. A mapping or callable instead of an iterable can be used directly to expand enumeration types. Choices with enumeration types in this book have been updated to reflect these changes. An instance of this can be found in the _Adding a status field_ section of this chapter.

Django 5 also comes with some improvements in asynchronous support**. Asynchronous Server Gateway Interface** (**ASGI**) support was first introduced in Django 3 and improved in Django 4.1 with asynchronous handlers for class-based views and an asynchronous ORM interface. Django 5 adds asynchronous functions to the authentication framework, provides support for asynchronous signal dispatching, and adds asynchronous support to multiple built-in decorators.

# Creating your first project

Your first Django project will consist of a blog application. This will offer you a solid introduction to Django’s capabilities and functionalities.

Blogging is the perfect starting point to build a complete Django project, given its wide range of required features, from basic content management to advanced functionalities like commenting, post sharing, search, and post recommendations. The blog project will be covered in the first three chapters of this book.

In this chapter, we will start by creating the Django project and a Django application for the blog. We will then create our data models and synchronize them to the database. Finally, we will create an administration site for the blog, and we will build the views, templates, and URLs.

_Figure 1.2_ shows a representation of the blog application pages that you will create:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781805125457/files/Images/B21088_01_02.png)

Figure 1.2: Diagram of functionalities built in Chapter 1

The blog application will consist of a list of posts including the post title, publishing date, author, a post excerpt, and a link to read the post. The post list page will be implemented with the `post_list` view. You will learn how to create views in this chapter.

When readers click on the link of a post in the post list page, they will be redirected to a single (detail) view of a post. The detail view will display the title, publishing date, author, and the complete post body.

Let’s start by creating the Django project for our blog. Django provides a command that allows you to create an initial project file structure.

Run the following command in your shell prompt:

```
django-admin startproject mysite
```

This will create a Django project with the name `mysite`.

Avoid naming projects after built-in Python or Django modules in order to prevent conflicts.

Let’s take a look at the generated project structure:

```
mysite/
    manage.py
    mysite/
      __init__.py
      asgi.py
      settings.py
      urls.py
      wsgi.py
```

The outer `mysite/` directory is the container for our project. It contains the following files:

- `manage.py`: This is a command-line utility used to interact with your project. You won’t usually need to edit this file.
- `mysite/`: This is the Python package for your project, which consists of the following files:
    - `__init__.py`: An empty file that tells Python to treat the `mysite` directory as a Python module.
    - `asgi.py`: This is the configuration to run your project as an ASGI application with ASGI-compatible web servers. ASGI is the emerging Python standard for asynchronous web servers and applications.
    - `settings.py`: This indicates settings and configuration for your project and contains initial default settings.
    - `urls.py`: This is the place where your URL patterns live. Each URL defined here is mapped to a view.
    - `wsgi.py`: This is the configuration to run your project as a **Web Server Gateway Interface** (**WSGI**) application with WSGI-compatible web servers.
## Applying initial database migrations

Django applications require a database to store data. The `settings.py` file contains the database configuration for your project in the `DATABASES` setting. The default configuration is a SQLite3 database. SQLite comes bundled with Python 3 and can be used in any of your Python applications. SQLite is a lightweight database that you can use with Django for development. If you plan to deploy your application in a production environment, you should use a full-featured database, such as PostgreSQL, MySQL, or Oracle. You can find more information about how to get your database running with Django at [https://docs.djangoproject.com/en/5.0/topics/install/#database-installation](https://docs.djangoproject.com/en/5.0/topics/install/#database-installation).

Your `settings.py` file also includes a list named `INSTALLED_APPS` that contains common Django applications that are added to your project by default. We will go through these applications later in the _Project settings_ section.

Django applications contain data models that are mapped to database tables. You will create your own models in the _Creating the blog data models_ section. To complete the project setup, you need to create the tables associated with the models of the default Django applications included in the `INSTALLED_APPS` setting. Django comes with a system that helps you manage database migrations.

Open the shell prompt and run the following commands:

```
cd mysite
python manage.py migrate
```

You will see an output that ends with the following lines:

```
Applying contenttypes.0001_initial... OK
Applying auth.0001_initial... OK
Applying admin.0001_initial... OK
Applying admin.0002_logentry_remove_auto_add... OK
Applying admin.0003_logentry_add_action_flag_choices... OK
Applying contenttypes.0002_remove_content_type_name... OK
Applying auth.0002_alter_permission_name_max_length... OK
Applying auth.0003_alter_user_email_max_length... OK
Applying auth.0004_alter_user_username_opts... OK
Applying auth.0005_alter_user_last_login_null... OK
Applying auth.0006_require_contenttypes_0002... OK
Applying auth.0007_alter_validators_add_error_messages... OK
Applying auth.0008_alter_user_username_max_length... OK
Applying auth.0009_alter_user_last_name_max_length... OK
Applying auth.0010_alter_group_name_max_length... OK
Applying auth.0011_update_proxy_permissions... OK
Applying auth.0012_alter_user_first_name_max_length... OK
Applying sessions.0001_initial... OK
```

The preceding lines are the database migrations that are applied by Django. By applying the initial migrations, the tables for the applications listed in the `INSTALLED_APPS` setting are created in the database.

You will learn more about the `migrate` management command in the _Creating and applying migrations_ section of this chapter.
- 🍅 (pomodoro::WORK) (duration:: 40m) (begin:: 2024-08-25 11:48) - (end:: 2024-08-25 12:28)