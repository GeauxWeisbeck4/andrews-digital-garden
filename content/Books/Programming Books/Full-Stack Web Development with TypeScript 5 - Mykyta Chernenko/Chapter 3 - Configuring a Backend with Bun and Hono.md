---
id: 01J6G7CRXC4G9JF8Y8C436ZTWV
modified: 2024-08-29T19:29:40-04:00
title: Configuring a Backend with Bun and Hono
description: Creating a backend with Bun and Hono
tags:
  - bun
  - hono
  - backend
  - typescript
  - full-stack
  - web-development
  - programming
---
In this chapter, we’re going to cover the following main topics:

- Introducing Bun
- Introducing Hono
- Setting up your project
- Adding linting and formatting
- Adding middleware
- Handling environment variables
- Discussing the project’s structure

# Technical requirements

First, we need to install Bun.

For Linux and MacOS, use the following command:

# Introducing Bun

**Bun** is a JavaScript runtime and is an alternative to Node.js and Deno. It’s rapidly gaining traction due to its exceptional performance and ease of use. It has native support for TypeScript and a lot of other benefits, including the following:

- **High-performance runtime**: Bun is built on the JavaScriptCore engine used in Safari, which is known for its speed. This and other optimizations make Bun an incredibly fast runtime, offering a performance boost, sometimes handling 5 or 10 times as much load as Node.js can.
- **Built-in bundling and transpiling**: Unlike traditional runtimes that require external tools for these tasks, Bun comes with built-in capabilities for bundling and transpiling. So, you only need Bun to interpret your code and transpile your code, so we don’t need configuring tools such as webpack, esbuild, or rollup.
- **TypeScript and JSX support**: Bun provides first-class support for TypeScript and JSX out of the box.
- **Efficient package management**: Bun’s package manager is designed to be faster and more efficient than traditional package managers such as npm or yarn, and is from 2 to 10 times quicker in different scenarios.
- **Node.js compatible**: Bun is designed to be a drop-in replacement for Node.js. Bun natively implements most of the essential Node.js and Web APIs. So, you can use packages developed for Node.js in Bun and even replace Node.js with Bun in your existing project. It doesn’t work in 100% of cases, but the compatibility is fairly high, and most projects will run as they are without needing to change much.
- **Useful built-in libraries and tools**: Bun provides some important tooling that is needed to write most real projects out of the box, such as test runners, environment variable handlers, and password generators. This is going to save us quite some time because, otherwise, we would need to introduce external libraries for these functions anyway.

If you also want to see more of the comparison between Node.js, Deno, and Bun, you can visit [https://bun.sh/](https://bun.sh/) to see the charts that compare them.

Bun will be quite helpful in developing our backend chat application, thanks to its built-in infrastructure, including even an HTTP server. However, for our backend needs, we require something more powerful in functionality. That’s where the Hono library steps in, offering a minimalist and flexible approach to web application development.

# Introducing Hono

**Hono** is a modern, lightweight web framework designed for JavaScript and TypeScript developers. It’s built to give you just what you need for web development without providing more features than necessary and taking choices away from you.

Hono handles essential tasks such as routing, data validation, and middleware creation but doesn’t enforce other things, such as database interactions, architectural patterns, or logging libraries. This flexibility lets you choose the best tools and practices for your project where you need to, but also provides a good number of tools to do the boring things.

The main features of Hono are as follows:

- **TypeScript integration**: Hono integrates smoothly with TypeScript, providing additional type safety around your endpoints
- **Routing**: It provides straightforward route handling, which helps us to identify which endpoint to call based on the URL
- **Middlewares**: Hono supports middleware that lets you add functionality before and after the request handler executes
- **Error handling**: The library includes built-in error handling to streamline debugging and provide meaningful errors to the caller
- **JSON support**: Hono has native support for JSON parsing and response formatting is a part of Hono, so we can easily accept and return the most popular API format
- **Query and parameter parsing**: It automatically parses URL parameters and query strings
- **Static file serving**: Hono can serve static files, such as images, CSS, and JavaScript
- **Customizable context**: You can extend the request context with custom properties or methods
- **Cookie handling**: The framework provides functions for setting, getting, and deleting cookies, which are often used for authentication purposes
- **Headers manipulation**: Hono simplifies the manipulation of HTTP request and response headers, which is important for security
- **Additional libraries**: Hono complements its core functions with external libraries, mainly as middleware, for functionalities such as CORS setup, request logging, and tracking endpoint performance metrics

As we develop our app, we will get more familiar with Hono and Bun and what they can do, so let’s set up our project and see them in action.