---
id: 01J63449FS1VFF7PX03GQ1TSY4
modified: 2024-08-25T18:59:12-04:00
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

## Running the development server

Django comes with a lightweight web server to run your code quickly, without needing to spend time configuring a production server. When you run the Django development server, it keeps checking for changes in your code. It reloads automatically, freeing you from manually reloading it after code changes. However, it might not notice some actions, such as adding new files to your project, so you will have to restart the server manually in these cases.

Start the development server by typing the following command in the shell prompt:

```
python manage.py runserver
```

You should see something like this:

```
Watching for file changes with StatReloader
Performing system checks...
System check identified no issues (0 silenced).
January 01, 2024 - 10:00:00
Django version 5.0, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Now, open `http://127.0.0.1:8000/` in your browser. You should see a page stating that the project is successfully running, as shown in _Figure 1.3_:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781805125457/files/Images/B21088_01_03.png)

Figure 1.3: The default page of the Django development server

The preceding screenshot indicates that Django is running. If you take a look at your console, you will see the `GET` request performed by your browser:

```
[01/Jan/2024 10:00:15] "GET / HTTP/1.1" 200 16351
```

Each HTTP request is logged in the console by the development server. Any error that occurs while running the development server will also appear in the console.




This server is only intended for development and is not suitable for production use. To deploy Django in a production environment, you should run it as a WSGI application using a web server, such as Apache, Gunicorn, or uWSGI, or as an ASGI application using a server such as Daphne or Uvicorn. You can find more information on how to deploy Django with different web servers at [https://docs.djangoproject.com/en/5.0/howto/deployment/wsgi/](https://docs.djangoproject.com/en/5.0/howto/deployment/wsgi/).

_Chapter 17_, _Going Live_, explains how to set up a production environment for your Django projects.

## Project settings

Let’s open the `settings.py` file and take a look at the configuration of the project. There are several settings that Django includes in this file, but these are only part of all the available Django settings. You can see all the settings and their default values at [https://docs.djangoproject.com/en/5.0/ref/settings/](https://docs.djangoproject.com/en/5.0/ref/settings/).

Let’s review some of the project settings:

- `DEBUG` is a Boolean that turns the debug mode of the project on and off. If it is set to `True`, Django will display detailed error pages when an uncaught exception is thrown by your application. When you move to a production environment, remember that you have to set it to `False`. Never deploy a site into production with `DEBUG` turned on because you will expose sensitive project-related data.
- `ALLOWED_HOSTS` is not applied while debug mode is on or when the tests are run. Once you move your site to production and set `DEBUG` to `False`, you will have to add your domain/host to this setting to allow it to serve your Django site.
- `INSTALLED_APPS` is a setting you will have to edit for all projects. This setting tells Django which applications are active for this site. By default, Django includes the following applications:
    - `django.contrib.admin`: An administration site.
    - `django.contrib.auth`: An authentication framework.
    - `django.contrib.contenttypes`: A framework for handling content types.
    - `django.contrib.sessions`: A session framework.
    - `django.contrib.messages`: A messaging framework.
    - `django.contrib.staticfiles`: A framework for managing static files, such as CSS, JavaScript files, and images.
- `MIDDLEWARE` is a list that contains middleware to be executed.
- `ROOT_URLCONF` indicates the Python module where the root URL patterns of your application are defined.
- `DATABASES` is a dictionary that contains the settings for all the databases to be used in the project. There must always be a default database. The default configuration uses a SQLite3 database.
- `LANGUAGE_CODE` defines the default language code for this Django site.
- `USE_TZ` tells Django to activate/deactivate timezone support. Django comes with support for timezone-aware datetimes. This setting is set to `True` when you create a new project using the `startproject` management command.

Don’t worry if you don’t understand much about what you’re seeing here. You will learn more about the different Django settings in the following chapters.

## Projects and applications
  
Throughout this book, you will encounter the terms **project** and **application** over and over. In Django, a project is considered a Django installation with some settings. An application is a group of models, views, templates, and URLs. Applications interact with the framework to provide specific functionalities and may be reused in various projects. You can think of a project as your website, which contains several applications, such as a blog, wiki, or forum, that can also be used by other Django projects.

_Figure 1.4_ shows the structure of a Django project:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781805125457/files/Images/B21088_01_04.png)

Figure 1.4: The Django project/application structure

## Creating an application

Let’s create our first Django application. We will build a blog application from scratch.

Run the following command in the shell prompt from the project’s root directory:

```
python manage.py startapp blog
```

This will create the basic structure of the application, which will look like this:

```
blog/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

These files are as follows:

- `__init__.py`: This is an empty file that tells Python to treat the `blog` directory as a Python module.
- `admin.py`: This is where you register models to include them in the Django administration site—using this site is optional.
- `apps.py`: This includes the main configuration of the `blog` application.
- `migrations`: This directory will contain database migrations of the application. Migrations allow Django to track your model changes and synchronize the database accordingly. This directory contains an empty `__init__.py` file.
- `models.py`: This includes the data models of your application; all Django applications need to have a `models.py` file but it can be left empty.
- `tests.py`: This is where you can add tests for your application.
- `views.py`: The logic of your application goes here; each view receives an HTTP request, processes it, and returns a response.

With the application structure ready, we can start building the data models for the blog.

# Creating the blog data models

Remember that a Python object is a collection of data and methods. Classes are the blueprint for bundling data and functionality together. Creating a new class creates a new type of object, allowing you to create instances of that type.

A Django model is a source of information about the behaviors of your data. It consists of a Python class that subclasses `django.db.models.Model`. Each model maps to a single database table, where each attribute of the class represents a database field.

When you create a model, Django will provide you with a practical API to query objects in the database easily.

We will define the database models for our blog application. Then, we will generate the database migrations for the models to create the corresponding database tables. When applying the migrations, Django will create a table for each model defined in the `models.py` file of the application.

## Creating the Post model

First, we will define a `Post` model that will allow us to store blog posts in the database.

Add the following lines to the `models.py` file of the `blog` application. The new lines are highlighted in bold:

```
from django.db import models
class Post(models.Model):
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    body = models.TextField()
    def __str__(self):
        return self.title
```

This is the data model for blog posts. Posts will have a title, a short label called `slug`, and a body. Let’s take a look at the fields of this model:

- `title`: This is the field for the post title. This is a `CharField` field that translates into a `VARCHAR` column in the SQL database.
- `slug`: This is a `SlugField` field that translates into a `VARCHAR` column in the SQL database. A slug is a short label that contains only letters, numbers, underscores, or hyphens. A post with the title _Django Reinhardt: A legend of Jazz_ could have a slug like _django-reinhardt-legend-jazz_. We will use the `slug` field to build beautiful, SEO-friendly URLs for blog posts in _Chapter 2, Enhancing Your Blog with Advanced Features_.
- `body`: This is the field for storing the body of the post. This is a `TextField` field that translates into a `TEXT` column in the SQL database.

We have also added a `__str__()` method to the model class. This is the default Python method to return a string with the human-readable representation of the object. Django will use this method to display the name of the object in many places, such as the Django administration site.

Let’s take a look at how the model and its fields will be translated into a database table and columns. The following diagram shows the `Post` model and the corresponding database table that Django will create when we synchronize the model to the database:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781805125457/files/Images/B21088_01_05.png)

Figure 1.5: Initial Post model and database table correspondence

Django will create a database column for each of the model fields: `title`, `slug`, and `body`. You can see how each field type corresponds to a database data type.

By default, Django adds an auto-incrementing primary key field to each model. The field type for this field is specified in each application configuration or globally in the `DEFAULT_AUTO_FIELD` setting. When creating an application with the `startapp` command, the default value for `DEFAULT_AUTO_FIELD` is `BigAutoField`. This is a 64-bit integer that automatically increments according to available IDs. If you don’t specify a primary key for your model, Django adds this field automatically. You can also define one of the model fields to be the primary key by setting `primary_key=True` on it.

We will expand the `Post` model with additional fields and behaviors. Once complete, we will synchronize it to the database by creating a database migration and applying it.

## Adding datetime fields

We will continue by adding different datetime fields to the `Post` model. Each post will be published at a specific date and time. Therefore, we need a field to store the publication date and time. We also want to store the date and time when the `Post` object was created and when it was last modified.

Edit the `models.py` file of the `blog` application to make it look like this; the new lines are highlighted in bold:

```
from django.db import models
from django.utils import timezone
class Post(models.Model):
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    def __str__(self):
        return self.title
```

We have added a `publish` field to the `Post` model. This is a `DateTimeField` field that translates into a `DATETIME` column in the SQL database. We will use it to store the date and time when the post was published. We use Django’s `timezone.now` method as the default value for the field. Note that we imported the `timezone` module to use this method. `timezone.now` returns the current datetime in a timezone-aware format. You can think of it as a timezone-aware version of the standard Python `datetime.now` method.

Another method to define default values for model fields is using database-computed default values. Introduced in Django 5, this feature allows you to use underlaying database functions to generate default values. For instance, the following code uses the database server’s current date and time as the default for the `publish` field:

```
from django.db import models
from django.db.models.functions import Now
class Post(models.Model):
    # ...
    publish = models.DateTimeField(db_default=Now())
```

To use database-generated default values, we use the `db_default` attribute instead of `default`. In this example, we use the `Now` database function. It serves a similar purpose to `default=timezone.now`, but instead of a Python-generated datetime, it uses the `NOW()` database function to produce the initial value. You can read more about the `db_default` attribute at [https://docs.djangoproject.com/en/5.0/ref/models/fields/#django.db.models.Field.db_default](https://docs.djangoproject.com/en/5.0/ref/models/fields/#django.db.models.Field.db_default). You can find all available database functions at [https://docs.djangoproject.com/en/5.0/ref/models/database-functions/](https://docs.djangoproject.com/en/5.0/ref/models/database-functions/).

Let’s continue with the previous version of the field:

```
class Post(models.Model):
    # ...
    publish = models.DateTimeField(default=timezone.now)
```

Edit the `models.py` file of the `blog` application and add the following lines highlighted in bold:

```
from django.db import models
from django.utils import timezone
class Post(models.Model):
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    def __str__(self):
        return self.title
```

We have added the following fields to the `Post` model:

- `created`: This is a `DateTimeField` field. We will use it to store the date and time when the post was created. By using `auto_now_add`, the date will be saved automatically when creating an object.
- `updated`: This is a `DateTimeField` field. We will use it to store the last date and time when the post was updated. By using `auto_now`, the date will be updated automatically when saving an object.

Utilizing the `auto_now_add` and `auto_now` datetime fields in your Django models is highly beneficial for tracking the creation and last modification times of objects.

## Defining a default sort order

Blog posts are typically presented in reverse chronological order, showing the newest posts first. For our model, we will define a default ordering. This ordering takes effect when retrieving objects from the database unless a specific order is indicated in the query.

Edit the `models.py` file of the `blog` application as shown below. The new lines are highlighted in bold:

```
from django.db import models
from django.utils import timezone
class Post(models.Model):
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    class Meta:
        ordering = ['-publish']
    def __str__(self):
        return self.title
```

We have added a `Meta` class inside the model. This class defines metadata for the model. We use the `ordering` attribute to tell Django that it should sort results by the `publish` field. This ordering will apply by default for database queries when no specific order is provided in the query. We indicate descending order by using a hyphen before the field name, `-publish`. Posts will be returned in reverse chronological order by default.

## Adding a database index

Let’s define a database index for the `publish` field. This will improve performance for query filtering or ordering results by this field. We expect many queries to take advantage of this index since we are using the `publish` field to order results by default.

Edit the `models.py` file of the `blog` application and make it look like this; the new lines are highlighted in bold:

```
from django.db import models
from django.utils import timezone
class Post(models.Model):
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    class Meta:
        ordering = ['-publish']
        indexes = [
            models.Index(fields=['-publish']),
        ]
    def __str__(self):
        return self.title
```

We have added the `indexes` option to the model’s `Meta` class. This option allows you to define database indexes for your model, which could comprise one or multiple fields, in ascending or descending order, or functional expressions and database functions. We have added an index for the `publish` field. We use a hyphen before the field name to define the index specifically in descending order. The creation of this index will be included in the database migrations that we will generate later for our blog models.

Index ordering is not supported on MySQL. If you use MySQL for the database, a descending index will be created as a normal index.

You can find more information about how to define indexes for models at [https://docs.djangoproject.com/en/5.0/ref/models/indexes/](https://docs.djangoproject.com/en/5.0/ref/models/indexes/).

## Activating the application

We need to activate the `blog` application in the project for Django to keep track of the application and be able to create database tables for its models.

Edit the `settings.py` file and add `blog.apps.BlogConfig` to the `INSTALLED_APPS` setting. It should look like this; the new lines are highlighted in bold:

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog.apps.BlogConfig',
]
```

The `BlogConfig` class is the application configuration. Now Django knows that the application is active for this project and will be able to load the application models.

## Adding a status field

A common functionality for blogs is to save posts as a draft until ready for publication. We will add a `status` field to our model that will allow us to manage the status of blog posts. We will be using the _Draft_ and _Published_ statuses for posts.

Edit the `models.py` file of the `blog` application to make it look as follows. The new lines are highlighted in bold:

```
from django.db import models
from django.utils import timezone
class Post(models.Model):
    class Status(models.TextChoices):
        DRAFT = 'DF', 'Draft'
        PUBLISHED = 'PB', 'Published'
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(
        max_length=2,
        choices=Status,
        default=Status.DRAFT
    )
    class Meta:
        ordering = ['-publish']
        indexes = [
            models.Index(fields=['-publish']),
        ]
    def __str__(self):
        return self.title
```

We have defined the enumeration class `Status` by subclassing `models.TextChoices`. The available choices for the post status are `DRAFT` and `PUBLISHED`. Their respective values are `DF` and `PB`, and their labels or readable names are _Draft_ and _Published_.

Django provides enumeration types that you can subclass to define choices simply. These are based on the `enum` object of Python’s standard library. You can read more about `enum` at [https://docs.python.org/3/library/enum.html](https://docs.python.org/3/library/enum.html).

Django enumeration types present some modifications over `enum`. You can learn about those differences at [https://docs.djangoproject.com/en/5.0/ref/models/fields/#enumeration-types](https://docs.djangoproject.com/en/5.0/ref/models/fields/#enumeration-types).

We can access `Post.Status.choices` to obtain the available choices, `Post.Status.names` to obtain the names of the choices, `Post.Status.labels` to obtain the human-readable names, and `Post.Status.values` to obtain the actual values of the choices.

We have also added a new `status` field to the model that is an instance of `CharField`. It includes a `choices` parameter to limit the value of the field to the choices in `Status`. We have also set a default value for the field using the `default` parameter. We use `DRAFT` as the default choice for this field.

It’s a good practice to define choices inside the model class and use the enumeration types. This will allow you to easily reference choice labels, values, or names from anywhere in your code. You can import the `Post` model and use `Post.Status.DRAFT` as a reference for the _Draft_ status anywhere in your code.

Let’s take a look at how to interact with status choices.

Run the following command in the shell prompt to open the Python shell:

```
python manage.py shell
```

Then, type the following lines:

```
>>> from blog.models import Post
>>> Post.Status.choices
```

You will obtain the `enum` choices with value-label pairs, like this:

```
[('DF', 'Draft'), ('PB', 'Published')]
```

Type the following line:

```
>>> Post.Status.labels
```

You will get the human-readable names of the `enum` members, as follows:

```
['Draft', 'Published']
```

Type the following line:

```
>>> Post.Status.values
```

You will get the values of the `enum` members, as follows. These are the values that can be stored in the database for the `status` field:

```
['DF', 'PB']
```

Type the following line:

```
>>> Post.Status.names
```

You will get the names of the choices, like this:

```
['DRAFT', 'PUBLISHED']
```

You can access a specific lookup enumeration member with `Post.Status.PUBLISHED` and you can access its `.name` and `.value` properties as well.

## Adding a many-to-one relationship

Posts are always written by an author. We will create a relationship between users and posts that will indicate which user wrote which posts. Django comes with an authentication framework that handles user accounts. The Django authentication framework comes in the `django.contrib.auth` package and contains a `User` model. To define the relationship between users and posts, we will use the `AUTH_USER_MODEL` setting, which points to `auth.User` by default. This setting allows you to specify a different user model for your project.

Edit the `models.py` file of the `blog` application to make it look as follows. The new lines are highlighted in bold:

```
from django.conf import settings
from django.db import models
from django.utils import timezone
class Post(models.Model):
    class Status(models.TextChoices):
        DRAFT = 'DF', 'Draft'
        PUBLISHED = 'PB', 'Published'
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250)
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='blog_posts'
    )
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(
        max_length=2,
        choices=Status,
        default=Status.DRAFT
    )
    class Meta:
        ordering = ['-publish']
        indexes = [
            models.Index(fields=['-publish']),
        ]
    def __str__(self):
        return self.title
```

We have imported the project’s settings and we have added an `author` field to the `Post` model. This field defines a many-to-one relationship with the default user model, meaning that each post is written by a user, and a user can write any number of posts. For this field, Django will create a foreign key in the database using the primary key of the related model.

The `on_delete` parameter specifies the behavior to adopt when the referenced object is deleted. This is not specific to Django; it is a SQL standard. Using `CASCADE`, you specify that when the referenced user is deleted, the database will also delete all related blog posts. You can take a look at all the possible options at [https://docs.djangoproject.com/en/5.0/ref/models/fields/#django.db.models.ForeignKey.on_delete](https://docs.djangoproject.com/en/5.0/ref/models/fields/#django.db.models.ForeignKey.on_delete).

We use `related_name` to specify the name of the reverse relationship, from `User` to `Post`. This will allow us to access related objects easily from a user object by using the `user.blog_posts` notation. We will learn more about this later.

Django comes with different types of fields that you can use to define your models. You can find all field types at [https://docs.djangoproject.com/en/5.0/ref/models/fields/](https://docs.djangoproject.com/en/5.0/ref/models/fields/).

The `Post` model is now complete, and we can now synchronize it to the database.

## Creating and applying migrations

Now that we have a data model for blog posts, we need to create the corresponding database table. Django comes with a migration system that tracks the changes made to models and enables them to propagate into the database.

The `migrate` command applies migrations for all applications listed in `INSTALLED_APPS`. It synchronizes the database with the current models and existing migrations.

First, we will need to create an initial migration for our `Post` model.

Run the following command in the shell prompt from the root directory of your project:

```
python manage.py makemigrations blog
```

You should get an output similar to the following one:

```
Migrations for 'blog':
    blog/migrations/0001_initial.py
        - Create model Post
        - Create index blog_post_publish_bb7600_idx on field(s)
          -publish of model post
```

Django just created the `0001_initial.py` file inside the `migrations` directory of the `blog` application. This migration contains the SQL statements to create the database table for the `Post` model and the definition of the database index for the `publish` field.

You can take a look at the file contents to see how the migration is defined. A migration specifies dependencies on other migrations and operations to perform in the database to synchronize it with model changes.

Let’s take a look at the SQL code that Django will execute in the database to create the table for your model. The `sqlmigrate` command takes the migration names and returns their SQL without executing it.

Run the following command from the shell prompt to inspect the SQL output of your first migration:

```
python manage.py sqlmigrate blog 0001
```

The output should look as follows:

```
BEGIN;
--
-- Create model Post
--
CREATE TABLE "blog_post" (
  "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
  "title" varchar(250) NOT NULL,
  "slug" varchar(250) NOT NULL,
  "body" text NOT NULL,
  "publish" datetime NOT NULL,
  "created" datetime NOT NULL,
  "updated" datetime NOT NULL,
  "status" varchar(10) NOT NULL,
  "author_id" integer NOT NULL REFERENCES "auth_user" ("id") DEFERRABLE INITIALLY DEFERRED);
--
-- Create blog_post_publish_bb7600_idx on field(s) -publish of model post
--
CREATE INDEX "blog_post_publish_bb7600_idx" ON "blog_post" ("publish" DESC);
CREATE INDEX "blog_post_slug_b95473f2" ON "blog_post" ("slug");
CREATE INDEX "blog_post_author_id_dd7a8485" ON "blog_post" ("author_id");
COMMIT;
```

The exact output depends on the database you are using. The preceding output is generated for SQLite. As you can see in the output, Django generates the table names by combining the application name and the lowercase name of the model (`blog_post`), but you can also specify a custom database name for your model in the `Meta` class of the model using the `db_table` attribute.

Django creates an auto-incremental `id` column that is used as the primary key for each model, but you can also override this by specifying `primary_key=True` on one of your model fields. The default `id` column consists of an integer that is incremented automatically. This column corresponds to the `id` field that is automatically added to your model.

The following three database indexes are created:

- An index in descending order on the `publish` column. This is the index we explicitly defined with the `indexes` option of the model’s `Meta` class.
- An index on the `slug` column because `SlugField` fields imply an index by default.
- An index on the `author_id` column because `ForeignKey` fields imply an index by default.

Let’s compare the `Post` model with its corresponding database `blog_post` table:

![Table
Description automatically generated with medium confidence](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781805125457/files/Images/B21088_01_06.png)

Figure 1.6: Complete Post model and database table correspondence

_Figure 1.6_ shows how the model fields correspond to database table columns.

Let’s sync the database with the new model.

Execute the following command in the shell prompt to apply the existing migrations:

```
python manage.py migrate
```

You will get an output that ends with the following line:

```
Applying blog.0001_initial... OK
```

We just applied migrations for the applications listed in `INSTALLED_APPS`, including the `blog` application. After applying the migrations, the database reflects the current status of the models.

If you edit the `models.py` file in order to add, remove, or change the fields of existing models, or if you add new models, you will have to create a new migration using the `makemigrations` command. Each migration allows Django to keep track of model changes. Then, you will have to apply the migration using the `migrate` command to keep the database in sync with your models.