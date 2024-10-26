---
id: 01JB594AX158AV24B132E27N0S
modified: 2024-10-26T16:44:56-04:00
title: Chapter 2 - Live in 30 minutes - You now have a website
tags:
  - hugo
  - jamstack
  - programming
  - books
---
# 2 Live in 30 minutes: You now have a website

This chapter covers

- Running the Hugo command line
- Setting up a Hugo website with themes and content
- Outlining the structure of a Hugo-based website
- Setting up a continuous deployment pipeline
- Measuring performance and analyzing website maintainability

Hugo is quick and easy to get started with. You can download Hugo and get going using just a basic text editor and a web browser. This chapter navigates through the entire length of Jamstack’s flow as figure 2.1 illustrates. We will create a website for a company named Acme Corporation. Acme Corporation is a leading manufacturer of shapes like lines, circles, squares, and triangles in digital form. We will use the Hugo command line to bootstrap the website (section 2.1) with a prebuilt theme (section 2.2) and some ready-to-use content (section 2.3). We will also host the website on the internet (section 2.4) and analyze the decisions made in this chapter for performance and maintainability (section 2.5). Note that we will enhance this website throughout the book.

![CH02_F01_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F01_Jain.png)  

Figure 2.1 This chapter runs through the entire flow of the Jamstack—from the developer to the published website.

Appendix A provides the information to get up and running with Hugo. You can also use the official website at [https://gohugo.io/](https://gohugo.io/) to download Hugo as well as to refer to its documentation. Hugo is available on all major platforms. For this book, you need Hugo with a version greater than or equal to 0.91.2.

## 2.1 Your first Hugo website

Hugo offers an extensive command line that exposes all of its functionality, including bootstrapping a new website. This section introduces you to Hugo’s command line.

### 2.1.1 The Hugo command line

Hugo is a command-line tool that’s well designed and provides all of Hugo’s functionality. It helps by migrating data, creating placeholders, and analyzing performance, along with the core task of building your website. The Hugo command line has two distinct parts:

- _Commands_—Determine tasks that you want Hugo to do. You can supply commands and subcommands by using hugo [command] on the command line. Hugo’s commands are hierarchical. A plain hugo call runs the default command to build the site. Issue hugo new to create new things. The default for hugo new creates new content pages. You can use hugo new site to build a site skeleton, and hugo new theme to generate a theme.
    
- _Flags (also called command-line parameters)_—Specify options that modify the result of the command by providing a different configuration. Flags are specific to the command, and each command can have independent flags. For example, --format yaml in the new site command changes the metadata format from the default TOML to YAML.
    

An intuitive way to learn the Hugo command line is to use the --help flag. Help with Hugo is hierarchical: hugo --help provides help for the hugo command and lists hugo new as a subcommand; hugo new --help provides documentation for the new command and mentions site as a subcommand. Hugo’s help also shows all the flags available for each command. You can also generate the Hugo command-line documentation in the man pages format (as used by the man command in UNIX-based operating systems). For this, use hugo gen man or use hugo gen doc for Markdown files.

Let’s see how all this fits together by creating our first website. To create a new website in Hugo, we’ll use the command in the following listing.

Listing 2.1 Hugo command to create a new website

hugo new site **acme-corporation** --format **yaml**

This command creates the Hugo skeleton folder structure with YAML as the metadata language in a subfolder called acme-corporation in the current folder. The various parts of this command are labeled in figure 2.2. Note that we’ll use YAML ([https://yaml.org/](https://yaml.org/)) instead of the default TOML ([https://toml.io](https://toml.io/)) metadata language for this book. YAML is more prevalent in the general programming community, less verbose than TOML, and GitHub has better support for it. It is an easier language to get started with and a better choice for users new to the entire Hugo ecosystem. We will discuss YAML as a metadata language for Hugo in chapter 3. Appendix B discusses TOML as the metadata language option. Note that the official Hugo documentation provides metadata in all supported languages.

![CH02_F02_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F02_Jain.png)  

Figure 2.2 The hugo command provides access to the Hugo command line. We can use all of Hugo’s functionality via this command line. You can use it to compile Hugo websites, run the development server, measure build performance, and access modules.

Exercise 2.1

Which of the following allow you to get help on Hugo?

1. --help flag
    
2. man command
    
3. Hugo website
    
4. All of the above
    

### 2.1.2 Adding to source control

The first step in any project is to commit the changes to a version control repository. The command-line interface does not have native undo/redo support. If you accidentally delete a file, it does not go to the recycle bin or to the trash folders. Any running script has the potential to cause data loss, including the hugo command. There is no turning back unless you have versioned the source code.

Version control systems allow for recovering deleted files and reverting to older versions. The version control system used in this book is Git. Git is the most popular system, and GitHub has tight integration for it. This also includes GitHub Pages, the most popular host for static websites on the internet. It is a good idea to commit each checkpoint to version control. You can use the git command or a GUI client like SourceTree or Fork to perform these tasks. On the command line, you can perform this using multiple Git commands as the following listing shows. To help with version control, take note of the code checkpoints where you can pause to check your code.

Listing 2.2 Git commands to create a new repository

cd **acme-corporation** 
git init **.**                                 ❶
  
git add *****                                  ❷
  
git commit -m **"Create website skeleton"**    ❸

❶ Creates an empty Git repository

❷ Adds the files that we just created using the hugo command

❸ Checks in the files to version control with a commit message

Even though we created a website skeleton, that does not mean we have a working website. Most of the skeleton folders created by the hugo command are empty. At the bare minimum, we need to provide some content and a theme to render it on our website.

Code checkpoint [https://github.com/hugoinaction/hugoinaction/tree/chapter-02-01](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-01).

Note Extra files (Readme.md, License.md, and .gitignore) were added to the repository on the server for better GitHub support.

Migrating to Hugo

Hugo supports importing content from Jekyll and automatically converts content from that format to a format that Hugo understands. You can use the hugo import jekyll <source jekyll folder> <target hugo folder> command to import the folder-equivalent content from Jekyll into a Hugo website. This command does not provide synchronization, but we can use it for a one-time import.

### 2.1.3 Structure of the Hugo source folder

Before adding a theme or some content, let’s look at what makes up a Hugo website. A Hugo source folder is more than templates and content. The hugo new command generates six folders, and we will create more as we use Hugo’s features. The critical folders in our website, as figure 2.3 shows, include the following:

- _archetypes_—Contains the templates for the content files. Hugo tries to minimize the copy and paste work needed to create content. We can create templates for Markdown files or folders in this folder, and Hugo uses them to create a basic content file. We will get to archetypes in chapter 5.
    
- _content_—Contains all the content that traditionally goes into the database. We can organize the content into files and folders as we desire. By default, Hugo generates the website output directly, based on this folder’s structure, although we can override that using the metadata in each file (called _front matter_, which we will discuss in chapter 3). We will work with the content folder throughout the book.
    
- _data_—Stores structured content in the form of YAML, TOML, CSV, or JSON files, which are made available as global variables throughout the website. A traditional database houses more than just web page content. There can be tables associated with structured data, which have no place in the content folder, so this folder comes in handy when we generate content from outside of Hugo and pass that information in as a JSON or a CSV file for Hugo to consume. We will read from the data folder in chapter 5.
    
- _layouts_—Overrides parts of the theme. Hugo gives us the flexibility to mix and match pages from themes and to write our own custom pages. In this folder, all customization of the theme occurs. We can use this directory to store these overridden theme layouts. The line between a theme and layout is blurred, and Hugo gives us total flexibility to create a theme slowly by overriding pages one by one. We will use the layouts folder to update the home page in this chapter and go into layouts in detail in chapters 6 and 7.
    
- _themes_—Contains the code that we use to make the content in the content folder presentable. We can use the Go template language to write themes. We will add themes in this chapter and create our own in chapter 7.
    
- _config_—Houses the website’s configuration. This directory contains the metadata shared across the website, including the theme’s name and any parameters that need to be passed to Hugo or to the theme to render content. By default, Hugo creates a single config.yaml file. Hugo supports splitting this configuration file into multiple files and having different environments for testing and production. That turns the configuration into a folder. We will go into the configuration in detail in chapter 4.
    
- _static_—Stores static content like fonts or PDF files. Hugo copies this content as is to the output directory. This folder is somewhat equivalent to the Apache/Nginx web server root folder, where you can place any HTML file for rendering. It is advisable to put as much content as possible in the content, data, themes, and layouts folders to have programmatic access to it and to benefit from Hugo’s render pipeline. In the static folder, we can store binaries files like .pdf, .woff (for web fonts), and .zip files for downloadable content that does not belong anywhere else. We will put some files in the static folder in this chapter.
    

![CH02_F03_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F03_Jain.png)  

Figure 2.3 The website source code and content in Hugo lies in the source folder. The hugo new command creates a basic set of folders, which designates the various parts of a Hugo website: the archetypes folder (for the content templates), the content folder (for the textual content), the data folder (for structured content, and key-value pairs), the layouts and themes folders (for templates and individual page designs), and the static folder (for additional content that needs to be hosted but does not fit into any other category). Other folders and files that show up during usage include assets (for unprocessed images and JS/CSS files), config (for settings and metadata, initially generated as a single file), resources (for caching processed assets), public (to hold the output), vendor and go.sum/mod (for Hugo Modules), package*.json and node_modules (for JavaScript), .github/netlify.toml (for continuous integration), and api (for custom first-party APIs).

Of all these folders, the content folder is where we usually spend the most time adding content to the website. The themes folder contains the theme that the developer can manage outside the website. In contrast, we change the other folders (except for the data folder for the data-driven web pages) infrequently, only when something significant needs to be added.

Exercise 2.2

Which of the following folders contains the text displayed on a web page?

1. markup
    
2. markdown
    
3. content
    
4. data
    
5. text
    

When building your Hugo-based website, here are some other files and folders that you will encounter:

- _assets folder_—Places images, JavaScript, and CSS files as unprocessed source code to be consumed globally from the website. This folder allows us to process these files during compilation. Hugo can resize images, bundle and minify JavaScript files, and convert SCSS to CSS via its asset pipeline (Hugo Pipes). We will learn about image manipulation and asset bundling in chapter 6 and work with JavaScript assets in chapter 10.
    
- _public folder_—Hugo’s default output directory, where the hugo command generates the HTML output to be deployed and cached at the CDN.
    
- _resources folder_—When processing data, Hugo caches the results of heavy operations in this folder. We should put this folder into our version control and reuse its data across builds. This folder is one of the critical ingredients for getting outstanding performance with Hugo. Processing images is a CPU-intensive operation and takes time. Most assets don’t change across builds, and caching the processed images for as long as they do not change provides Hugo with a significant performance boost.
    
- _go.mod and go.sum files_—Hugo Modules uses these files to synchronize project dependencies. We rarely look into these files, but we do need to put these files in version control. We will introduce these files in chapter 8.
    
- _vendor folder_—Stores third-party dependencies that we can include via Hugo Modules. We will create this folder while working with Hugo Modules in chapter 8.
    
- _node__modules, package.json, package-lock.json, and package.hugo.json files_—Associates and integrates Hugo with the JavaScript ecosystem. We will discuss using JavaScript with Hugo in detail in chapter 10.
    
- _.github folder and netlify.toml files_—Associates Hugo with the continuous integration services GitHub and Netlify. We will use these services throughout the book.
    
- _api folder_—Although not standard, we’ll create this folder to house custom APIs in chapter 11.
    

Exercise 2.3

Match the file type to the most likely folder to place the file.

 

1. YAML             a. assets

2. Markdown     b. static

3. PDF                c. content

4. HTML             d. config

5. CSS                e. themes

## 2.2 Adding a theme

Coming back to Acme Corporation’s sample website, before that website can see the light of the day, it needs a theme and some content. A _theme_ in Hugo represents all the logic that converts markup documents into presentable web pages. It consists of template code, JavaScript, and CSS assets and images used for common elements like icons and backgrounds. Creating a Hugo theme is time-consuming, and it is a good idea to try out some prebuilt themes to begin with.

Note If you plan to use a theme created by someone else, you may not need to learn the Go template language to use Hugo.

You can create a website by learning a markup language like Markdown and a metadata language like YAML. You can always modify the theme to customize the UI, but if you want to get a website up and focus on the content, you only need to know a content markup and a metadata language. For Acme Corporation, we will start with a prebuilt theme that’s ready to use. There are multiple ways to get a theme:

- _Use Hugo Modules to integrate the theme._ Hugo Modules is Hugo’s package management system that allows themes to have dependencies. Hugo can automatically fetch dependencies required by a theme when building your site using Hugo Modules. Themes with dependencies will not work with other integration methods. Hugo Modules have setup requirements that we will discuss when introducing it in chapter 8.
    
- _Use Git Submodules to reference the theme in the themes folder._ The Git version management system can set this up for you. This allows one Git repository to include another repository as a module within it. The dependencies can be linked to another server location and built independently. While the submodule feature is a part of Git and needs no separate installation, it still needs to be set up.
    
- This feature is less potent than Hugo Modules. Theme authors who have not updated their themes to support Hugo Modules mention Git Modules as the integration method for their theme. However, over time, the use of Git Submodules will diminish in the Hugo world, and we do not recommend using it in newer themes and websites.
    
- _Download and copy the theme to the themes folder._ The download-and-copy approach is the simplest of techniques. Because the theme code is available locally, we can easily read it to understand what the theme is doing, modify it, and view our website’s updates. When developing a new theme, this approach allows for making changes quickly and saves us from the overhead of managing different repositories. To simplify getting started, we will use the download-and-copy approach for the book’s first seven chapters.
    

### 2.2.1 Adding a theme to the website

We can find themes on the Hugo website at [https://themes.gohugo.io/](https://themes.gohugo.io/). While most themes work with the download-and-copy approach, some may have dependencies for which Hugo Modules are necessary. We will use the Eclectic theme, which has no such requirements. A copy of the Eclectic theme is provided in the code samples accompanying this book ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/01](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/01)). It is also available at [https://github.com/hugoinaction/Eclectic](https://github.com/hugoinaction/Eclectic).

We need to download and paste the Eclectic folder into the themes folder for our website for it to be made available. The files are present in the proper subfolder so that you can place them in the root folder of the website. Each listing comes with the path to the file and the filename where the changes need to be made. For loading Eclectic as the theme for our website, we need to specify it in the website configuration file using the theme key. Listing 2.3 tells Hugo to look for a folder named Eclectic in the themes folder and to load the theme from that folder.

Note In the chapter resources throughout the book, the files are provided in the proper relative paths from the website root and need to be placed in the exact same relative location for your website.

Listing 2.3 Updating the theme in the config file (config.yaml)

...                ❶
**theme:** Eclectic

❶ Existing data that’s generated by the hugo new command.

Code checkpoint [https://chapter-02-02.hugoinaction.com](https://chapter-02-02.hugoinaction.com/), and source code: [https://github.com/hugoinaction/hugoinaction/tree/chapter-02-02](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-02).

Note You can compare various GitHub branches by navigating to [https://github.com/hugoinaction/hugoinaction/compare/chapter-02-01..chapter-02-02](https://github.com/hugoinaction/hugoinaction/compare/chapter-02-01..chapter-02-02), where chapter-02-01 and chapter-02-02 are branch names. The Readme file at [https://github.com/hugoinaction/hugoinaction](https://github.com/hugoinaction/hugoinaction) provides every code checkpoint (along with its respective section), a link to the hosted version, and the diff from the previous code checkpoint. It is a good idea to view the hosted version of a code checkpoint before reading the corresponding section of this book.

### 2.2.2 Running the dev server

We can run our Acme website in development mode using the command hugo server on the command line (we could also use hugo serve). This command creates a development server that provides local content. The development server mode compiles the code automatically when changed. It has near real-time updates to the website’s locally-hosted version (popularly called _live reload_) with content changes. The default port (the location in the machine where we can find the website) for Hugo is 1313, and unless something else is running at that port (in which case, it can be changed by --port <number> flag), the development mode website should be available there. You can open http://localhost:1313 in your browser to find the default website as figure 2.4 shows. The default home page is unique to most themes but needs configuration to be used.

![CH02_F04_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F04_Jain.png)  

Figure 2.4 Default website with the Eclectic theme. When we chose the Eclectic theme for a Hugo-based website, Hugo created an index page based on that theme, which the website can render even if we provide no content for the page. (It will look better when we configure the page, but it still works without anything.) This page can be used as a starting point to develop the rest of the website. (Background image by theglassdesk on Pixabay.)

Listing 2.4 shows how we can run the Hugo development server by using the hugo server command. This command hosts the Hugo-based website locally at http://localhost:1313/ by default. It automatically rebuilds the server as the content changes so that we can view it in the web browser.

Listing 2.4 Running the Hugo development server

> hugo **server** 
Start building sites ...
  
WARN 2021/04/06 22:51:34 Page.Hugo is deprecated 
and will be removed in a future release. Use the 
global hugo function.                                    ❶
  
                   | EN
-------------------+-----
  Pages            |  7                                  ❷
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  3
  Processed images |  2
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0
  
Built in 78 ms                                           ❸
Watching for changes in acme-corporation/ 
➥ **{archetypes,content,data,layouts,static,themes}**       ❹
Watching for config changes in 
➥ acme-corporation/**config.yaml**                          ❹
Environment: **"development"**                               ❺
Serving pages from **memory**                                ❻
Running in Fast Render Mode. For full rebuilds on 
➥ change: **hugo server --disableFastRender**               ❼
Web Server is available at http://localhost:1313/ 
➥ (bind address 127.0.0.1)                              ❽
Press **Ctrl+C** to stop                                     ❽

❶ Displays warnings when you use any deprecated feature. This output also shows how to fix this. (This warning may not be present when you run this command.)

❷ Indicates the number of pages Hugo has compiled

❸ Gives the compilation time

❹ Places where changes will cause Hugo to rebuild automatically

❺ Configuration environment Hugo uses for this compilation

❻ The output is not updated in the public folder.

❼ Updates web pages only if there are changes and they are actively being requested by a web browser

❽ Information on the dev server and where we can preview the website

If we run hugo without additional arguments, Hugo compiles the entire website and places the files in the public folder. We also refer to development mode as _server mode_ or _live reload mode_. It listens to changes in the filesystem and rebuilds the website with the update. Hugo also supports fast rendering in development mode, which involves building only the page requested on demand. Because Hugo is blazingly fast, we don’t notice the delay in rebuilding the web page. We can disable fast rendering or live reload if it interferes with the JavaScript state by using the command-line flags --disableFastRender and --disableLiveReload, respectively. Note that you can run the website’s production version in development mode using the --environment command-line flag. Chapter 4 discusses the difference between the various build environments.

There is no need to quit the Hugo development server through most of this book as it supports live reload so we can easily switch content. But you are free to abort it at any time by pressing Ctrl-C and running the hugo server command again.

Tip The Hugo development server optimizes for refreshes with content changes. Theme changes affecting multiple files are error-prone when reloading. If you change a theme’s contents, it’s possible that caching in the browser or incrementally building with the development server will get in the way of viewing updates. Restart the dev server, clear the browser cache, and use hugo server --noHTTPCache --disableFastRender to help in these cases.

Exercise 2.4

The default port for Hugo is _________.

When you run the website in development mode for the first time, the images provided by the theme and its JavaScript and CSS files are optimized by Hugo and cached in the resources folder we discussed earlier in this chapter. This process may cause a slower build. It is OK to commit the resources folder to source control to prevent Hugo from generating it again.

Note Most Hugo themes need some configuration and content to be functional. You might get a blank screen if you try replacing Eclectic with a different theme and have not provided Hugo with the appropriate configuration.

## 2.3 Adding content

We will convert the empty page generated with the Eclectic theme into a fully functional website. This conversion includes configuring the theme by providing it with some settings and metadata, adding pages like the privacy policy and terms of use, and overriding the theme’s landing page with a custom version.

Note Custom data for a theme is not portable. You will have to look at the theme’s documentation to figure out the theme-specific configuration. If you are still judging the theme as you develop the website, it is recommended to focus first on the standard template-based content pages (like posts) rather than the unique pages (like the landing page and Contact Us).

Exercise 2.5

The ____________ command runs Hugo in development mode.

### 2.3.1 Configuration

The fact that the website ran so well with two lines of code is the magic of the well-thought-out defaults in Hugo. We can do better by passing it the right options for our website. The configuration file has two distinct parts: the top-level configuration, which is common across themes, and the theme-specific params section, which differs across themes. Let’s add some data to the configuration file, config.yaml. This is needed to be successful with the Eclectic template in Hugo. These changes provide the information to fill up the menus, the footers, copyright notices, and the title and author information, per the requirements of the Eclectic theme for Acme Corporation.

The updated configuration file is present in the chapter 2 resources folder that accompanies this book ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/02](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/02)). You do not need to understand the entire file yet. We will be working with these settings in the following chapters, where they will become clearer. Listing 2.5 shows the configuration file that we’ll use for the Acme Corporation website. A typical Hugo configuration file contains:

- Configuration options that are standard across all themes (such as the URL of the website, its name, and language)
    
- Options for specific Hugo features (like menu)
    
- Theme-specific parameters (like params)
    

Listing 2.5 Setting up Acme Corporation’s configuration file (config.yaml)

**baseURL:** http://example.org/                   ❶
**languageCode:** en-us                            ❷
**title:** Acme Corporation                        ❸
**theme:** Eclectic                                ❹
**author:**                                        ❺
  **facebook:** " https://facebook.com/example"    ❺
  **twitter:** " https://twitter.com/example"      ❺
  **email:** "contact@example.org"                 ❺
  **name:** "Acme Corporation"                     ❺
  **location:** New York                           ❺
  **phone:** (999) 999-9999                        ❺
  **hours:** "**Mon-Fri:** 9:00AM - 6:00PM, ET"        ❺
  
**menu:** 
  **main:**                                        ❻
    - **identifier:** about                        ❻
      **name:** About                              ❻
      **url:** /about                              ❻
      **weight:** 100                              ❻
    - **identifier:** contact                      ❻
      **name:** Contact                            ❻
      **url:** /contact                            ❻
      **weight:** 200                              ❻
**params:**                                        ❼
  **color:** "#4f46e5" 
  **copyright:** "Copyright &copy; 2022 Acme Corporation. 
    All Rights Reserved." 
  **footer:** 
    - **title:** About 
      **content:**  > 
        Acme Corporation is the world's leading 
        manufacturer of digital shapes. From squares and 
        circles to triangles and hexagons, we have it 
        all. Browse through our collection of various 
        forms with different thicknesses and line styles. 
        We shape the world. You live in it. 
  
  
    - **title:** Recent Blog Posts 
      **recents:** blog 
      **recentCount:** 7 
    - **title:** Contact Us 
      **contact:** true 

❶ URL of the website. Change this to your website location.

❷ Website language. Hugo supports multilingual websites so we should use this option when building for only one language.

❸ Name of the website

❹ Name of the folder that contains the theme

❺ Author section in Hugo, a top-level section that applies to all themes. If there is one author, this is the right place to provide author information.

❻ Main menu of the website

❼ The theme parameters

Hugo supports multiple authors via a feature called _taxonomies_ (discussed in section 4.4). Hugo also provides a standard way to define menus. The menu section in the configuration file has keys, each of which specifies a menu name. Each menu has a list of entries, which can have a unique identifier, a name to display, a URL, and a weight to sort menu items. In the configuration file, the params section is theme-specific; its contents can differ across themes.

We wrote the configuration file in listing 2.5 in the YAML metadata language, which we will discuss in chapter 3. It provides structured information using keys and values separated by colons. YAML is human-readable and case-sensitive, but changes in spacing can cause problems with the YAML parser.

Hugo also supports the more “spacing-friendly” TOML format. The resources with this book also contain the TOML version of the configuration file. If you use that as an alternative, config.yaml should be removed.

Note Update the actual baseURL of the website instead of http://example.org/ in the configuration file before publishing. Leaving the file example .org breaks absolute links in the website.

The Eclectic theme allows us to provide our logo and even control the website background image by placing these in the assets/image folder (not in the themes/Eclectic/ assets/image folder). We will place logo.svg and background.svg in this folder to personalize the website. We will need to create this folder if it does not exist. (You may need to restart your development server for the changes to take effect.) These files are present in the code bundle for chapter 2 ([https://github.com/hugoinaction/hugoinaction/ tree/chapter-02-resources/03](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/03)).

Code checkpoint [https://chapter-02-03.hugoinaction.com, and source](https://chapter-02-03.hugoinaction.com/) code: [https://github.com/hugoinaction/hugoinaction/tree/chapter-02-03.](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-03) ↻ Restart your dev server.

Hugo standardizes some previously specified parts (like menu and title) in the configuration file. We will cover those parts in chapter 4. Other parts (like params) are different for each theme. Even image locations like that of the logo.svg are theme-specific.

Exercise 2.6

Which of the following is used to provide the website endpoint for Hugo to compile?

1. baseURL
    
2. endpoint
    
3. website
    
4. url
    
5. host
    
6. domain
    
7. server
    

You can see the impact of providing the metadata on the Acme website instantly. With the configuration mentioned previously, the site should look similar to figure 2.5.

![CH02_F05_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F05_Jain.png)  

Figure 2.5 The Acme Corporation website looks much more complete after configuring the theme. Now the main menu and the footer section added in the configuration file are available on all pages. The logo and background images are updated as soon as we place the image files. (Abstract vector created by BiZkettE1 at [www.freepik.com](https://www.freepik.com/).)

### 2.3.2 Content pages

A website’s objective is to serve content, and we have none on our beautiful website so far. The entries added at the top menu of the website link to pages that do not exist! We need to create pages on the website to make it functional. We will begin by adding content to the pages linked to in the menu in this chapter and then will format the content in chapter 3.

We can create content pages as text or markup files in the content folder. We can place a privacy.md file in that folder with Markdown-based content to get the https:// localhost:1313/privacy URL. Similarly, we can add the about.md, credits.md, terms.md, and contact.md pages ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/04](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/04)). Hugo automatically applies the theme, and the page should render as soon as you add the document. This way, we can add as many pages as we desire to generate the website’s core structure. Markdown provides a variety of formatting options that we will study in chapter 3.

### 2.3.3 Index page

The _index page_ (also called the _home page_ or the _landing page_) is the first page of the website and is responsible for orienting the user on what to expect. Its content is unique and different than all other pages. A text-based content works well for some pages, but many websites implement custom content for the index page. Websites even have tailor-made carousels and sections with extensive imagery that would need a custom implementation. Hugo recognizes this and provides a unique template for the index page, which is called the _index template_. In many themes, the index template is customized in a theme-specific way, and the index page configuration is not portable across themes.

Note Most Hugo themes provide a folder called exampleSite, which contains a starter website using that theme. This folder is extremely useful in exploring theme-specific configurations and customization options.

Hugo’s templates are HTML files, but these can be in any text-based file format (for example, JSON, XML, or even plain text), with additional template tags that participate in the compilation step. For users trying to build custom Hugo templates, it is a good idea to start with the index template because it impacts only one page of the website. Hugo templates can be overridden using the layouts folder. In this chapter, we will not be using any template tags and will start with a plain HTML template that we will place as layouts/index.html. It is still a Hugo template and has access to all the variables, which are optional.

For Acme Corporation’s index page, we will override the theme’s index page with a custom page, hardcoded in HTML and CSS, as figure 2.6 shows. This page will contain the website logo, title, subtitle, a button with a call to action (telling the reader to explore more), and a footer with links to additional pages.

![CH02_F06_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F06_Jain.png)  

Figure 2.6 We can create a custom landing page in a Hugo website by placing a file called index.html in the layouts folder. This page overrides the home page provided by the theme. For Acme Corporation, we used a landing page with hardcoded HTML and CSS and eschewed the theme-specific features of Eclectic to create pages based on structured data.

In the layouts folder, we will place a new file named index.html with custom HTML content ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/05](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/05)). Because we are not using Hugo’s template language, we will be hardcoding all paths and using relative locations to various support-hosting locations.

We can override templates in a Hugo theme by placing an HTML template file in the layouts folder as listing 2.6 demonstrates. Doing that provides someone who understands HTML with a quick way to customize a website without learning Hugo. Custom HTML can be unique to a particular website. Until we use Hugo’s template language, we have to be careful with the HTML we are writing as the custom HTML page does not change automatically when the content it links to changes.

Listing 2.6 Overriding Hugo’s themes (layouts/index.html)

<!DOCTYPE html> 
**<html** lang_=_"en"**>** 
  **<head>** 
    **<meta** charset_=_"UTF-8"**>** 
    **<meta** name_=_"description" content_=_"Welcome to the 
      website of Acme Corporation, the leading creator
      of digital shapes on the planet, providing
      precise shape creations that are ready to use."**>** 
    **<meta** name_=_"viewport" content_=_"width=device-width, 
      initial-scale=1.0" **/>** 
    **<link** rel_=_"stylesheet" href_=_"./index.css"**>**             ❶
    **<title>**Acme Corporation**</title>** 
  **</head>** 
  **<body** class_=_"home"**>** 
    **<section>** 
      **<img** src_=_"./image/logo.svg" alt_=_"Acme Logo" 
        width_=_"64"**/>**                                       ❷
      **<h1>**Acme Corporation**</h1>** 
      **<h2>**Shaping the world for you to live in**</h2>** 
      **<a** href_=_"./blog"**>**Explore**</a>** 
    **</section>** 
    **<footer>** 
        **<a** href_=_"./about"**>**About Us**</a>**                     ❸
        **<a** href_=_"./privacy"**>**Privacy Policy**</a>**             ❸
        **<a** href_=_"./terms"**>**Terms of Use**</a>**                 ❸
        **<a** href_=_"./contact"**>**Contact Us**</a>**                 ❸
    **</footer>** 
  **</body>** 
**</html>** 

❶ Relative paths for resources. Absolute paths cause problems when we publish this code with subfolders in a hosting environment.

❷ Assets from the static folder. Assets referred to in the HTML should be provided in the static folder for correct links in the final website.

❸ Hardcoded menus. In plain HTML, we have to assume that the URLs of the menu entries and their names match what is specified.

Note Hugo does not modify the HTML provided inside the template.

The plain HTML file needs images and an index file to function properly. The images in the assets folder, which we placed for the Eclectic theme, require the use of Hugo Pipes. (We will discuss Hugo Pipes in chapter 6.) For content that does not need processing, we have to use the static folder. Until we start using Hugo’s assets-processing pipeline, we will need to place a second copy of the assets in the static folder. This includes static/index.css, static/image/background.svg, static/image/logo.svg, and static/favicon.ico. These assets are provided with the chapter resources ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/06)](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/06).

Exercise 2.7

For theme independence, it is advisable to customize which page in plain HTML?

1. privacy
    
2. index
    
3. robots.txt
    
4. English
    
5. settings
    

Code checkpoint [https://chapter-02-04.hugoinaction.com](https://chapter-02-04.hugoinaction.com/), and source code: [https://github.com/hugoinaction/hugoinaction/tree/chapter-02-04](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-04). ↻ Restart your dev server.

## 2.4 Continuous delivery

A huge benefit of Hugo and the Jamstack is the ability to have low maintenance and cheap and efficient hosting readily available. We get this power through continuous delivery from the code repository. _Continuous delivery_ is the concept of deploying the changes to our code in an ongoing manner. Good continuous delivery pipelines are automated and require minimal manual effort.

There are many ways to achieve continuous delivery with Hugo, such as writing a script to push our code to a storage provider like Amazon S3 or to place it with Apache/Nginx at the web server layer as with any other web stack. We will focus on the approaches most popular within the Hugo community. You can find more hosting information on the Hugo website ([https://gohugo.io/hosting-and-deployment/](https://gohugo.io/hosting-and-deployment/)), which maintains a running list of various popular hosting providers and scripts to set up Hugo-based hosting.

Although deploying a Hugo-based website on a public cloud provides access to many other services and immense power, the simplicity of Netlify and GitHub Pages is the best approach to get started with learning Hugo. These approaches also support _continuous deployment_, where changes are made live as soon as we submit the code to the code repository. We will focus on Netlify and GitHub Pages as our hosting solution in this book.

Note The following sections assume that the website’s source code has been uploaded to GitHub. Every code checkpoint in the book is a good time to commit changes and deploy it to get a new build.

### 2.4.1 Netlify hosting

Netlify, whose founder coined the term Jamstack, is a leading hosting service for static websites. Netlify provides deployment services with built-in support for Hugo. Netlify takes care of continuous integration and provides APIs for websites to utilize. We can connect our GitHub repository and get static hosting on Netlify (even for private repositories) for free, until we reach its bandwidth limits. Netlify provides a handy command-line tool to perform tasks without leaving the terminal. We can also offer our build instructions via a configuration file called netlify.toml. Netlify additionally supports domain purchases, DNS, and CDN management with things like custom headers.

Tip If you use Netlify, make sure to check out the branch domain feature. Netlify builds and hosts each pull request in a different website and can maintain different versions via branches. We’ll use this feature to host the various versions of the website that we demonstrate in this book. You can navigate to [https://chapter-02-04.hugoinaction.com](https://chapter-02-04.hugoinaction.com/) to see a live website with content up to this chapter so far.

Once we sign up for Netlify ([https://app.netlify.com/signup](https://app.netlify.com/signup)), it provides a step-by-step wizard to host our website. If we have already pushed our website’s source code to GitHub, we can click New Site from Git as figure 2.7 shows, after signing into Netlify to begin deployment.

![CH02_F07_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F07_Jain.png)  

Figure 2.7 After signing up, Netlify presents us with a screen that lists a summary of our Netlify usage and provides the means to set up a new Netlify website. We can connect to a hosting provider or upload our website directly. Connecting to a provider is recommended to get continuous deployment when pushing code.

The New Site from Git button takes us to [https://app.netlify.com/start](https://app.netlify.com/start), where we can connect with our hosting provider (figure 2.8). Once we select the hosting provider, we need to log in and authorize Netlify to access our code repositories.

![CH02_F08_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F08_Jain.png)  

Figure 2.8 Netlify supports connections with multiple hosting providers. Connecting to them is as simple as clicking a button and then logging in.

Once we provide the credentials, Netlify can browse our repository list and provide all repository names in Netlify’s UI for us to select the one we want to deploy (figure 2.9). Note that Netlify does not read GitHub organizations by default, so we need to configure Netlify by using a link on the bottom to provide access.

![CH02_F09_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F09_Jain.png)  

Figure 2.9 Once logged in, we can search for the code repository for the code we want to host via Netlify.

Next, we can specify the branch to build, the build command, and the output directory (figure 2.10). We provide the website URL to Hugo with the command-line arguments hugo --minify --baseURL $DEPLOY_PRIME_URL. The baseURL flag overrides the setup in config.yaml with the one Netlify uses for building branches. If we use pull request previews and branch deploys, it might be better to give each deployment a proper URL. We can also specify build parameters in a file called netlify.toml ([https://docs.netlify.com/configure-builds/file-based-configuration](https://docs.netlify.com/configure-builds/file-based-configuration)).

Note To specify the exact version of Hugo, we can click the Show Advanced button when we specify the build command and then add the environment variable HUGO_VERSION with the correct value, which is the version of Hugo we want to use (for example, 0.91.2). Netlify does not guarantee setting up the latest version of Hugo if the version number is not specified. It is better to have control over the build version by providing it manually.

![CH02_F10_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F10_Jain.png)  

Figure 2.10 Specifying the branch for continuous integration and providing the build command and the output folder to deploy our website. We can designate the Hugo version to use with advanced options by clicking the Show Advanced button.

### 2.4.2 GitHub Pages

GitHub is the Swiss army knife of development. With its extreme popularity in the developer community and its ability to have unlimited free hosting for open source code, GitHub is a perfect place to get started with static hosting. The Pages service can render static HTML from a branch or a folder in our source code repository. GitHub Actions perform continuous integration. There are multiple actions available in the GitHub Actions marketplace for Hugo. We will be using Hugo setup ([https://github .com/marketplace/actions/hugo-setup](https://github.com/marketplace/actions/hugo-setup)) in this section.

The steps for hosting our Hugo-based Acme Corporation website on GitHub Pages follow. Listing 2.7 provides the code for enabling GitHub Pages.

Listing 2.7 Enabling GitHub Pages (.github/workflows/gh-pages.yml)

**name:** GitHub Pages                               ❶
  
**on:** 
  **push:** 
      **branches:** 
        - main                                   ❷
  **workflow_dispatch:**                             ❸
  
**jobs:** 
  **deploy:** 
    **runs-on:** ubuntu-18.04 
    **steps:** 
      - **uses:** actions/checkout@v2                ❹
        **with:** 
          **fetch-depth:** 0 
  
      - **name:** Setup Hugo 
        **uses:** peaceiris/actions-hugo@v2 
        **with:** 
          **hugo-version:** '0.91.2'                 ❺
          **extended:** true 
  
      - **name:** Build 
        **run:** hugo --minify --baseURL=            ❻
--> https://hugoinaction.github.io/GitHubPages 
  
      - **name:** Deploy 
        **uses:** peaceiris/actions-gh-pages@v3      ❼
        **with:** 
          **github_token:** ${{ secrets.GITHUB_TOKEN }} 
          **publish_dir:** ./public 

❶ We can name this workflow what we want.

❷ Triggers this workflow when a push happens on the main branch

❸ Enables workflow dispatch for manual triggers

❹ The checkout action gets the source code. We use fetch-depth to keep some Git history for Hugo’s .Gitinfo (more on this in chapter 4).

❺ “true” specifies that we want to use the extended flavor of Hugo version 0.91.2 for this action, or you can use “latest” to get the latest version.

❻ Command to compile code. We can specify the output page here.

❼ The Deploy action pushes to the gh-pages branch for GitHub Pages to deploy.

1. Create the GitHub Actions file at .github/workflows/gh-pages.yml, which tells GitHub the actions to take ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/07](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/07)). When these changes are pushed to GitHub, these actions automatically execute, creating the gh-pages branch with the compiled version of our website.
    
2. In the GitHub settings for the repository, enable GitHub Pages from the gh-pages branch repository (see figure 2.11). Once enabled, the URL of the website will be visible in the interface.
    
3. After deployment, change the base URL in the GitHub Actions file and in the config.yaml file from the sample value to the correct one provided in the GitHub Actions, then push it again. We can view the updates on the Actions tab on GitHub as figure 2.12 shows.
    

An example website using GitHub Pages is hosted at [https://hugoinaction.github.io/GitHubPages/](https://hugoinaction.github.io/GitHubPages/) with the source code at [https://github.com/hugoinaction/GitHubPages/](https://github.com/hugoinaction/GitHubPages/). We can navigate to the Actions tab in the GitHub UI to see the results of running GitHub Actions, which deploys the website.

![CH02_F11_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F11_Jain.png)  

Figure 2.11 Options for GitHub Pages as a host for a static website. Use the Branch: gh-pages option for Hugo.

![CH02_F12_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F12_Jain.png)  

Figure 2.12 The Actions tab on GitHub shows all executed actions. Each code push can potentially run a GitHub action.

Now the website should be available on the web, and we should be able to navigate to the link provided by GitHub in the pages section once it goes live. GitHub provides a CDN that distributes websites across the planet and is free for a website under its quota limits (size less than 1 GB, a monthly bandwidth of 100 GBs, and around 10 builds per hour as of writing this book). This is a good place for a personal website or for test-driving the Jamstack. Many GitHub Pages document source code already on GitHub, and Hugo is one popular tool for generating that.

Editing on the go

There is a popular misconception that websites built with the Jamstack architecture are difficult to edit unless you have the development environment set up. Most modern Jamstack websites have a continuous environment setup, and we can push it to production with a simple check-in. This system makes Jamstack more flexible than a traditional database-based website stack. We can change not only content but also designs, configurations, and even business logic without setting up a development environment. With scaling not a concern, it is easier to edit on the Jamstack than with the traditional web stack.

  

In case of minor edits, GitHub’s web interface is a valuable tool that provides the ability to edit the website from anywhere. There are applications like CodeHub, PocketHub, or Working Copy (available on both mobile and tablet) to create or modify Markdown documents from a Git repository. We can make our changes anywhere we want, and the continuous integration system ensures they go live within seconds of being committed. Unlike traditional stacks, setting up the local development environment for the Jamstack is much easier, and when we have to, it does not take days.

### 2.4.3 Vercel, Cloudflare, AWS Amplify, and other dedicated Jamstack hosts

Like Netlify, other dedicated Jamstack hosts provide similar feature sets like branch/ commit previews, automatic continuous integration and deployment, and API creation support and management. Vercel provides robust support for managing JavaScript and can be an advantage if our website is getting JavaScript-heavy. Cloudflare Pages are built by one of the biggest CDNs on the planet and provide unlimited bandwidth, better performance than most other services, and a well-defined means to create our APIs (with Cloudflare Workers and Cloudflare Workers KV). AWS Amplify is an AWS service that provides excellent integration with the rest of AWS. The Hugo hosting setup for Cloudflare, AWS Amplify, and Vercel is similar to Netlify, and one cannot go wrong in choosing any of these providers.

### 2.4.4 AWS, Azure, and Google Cloud file storage

If you are using the public cloud features for the other parts of the Jamstack or desire more fine-grained control than that provided with standardized hosting, deploying from Hugo to the cloud is also available. Hugo comes with a built-in command, hugo deploy ([https://gohugo.io/hosting-and-deployment/hugo-deploy/](https://gohugo.io/hosting-and-deployment/hugo-deploy/)), to deploy the website to an AWS S3 bucket, Google Cloud Storage, or Azure Storage. Once we set up the authentication credentials on our machine, we can specify the link to the specific service in the deployment.targets.URL section in the config.yaml file. For example, to deploy to AWS S3, you would enter s3://<Bucket Name>?region=<AWS region>. Hugo automatically identifies the changes between the cloud and the current build and synchronizes those when we run hugo deploy <target name>. We can also specify the caching policies that the cloud exposes to the website’s users in the same section.

![CH02_F13_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F13_Jain.png)  

Figure 2.13 Publishing with the Jamstack. Alex does not give up on the Jamstack even after Bob gets additional resources to continue with the existing stack.

## 2.5 Meeting the goals for performance and maintainability

Hugo and the Jamstack promise solid performance and low ongoing maintenance. Both of these are not absolutes in themselves. There is a gradient: we need to choose the right balance of features, ease of development and use, maintenance, and performance to get the best benefit. A website with no images would likely be faster to load than one with hundreds of them, but that does not mean that it would be the best website for all use cases. Therefore, when analyzing performance and maintainability, we need to consider the use case.

### 2.5.1 Performance

Performance is one notable metric that Hugo’s development team uses to benchmark its builds. We should be able to get good performance for a typical use case without any significant difficulties. We are hosting all the web pages for Acme Corporation on a CDN (prerendered), and the client does not need to do much processing to display the site. While we should find the website quick to load, it is vital to get the performance as a number and tabulate that across builds to be able to compare changes and to fix regressions.

The standard tool for measuring performance is the Audit tool called Lighthouse ([https://developers.google.com/web/tools/lighthouse/](https://developers.google.com/web/tools/lighthouse/)). It’s built into Google’s Chrome browser (figure 2.14). For Acme Corporation, the About page represents a regular page of the website, which we will measure for performance.

![CH02_F14_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F14_Jain.png)  

Figure 2.14 Performance audit for the About page for Acme Corporation using the Google Chrome Lighthouse performance test.

Note Chrome regularly updates the Lighthouse tool with new tests, so the measurement results might not exactly match the screenshot shown.

It is essential to measure the hosted site’s performance on the CDN as the development server from Hugo is not what the users get in production. It is built for development and does not provide the right results. To measure the hosted site’s performance,

1. Go to the View > Developer > Developer Tools menu in Google Chrome to open the web inspector.
    
2. Go to the Lighthouse tab and run an audit. You should be able to achieve a decent audit score for performance on most Hugo websites.
    

Lighthouse may suggest issues in the theme. If so, there is an option to clone the theme or to create a bug for the theme developer to fix.

### 2.5.2 Maintainability

The maintainability of the web setup is difficult to measure directly. There is no tool to tell whether a stack is maintainable. One way to check how much effort it would take to maintain a system is to list each of its dependencies and figure out which dependencies require ongoing security updates, which need to be abandoned by the developer, or which can become difficult to update due to nested dependencies. We should also measure the effort to remove a dependency in case it is not actively maintained. Luckily, for the Hugo-based setup we just discussed, we have few dependencies. In our measurement system, we can consider a rewrite, huge updates, or partial rewrites as high risks, and tweaks that do not involve many changes as medium risks. At the same time, a low risk would refer to no minimal manual intervention. Let’s try to assess this for the website we have built so far, right after the next exercise.

Exercise 2.8

What is the primary reason to benchmark the performance of a website?

1. Find overall performance issues.
    
2. Plot a graph to show on our website.
    
3. Compare performance across multiple builds of our website and multiple builds of Hugo to find faulty behavior.
    
4. Find bugs in our website.
    

- _Our Acme Corporation’s website created in this chapter depends on Hugo._ Hugo has had breaking changes in the past releases, but most of them have been minor. We do not need to update for security fixes because it is a development-only dependency. This could be rated as low in an ongoing effort if we are happy with the website or as medium for an upgrade.
    
- _The hosting on GitHub Pages requires no ongoing effort to maintain._ This is among the most critical services for developers on the internet. We can, therefore, rate both ongoing maintenance and upgrade as low. If we use Netlify, it manages the upkeep for us, and the effort there is also low. Because it is a lot less popular than GitHub, there is an inherent risk of Netlify pivoting to a new business model or shutting its doors. Migration to GitHub is easy for the type of website built here, and its overall risk is low.
    
- _The Eclectic theme chosen for Acme Corporation is dependent on a few JavaScript-based plugins._ These plugins are stable, however, and haven’t had significant changes in years. Still, Eclectic is not heavily used, and if it gets abandoned, the team at Acme Corporation will have to pick up the task of adding fixes to support newer Hugo versions when they want to update the website. That would be a medium effort commitment (unless they want new features).
    

Overall, the ongoing work to keep the website we built in this chapter alive is meager. If we need to upgrade it, the effort would be low to medium, depending on the breaking changes in Hugo and the theme developer’s ability to adapt to those. Note that as we progress further along with this book, we will add more dependencies to our website, especially in part 2. This will increase the maintenance overhead. While an attempt has been made to look for dependencies that are self-contained, readers are advised to weigh the pros and cons of adding dependencies independently every time something is needed in their own projects.

### 2.5.3 Choose the theme wisely

The performance and maintenance risks of a website depend heavily on the theme selected. If the theme is not good, Hugo’s hard work maintaining its performance will not show in your website’s build time. The main maintenance risk to a Hugo-based website is the risk of depending on a theme that stops being compatible with the newer versions of Hugo. We can continue to use the older version of Hugo and the theme indefinitely without worrying too much about security issues because the content is static. But if we ever want to update Hugo and the theme is not supported anymore, we would be on our own to maintain the theme. It is a good idea to be theme agnostic, at least early on in a website project, so that if we find a problem with the theme we are using, we can move to a different one quickly.

Themes can also be an excellent source for learning how to use Hugo best. Many developers using Hugo choose the themes as the starting point rather than the absolute solution. One big reason to choose Hugo is to customize everything, and forking the theme is a powerful way to perform that task. We will be moving out of the Eclectic theme into our custom theme by the end of chapter 7.

If we want to continue to build our website with a theme maintained by someone else, it is a good idea to investigate portability. Hugo provides standardization across themes, and switching Hugo themes is not difficult (see listing 2.8). We will be adding another theme to Acme Corporation’s website to make sure our code is portable. We provide a copy of the Universal theme for Hugo in chapter resources ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/08](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/08)) and also host it at [github .com/hugoinaction/Universal.](https://github.com/hugoinaction/Universal) You can copy that theme to the themes folder and enable it with the website configuration. You may need to restart the development server.

Listing 2.8 Changing a theme to Universal (config.yaml)

**theme:** Universal 

While the previous code works and renders the website, there is more configuration that we need to do to get the maximum benefit of the Universal theme. For that, place logo.png in the static/image/logo.png folder and update the configuration to include the parameters in the following listing (below the existing params for footer) for Universal to be able to parse them ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/09](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/09)).

Listing 2.9 Changes to support the Universal theme (config.yaml)

**theme**: Universal 
**params:** 
  **footer**: 
        ... 
  **style:** blue 
  **logo:** /image/logo.png 
  **logo_small:** /image/logo.png 
  **about_us:**  > 
    Acme Corporation is the world's leading manufacturer 
    of digital shapes. From squares and circles to 
    triangles and hexagons, we have it all. Browse through 
    our collection of various forms with different 
    thicknesses and line styles. We shape the world. 
    You live in it. 
  
  
  **recent_posts:** 
    **enable:** true 

The configuration file for Universal is available in both TOML and YAML format in the code resources with this book ([https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/10](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-resources/10)). Note that the configuration for Eclectic has not been removed, and we can switch between the two themes easily.

Code checkpoint [https://chapter-02-05.hugoinaction.com](https://chapter-02-05.hugoinaction.com/), and source code: [https://github.com/hugoinaction/hugoinaction/tree/chapter-02-05](https://github.com/hugoinaction/hugoinaction/tree/chapter-02-05).

Because each theme has a unique home page, switching themes will be considerably easier if we choose our own customized HTML-based home page. That way, if we render now or with a live reload, the home page remains the same. Because the About page is styled, it will match the Universal theme (figure 2.15) if we switch to that theme.

![CH02_F15_Jain](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297007/files/Images/CH02_F15_Jain.png)  

Figure 2.15 Terms of Use page for Acme Corporation in code (left), Eclectic (middle), and Universal (right). When we switch themes in Hugo, most of the content that we provide as Markdown still works. Only the parameters provided in places like the configuration file need to be reworked.

We will be reverting to Eclectic for the rest of the book. With a running website, it is time to add some more content, so we will do that in chapter 3.

## Summary

- Hugo is available for installation in most major package managers on Linux, macOS, and Windows.
    
- Hugo has extensive command-line functionality to minimize the work that its users need to do. It has handy options that help build all parts of a website, from adding module dependencies to creating new Markdown-based documents.
    
- A Hugo project consists of folders beyond the content and themes folders: static for static content, data for structured data, layouts for theme overrides, resources for Hugo’s internal caching, assets for images, JavaScript, and CSS files, and public for the generated output. It also includes archetypes for posted templates and a configuration file for global settings.
    
- Hugo themes can be added in various ways, the simplest of which is to directly copy a theme to the themes folder. We need to configure these with standard and theme-specific parameters and file placements before using.
    
- Content can be added as Markdown, theme-specific structured data, or in an overridden HTML template.
    
- Hugo websites can be hosted easily across the planet via GitHub Pages and Netlify, which provide continuous delivery support without making the developer do much work.
    
- We can switch themes, but if we use a lot of theme-specific data (like data supplied via params in the configuration), then that work needs to be redone. We should investigate theme switching early on so that we can switch out quickly if the Hugo theme gets abandoned.
    
- We can use Google Chrome’s Lighthouse feature for measuring performance. We should also do a full dependency audit to check maintainability.
    
- Every website needs to be monitored for maintainability and performance regularly during development to ensure quality. Hugo offers excellent performance and has a small set of dependencies, but the website performance and maintainability still depend on the chosen theme.