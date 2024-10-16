---
id: 01JA8KRT8F4XX2R19A68DTBNX3
modified: 2024-10-15T13:33:45-04:00
title: Chapter 3 - Configuring a Backend Environment with Bun and Hono
description: Using Bun with Hono
tags:
  - typescript
  - bun
  - hono
  - books
  - full-stack
  - backend
---
In this chapter, we will start with understanding what the Bun JavaScript runtime is and why it’s becoming a popular JavaScript runtime among developers. We’ll then introduce a library for developing backends called Hono, which provides a simple yet powerful way to build web applications. You’ll set up your project, including installing Bun and Hono, learn about some useful code structures, and configure essential tools such as middleware, environment files, Prettier, and ESLint.

By the end of this chapter, you’ll have a strong grasp of Bun’s environment and will be able to set up your own backend projects. All of this is going to be essential for you to be able to develop functional and complete backend applications and build the foundation for our chat app. You will also learn how to improve the readability and configurability of our code and make it easier to develop.

In this chapter, we’re going to cover the following main topics:

Introducing Bun
Introducing Hono
Setting up your project
Adding linting and formatting
Adding middleware
Handling environment variables
Discussing the project’s structure
Technical requirements
First, we need to install Bun.

For Linux and MacOS, use the following command:


$ curl -fsSL https://bun.sh/install | bash
Bun has limited experimental support for Windows, so it’s recommended to use Windows Subsystem for Linux (WSL), but you can still install it using the following command:


$ powershell -c "irm bun.sh/install.ps1|iex"
If you get a command not found error, you need to add the C:/Users/currentUser/.bun folder and C:/Users/currentUser/.bun/bin to the Windows PATH system variable.

This is enough to get started, and we will install more libraries as we go. All the code examples we discuss are available in the GitHub repository:

https://github.com/PacktPublishing/Full-Stack-Web-Development-with-TypeScript-5/tree/main/Chapter03

Introducing Bun
Bun is a JavaScript runtime and is an alternative to Node.js and Deno. It’s rapidly gaining traction due to its exceptional performance and ease of use. It has native support for TypeScript and a lot of other benefits, including the following:

High-performance runtime: Bun is built on the JavaScriptCore engine used in Safari, which is known for its speed. This and other optimizations make Bun an incredibly fast runtime, offering a performance boost, sometimes handling 5 or 10 times as much load as Node.js can.
Built-in bundling and transpiling: Unlike traditional runtimes that require external tools for these tasks, Bun comes with built-in capabilities for bundling and transpiling. So, you only need Bun to interpret your code and transpile your code, so we don’t need configuring tools such as webpack, esbuild, or rollup.
TypeScript and JSX support: Bun provides first-class support for TypeScript and JSX out of the box.
Efficient package management: Bun’s package manager is designed to be faster and more efficient than traditional package managers such as npm or yarn, and is from 2 to 10 times quicker in different scenarios.
Node.js compatible: Bun is designed to be a drop-in replacement for Node.js. Bun natively implements most of the essential Node.js and Web APIs. So, you can use packages developed for Node.js in Bun and even replace Node.js with Bun in your existing project. It doesn’t work in 100% of cases, but the compatibility is fairly high, and most projects will run as they are without needing to change much.
Useful built-in libraries and tools: Bun provides some important tooling that is needed to write most real projects out of the box, such as test runners, environment variable handlers, and password generators. This is going to save us quite some time because, otherwise, we would need to introduce external libraries for these functions anyway.
If you also want to see more of the comparison between Node.js, Deno, and Bun, you can visit https://bun.sh/ to see the charts that compare them.

Bun will be quite helpful in developing our backend chat application, thanks to its built-in infrastructure, including even an HTTP server. However, for our backend needs, we require something more powerful in functionality. That’s where the Hono library steps in, offering a minimalist and flexible approach to web application development.

Introducing Hono
Hono is a modern, lightweight web framework designed for JavaScript and TypeScript developers. It’s built to give you just what you need for web development without providing more features than necessary and taking choices away from you.

Hono handles essential tasks such as routing, data validation, and middleware creation but doesn’t enforce other things, such as database interactions, architectural patterns, or logging libraries. This flexibility lets you choose the best tools and practices for your project where you need to, but also provides a good number of tools to do the boring things.

The main features of Hono are as follows:

TypeScript integration: Hono integrates smoothly with TypeScript, providing additional type safety around your endpoints
Routing: It provides straightforward route handling, which helps us to identify which endpoint to call based on the URL
Middlewares: Hono supports middleware that lets you add functionality before and after the request handler executes
Error handling: The library includes built-in error handling to streamline debugging and provide meaningful errors to the caller
JSON support: Hono has native support for JSON parsing and response formatting is a part of Hono, so we can easily accept and return the most popular API format
Query and parameter parsing: It automatically parses URL parameters and query strings
Static file serving: Hono can serve static files, such as images, CSS, and JavaScript
Customizable context: You can extend the request context with custom properties or methods
Cookie handling: The framework provides functions for setting, getting, and deleting cookies, which are often used for authentication purposes
Headers manipulation: Hono simplifies the manipulation of HTTP request and response headers, which is important for security
Additional libraries: Hono complements its core functions with external libraries, mainly as middleware, for functionalities such as CORS setup, request logging, and tracking endpoint performance metrics
As we develop our app, we will get more familiar with Hono and Bun and what they can do, so let’s set up our project and see them in action.

Setting up the project
Let’s begin by setting up the core part of our project. We’ll make this easier by using templates, which help us quickly create the basic structure:

Start by running this command to create a Hono project from a template using Bun:

$ bun create hono chat_backend
When prompted, select the bun template and hit Enter. In a few seconds, you’ll have a minimal setup ready. Next, navigate to the project folder, install the necessary dependencies, and launch the local backend server with these commands:

$ cd chat_backend
$ bun install
$ bun run dev
After this, you can expect the following output in our terminal:

$ bun run --hot src/index.ts
Started server http://localhost:3000
Now, if you visit http://localhost:3000 in your browser, you should see the message Hello Hono! displayed. Congratulations, you’ve just created your first Hono application! It’s pretty basic for now, but we’ll be adding more to it shortly.

Let’s take a look at the package.json file to understand its contents:

package.json


{
  "scripts": {
    "dev": "bun run --hot src/index.ts"
  },
  "dependencies": {
    "hono": "^3.12.2"
  },
  "devDependencies": {
    "@types/bun": "^1.0.0"
  }
}
In this file, we can see a scripts key, which defines what command we will be able to run. The dev script is what Bun runs. It’s straightforward – it starts our project using the entry point src/index.ts. A notable feature here is the --hot flag, which enables hot reloading. This means that Bun automatically reloads files that have been changed without restarting the entire OS process. This makes for a much faster and more pleasant development experience.

Regarding dependencies, we currently have one production dependency, hono, and one development dependency, @types/bun. The @types/bun package is a TypeScript pattern that you’ll often see. It provides type definitions that are specific to the Bun environment, such as the return types of Bun’s functions.

Now, let’s check out the TypeScript configuration in tsconfig.json. We’ll remove JSX-related configurations (as we do not use JSX in the backend) and add Bun types, leading to the following content:

tsconfig.json


{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "types": ["bun-types"]
  }
}
Let’s break down what each part of compilerOptions does:

"esModuleInterop": true: This setting allows for the compatible use of CommonJS modules in TypeScript, similar to ES6 modules. It simplifies importing CommonJS modules, enabling syntax such as import fs from 'fs' instead of import * as fs from 'fs'.
"strict": true: Activates all strict type-checking options in TypeScript. This leads to the most thorough type checking, including strict null checks and no implicit any, among others. It’s a way to ensure more comprehensive error checking during code compilation.
"types": ["bun-types"]: Specifically includes the Bun type library for the compiler. Usually, you might not need to specify types like this, but Bun, being a unique environment, requires its types to be explicitly declared in this configuration.
Next, let’s discuss the bun.lockb file. This file is a lockfile utilized in Bun. Its main role is to maintain consistency in your project’s dependencies by locking down specific versions of each package, along with their transitive dependencies.

For instance, if your package.json indicates hono as "^3.12.2" (meaning any version from 3.12.2 up to, but not including, 4.0.0), bun.lockb might specifically lock it to "3.13.0". This ensures that every developer working on the project is using the exact same version, preventing the common issue of “it works on my computer.”

Notably, the bun.lockb file is in binary format, which Bun uses to reduce file size and optimize the performance of its package management system.

Now, let’s take a look at the final important file, src/index.ts, which contains our server’s code:

src/index.ts


import { Hono } from 'hono'
const app = new Hono()
app.get('/', (c) =%3E {
  return c.text('Hello Hono!')
})
export default app
Now, let’s break down what’s happening in this code:

Importing Hono and server initialization: We start by importing the Hono library. After that, we create a new instance of the Hono server with const app = new Hono().
Route definition: The line app.get('/', is where we define a route. It instructs the Hono app to handle HTTP GET requests at the root URL path '/'. When this URL is accessed, the callback function (c) => { return c.text('Hello Hono!') } is executed. This function utilizes the context object c provided by Hono to send a text response Hello Hono! back to the client.
Exporting the app: export default app allows the app instance to be imported and utilized in other files, or it can be used to start the server. However, working with plain text responses isn’t very common, so we’ll change our endpoint to return a JSON response instead. So, we’ll replace return c.text('Hello Hono!') with return c.json({'message':'Hello Hono!'}).
After making this change and reloading the page, your browser should display the following:


{
  "message": "Hello Hono!"
}
And here we are, with our first JSON-returning endpoint! Before moving forward with the development of our application, it’s a good idea to integrate some tools that will simplify our workflow and enhance our code’s quality. We will set up linting, formatting, and utility middleware, and we will also discuss handling environment variables. Let’s begin with linting and formatting.

Adding linting and formatting
Linting is the process of scanning code to find errors or inconsistencies without actually running it. It’s incredibly useful for catching unused variables, preventing console.log statements in production, and even identifying logical errors such as infinite loops.

As with static typing, it’s all about making our code better and catching issues early. For this project, we’ll use ESLint, the most common linter for JavaScript. We’ll keep things simple and stick to the rules recommended by the Hono project.

First, let’s install eslint globally so that we can use it from the command line:


$ bun install –g eslint
Now let’s install the rules recommended by Hono:


$ bun install --dev @hono/eslint-config
Now, we can create a file in the root of our project with eslint configs and put the following configuration into it to include the recommended linting rules by Hono:

.eslintrc.cjs


module.exports = {
  extends: ["@hono/eslint-config"]
};
This sets you up with the basic eslint configuration. We can run our linter with the following command:


$ eslint --fix
It’s beneficial to integrate the linter with your IDE. This allows it to run automatically when you save a file to make the developing experience even smoother.

Here’s how you can do it in WebStorm:

Open Settings.
Go to Languages & Frameworks | JavaScript | Code Quality Tools | ESlint.
Turn on Automatic ESlint config if it was turned off.
Check the mark on run eslint --fix on save.
To enable it in Visual Code, you can do the following:

Open Settings.
Type editor.codeActionsOnSave into the search bar at the top.
Choose Edit in settings.json to open the settings.json file in VSCode.
Insert these lines into your settings.json file:

"editor.codeActionsOnSave": { "source.fixAll.eslint": true},
"eslint.alwaysShowStatus": true
To see how it works in action, you can create a variable in src/index.ts that is not used. Now, save the file and you will see a warning in our IDE that the variable is not used. Also, now you can run eslint in your terminal and you are going to get a message similar to this:


 8:7  warning  'a' is assigned a value but never used  @typescript-eslint/no-unused-vars
 ✖ 1 problem (0 errors, 1 warning)
Now, let’s focus on formatting. Formatting ensures that our code has a consistent style, which is incredibly valuable when multiple people are collaborating on a project. It’s interesting to note that there are many effective code styles out there. The key isn’t which style you choose, but rather that everyone sticks to the same style consistently. A formatting tool that automatically enforces a style is a lifesaver, saving countless hours during code reviews and preventing disputes over code style. For our project, we’ll integrate Prettier, a popular tool that automatically formats code to a predefined style.

It’s also very comfortable to integrate Prettier in eslint so that they run together and fix both formatting and linting errors at the same time. This allows both tools to run together, fixing formatting and linting errors simultaneously. First, we will need to install additional libraries to handle Prettier as eslint plugins:


$ bun install --dev eslint-config-prettier eslint-plugin-prettier
And now you can expand our eslint configuration to look like this:

.eslintrc.cjs


module.exports = {
  extends: ["@hono/eslint-config", "plugin:prettier/recommended", "prettier"],
  plugins: ["prettier"],
  rules: {
    "prettier/prettier": "error",
  }
};
This addition includes Prettier rules in the eslint configuration. We’ve set Prettier errors to be treated as errors rather than warnings. This strict approach helps maintain the overall quality of the project.

Now, try adding extra tabs or spaces to a variable in your code. When you save the file, you’ll notice it automatically formats to the correct style, thanks to Prettier. Handy, right?

With linting and formatting set up, let’s move on to discussing additional middleware that can further enhance our development experience.

Adding middleware
Middleware in Hono acts as interceptor functions that process the incoming HTTP request before it reaches the final route handler. Middleware can also process responses before they are sent back to the client. They can modify requests (e.g., parse the body, add headers) and responses (e.g., set cookies, modify headers).

Middleware executes in the order it is defined in the code. Each middleware function can decide whether to pass a request to the next piece of middleware or to end the response cycle. It is commonly used for logging, authentication, error handling, and data parsing.

We are going to implement our own middleware to handle authentication. However, for now, let’s integrate some ready-made middleware provided by Hono. We’ll add one for basic request logging and another to append performance metrics to responses.

Here’s how we modify the beginning of our index.ts file to include these:

src/index.ts


import { Hono } from 'hono'
import { logger } from "hono/logger";
import { timing } from "hono/timing";
const app = new Hono()
app.use("*", timing());
app.use("*", logger());
In this code, we import the logger and timing libraries and then add them to our app using the app.use function with the path '*'. This path specification ensures the middleware is applied to all incoming requests, though it can be adjusted to target specific endpoints.

Now, when you hit the endpoint, the logs will include additional lines such as this:


  %3C-- GET /
  --> GET / 200 4ms
These log entries provide details such as the accessed endpoint, the response status code, and the time taken for the response. Additionally, if you check the response headers in your browser’s developer console, you’ll notice a new 'Server-Timing' header attached. It carries similar information, such as this:


total;dur=0.1;desc="Total Response Time"
This addition offers valuable insights into the performance of our requests.

Now, let’s discuss how to manage environment variables effectively.

Handling environment variables
Environment variables are a standard way of keeping secrets or platform-specific details outside of the code base. They’re also handy for configuring various aspects of an application. Bun has a built-in mechanism for handling environment variables, which can be accessed in your code through Bun.env.VARIABLE_NAME. It even automatically reads from a .env file, so all we need to do is provide this file.

Here’s how to set it up:

First, create a file named .env.
Inside the .env file, define a variable such as TEST=test value.
In our src/index.ts, add the line console.log(Bun.env.TEST). Remember, environment variables aren’t hot-reloaded, so you’ll need to restart the server to see the changes in your console. After restarting, you should see test value printed in the terminal.
Sometimes, you might need different environment files for various scenarios, such as local development versus production. Or, you might have different sets of environment variables for different use cases, so you end up with multiple .env files, while Bun will handle only .env for you. In such situations, you can use a tool such as dotenv to load additional environment variables from multiple files.

Let’s illustrate it by logging another environment variable from .env.dev. Add console.log(Bun.env.AI) to src/index.ts and add AI=chat in our .env.dev file. You will see that the value is undefined. Let’s fix this by restarting our server and providing it with additional environment variables files:


$ bunx dotenv –e .env -e .env.dev -- bun run dev
After restarting the project with this command, you should see both test value and chat in your console.

With environment variables set up, we’re now ready to dive into discussing the project structure for our application.

Important note

Do not add environment files to your Git repo because it is a security risk. Your environment files must be put into .gitignore file, and should not be committed.

Discussing the project structure
When structuring code for a project, it’s essential to have a clear separation of concerns. This means organizing files and folders in a way that each component has a distinct responsibility, contributing to overall project clarity and maintainability. For our relatively small project, we’ll opt for a simple structure.

Here’s the structure we’ll use and the rationale behind it:

src/controllers: This folder will contain specific REST endpoint handlers. Each controller deals with incoming requests and generates appropriate responses. By isolating endpoint logic in controllers, we make it easier to update or extend API functionalities.
src/middlewares: Here, we’ll store additional middleware functions for Hono. Middleware is crucial for processing requests and responses, offering functionality such as authentication, logging, or data parsing. Keeping them in a dedicated folder allows for easy reuse and management.
src/models: This directory is designated for type definitions of the objects used in our code. It ensures a centralized location for data structure definitions, enhancing code consistency and reducing the likelihood of type-related errors.
src/storage: A folder to manage code that interacts with various storage solutions, such as in-memory databases, SQL databases, or ORMs. This separation ensures that changes in storage logic don’t impact other parts of the application.
src/constants.ts: This is a file to hold project-wide constants. By centralizing constants, we ensure uniformity and prevent discrepancies that can arise from hardcoding values in multiple locations.
src/index.ts: This is the entry point of our application. This is where we tie together various components of our application, setting up the server, middleware, routes, and any initial configurations.
tests: This is a dedicated folder for storing test files.
This structure is tailored for our chat backend application, ensuring each module has a clear role and responsibility. It fosters an organized development environment, making it easier to navigate, maintain, and scale our application as needed.

Summary
In this chapter, we explored the essentials of Bun, a rising star in the JavaScript runtime landscape, and Hono, a framework known for its simplicity and effectiveness in web application development.

We walked through the practical steps of setting up a development environment tailored for Bun and Hono. This included the installation process, establishing a coherent code structure, and integrating essential tools such as logging and ESLint. The chapter also covered key topics such as project setup, linting and formatting, middleware integration, managing environment variables, and discussing effective project structure strategies.

In the next chapter, we’re going to implement the backend functionality for our chat app. Our backend will use in-memory storage, and we’ll have the functionality from start to finish.>)