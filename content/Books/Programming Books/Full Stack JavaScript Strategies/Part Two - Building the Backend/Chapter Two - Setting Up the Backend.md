---
id: 01JGFNYSRT5CVRK2XNWA7HXD5M
modified: 2024-12-31T19:59:03-05:00
---
# Chapter 2. Setting Up the Backend

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 2nd chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You’ve got all your tickets ready to go, and you can assume you’ve been tasked with building the bones of the backend. The framework of choice at your company is [NestJS](https://nestjs.com/). It’s OK if you’ve never seen NestJS or even heard of it before. The docs for it are good, and you’ll have other code you can reference. Reach out to other developers to see why certain architecture and design decisions have been made on existing backends.

In this chapter, you’ll learn about:

- Making architecture decisions for the backend
    
- Checking the app to make sure it runs after initial setup
    
- Writing the initial documentation
    

These tasks are all crucial to ensuring that development is able to run smoothly as both the Product and the dev team grow and change over time. Everything that you build on the backend will start from the foundation you set here, so take your time to thoroughly discuss options with your team.

# Why NestJS?

Before we dive too deep into the weeds, let’s discuss why NestJS is a solid choice. It’s basically like a backend architecture in a box. As soon as you initialize the app, you immediately have access to validation, authentication, routing, controllers, data schema, and a lot of other functionality. Instead of you having to piece everything together, NestJS gives you everything you need to make a scalable backend from the beginning. That does mean it’s a little less flexible in how you organize it, but that’s the trade-off for all the up-front scaffolding.

This is when you get to have some fun as you set up the initial repo for the backend. The decisions you make here drive the direction for the future of this project. As you come to questions that have options, ask your team for their technical input. They might be aware of something you don’t know about. Not only does this help you build a better app, but it also helps everyone on the team learn.

There are so many considerations for building a backend that having other input will help you make decisions faster and more confidently. It will also help to keep architecture decision records (ADRs). These keep track of why and when important architectural decisions were made. You can see some [examples in this GitHub repo](https://github.com/joelparkerhenderson/architecture-decision-record).

It can be a little overwhelming being the one to open the empty terminal and bootstrap a completely new project. There’s nothing to build on except examples and boilerplate code. It will go in whatever direction you and your team decide to take it. This is a fun time because you can really set up the codebase to be clean to update, test, and expand.

# Choosing a Project Approach

This project will be a monolithic backend with a couple of serverless functions. This will allow you to build and deploy quickly, and you’ll get to work with a hybrid backend architecture. You could have also decided to take a microservices approach. Then all your APIs could be serverless functions, for example. Your choice of backend architecture depends on how the app is going to grow over time.

Microservices can take more resources and more engineers to maintain compared to monoliths, but they can be great if you go with a domain-driven architecture, which you’ll learn about in [Chapter 30](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch30.html#understanding_the_business_domain). On the other hand, monoliths can make it more difficult to make larger changes as quickly as you can with microservices.

You’ll also be using [TypeScript](http://typescriptlang.org/) to build the full stack. The reason you’re starting the project off with TypeScript is to help make debugging easier because it’s harder to write incorrect code. By strictly enforcing typing, you’ll catch some errors before you even run the code. TypeScript also makes it easier to see what fields and data types you’re working with and where they come from, which is wonderful for keeping types consistent across the frontend and backend. Having typing in place is like having documentation for how things are connected, and it lets you see all the parameters you have available.

# Setting Up NestJS

You’ll walk through all the steps to bootstrap this whole project. To start, you need to have a few environmental things in place. You can follow along with [the demo repo](https://github.com/flippedcoder/dashboard-server). You’ll need to install the NestJS CLI. The current version of NestJS as of this writing is 9.2.0. Open a terminal and run the following command:

npm i -g @nestjs/cli

You can create a new NestJS project with the command:

nest new <your-project-name>

For this project, you’ll use this command:

nest new dashboard-server

NestJS will ask you which package manager you want to use. Select `npm` and hit Enter.

? Which package manager would you ❤️  to use? `(`Use arrow keys`)`
❯ npm 
  yarn 
  pnpm

###### NOTE

Keeping track of package versions is something a senior dev might do silently. As we go through building this app, I’ll introduce you to some leadership best practices, such as keeping docs updated and making sure everyone’s local environment is running correctly. You’ll also have to consider things like linting with something like [ESLint](https://eslint.org/), formatting rules with a common tool like [Prettier](https://prettier.io/) for the repo, and any number of environment configs to keep everything consistent in the code for everyone.

After that, the installation will start, which might take a few minutes. This process includes installing all the project dependencies, scaffolding the entire architecture, adding boilerplate code, and more. It even comes with TypeScript preconfigured. Take some time to go through this repo before you start making changes. Whether you’re working on a brand-new app like this one or a legacy project, always take time in the beginning to get a high-level understanding of how everything works.

Key things to look at are:

- The _package.json_ so you can see which packages are used in the project
    
- All the config files so you know how things work under the hood
    
- The test files because these give you a good idea of the major functionality of the app
    
- Some of the actual code to see how developers are expected to write things and if anything can be improved
    

After you’ve poked around a bit, go ahead and run the app with these commands:

```
cd
```

You’ll see an output in the terminal like this that will let you know the app is running successfully:

> dashboard-server@0.0.1 start
> nest start

`[`Nest`]` `20891`  - `03`/03/2023, `8`:18:46 PM     LOG `[`NestFactory`]` Starting Nest application...
`[`Nest`]` `20891`  - `03`/03/2023, `8`:18:46 PM     LOG `[`InstanceLoader`]` AppModule dependencies initialized +11ms
`[`Nest`]` `20891`  - `03`/03/2023, `8`:18:46 PM     LOG `[`RoutesResolver`]` AppController `{`/`}`: +5ms
`[`Nest`]` `20891`  - `03`/03/2023, `8`:18:46 PM     LOG `[`RouterExplorer`]` Mapped `{`/, GET`}` route +3ms
`[`Nest`]` `20891`  - `03`/03/2023, `8`:18:46 PM     LOG `[`NestApplication`]` Nest application successfully started +1ms

This is where one of your first senior decisions comes up. How exactly are you going to test this backend? Having a consistent environment for development will change how quickly you can test code and how much frustration the team experiences as the product grows and the team grows, too.

###### NOTE

A standard choice is to use [Docker](https://docs.docker.com/get-started/) because you can set up everything from a Postgres database instance to the server that the app will run on in a container (or two). These containers can hold the exact same data and versions and be set up on any local environment, regardless of the operating system (OS). Another option would be using some functionality in VS Code called [Dev Containers](https://code.visualstudio.com/docs/devcontainers/create-dev-container). Depending on the team and the project, this can be a good choice.

## Testing the Backend Locally

Some popular testing tools you’ll see include [Postman](https://www.postman.com/downloads/) and [RapidAPI](https://paw.cloud/). You’ll work with Postman in this book (and many times in real life) because it has a free version. This is how you’ll be able to make requests to your endpoints with different headers, body values, and other values and see what responses you get. That’s typically how you test the backend before a UI is available.

Remember, tools like these are here to help you move faster. Don’t get caught up in all the tool options. At the end of the day, it doesn’t matter what you use for your local development as long as you’re able to make meaningful progress. Now that you have Postman available, make a GET request to _http://localhost:3000_. You should see `Hello World!` as the response, like in [Figure 2-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch02.html#get_request_and_response_in_postman).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0201.png)

###### Figure 2-1. GET request and response in Postman

Everyone will have a different preference for the tools they like to use for backend testing, and it typically doesn’t matter. However, it’s easier to troubleshoot when all the devs on the team are in agreement on this particular tool. You can share your endpoint tests with other devs, which helps everyone find issues faster and more consistently. Eventually, your backend tests can serve as some of the documentation for the frontend. This will be something worth making a ticket for so that you can get it included in a sprint.

Now that you know the app is working, you can start making changes to get it ready for the team.

## Updating the README

Start by updating the _README.md_ with specific instructions for how to set up and run the app. There’s already a lot of good stuff in there, so you can trim it up a bit and add a few things. A basic update can look like this:

```
# Dashboard Server
```

This doesn’t have to be a huge doc that expounds on your philosophical thoughts about code and the project. Give enough guidance that any dev could clone this repo and get it running. Other docs for the project will be stored somewhere else. As the repo grows, adding more details to the README will be helpful. This is a living document that everyone should feel comfortable updating as part of their PRs when needed.

## Adding a CHANGELOG

As a best practice, it’s good to include a _CHANGELOG.md_ file in your repo. This will keep a close record of everything in each release over time. Having this record will help you determine which version of the app contains which updates; this comes in handy when there are issues with production. Here’s what the start of this file can look like:

```
# CHANGELOG
```

You might consider setting specific rules on commit messages. If you prefer a certain format, set up a tool like [commitlint](https://commitlint.js.org/#/) to enforce it early. This means your commit messages can include details such as whether it’s a fix or a chore. It can include ticket numbers if needed. You can also make it enforce things like the message starting with a verb to state what the change does.

The commit message might look like this:

fix: update modal to send API call once

You might be responsible for preparing the release PRs, so it’s important to make sure that updating this file gets included in that process. A good way to remember is when you update the app version in _package.json_, update this file, too.

In many projects, you may need to set up ESLint, Prettier, and TypeScript config files. This app comes with a lot of configs set by default, so you should go through them and add or remove values as needed. That’s all for the initial app setup. Now you can turn your attention to building the data schema.

# Monolith and Microservice Architectures

This is a crucial point in the project because you’re setting the foundation for how everything will be built for the foreseeable future. You’ll need to think about the future of the project, how it might grow, and what blockers that growth might run into.

Take some time to make an architecture diagram. It doesn’t have to be fancy, but it does need to represent how the backend interacts with different pieces of the entire system. This includes everything the app touches, ranging from the frontend to cron jobs. _Cron jobs_ are tasks that run in the background of your system on a schedule. They can be written in TypeScript as well.

The backend is like your control center. It drives the development of everything else because it interacts with a number of parts in the overall system. That’s why it’s so important to take your time and make this calculated decision. The two common architectures you’ll run into are monoliths and microservices. I’ll cover the differences here. Think of this as a guide to help you choose one or the other.

A _monolith_ is when all your backend functionality is contained in a single codebase. You’ll see this a lot with enterprise apps. There’s one database and one server for the application. It gets deployed as a single unit that is connected to all the services you work with. All the endpoints, maybe some cron jobs, and some third-party integrations will be handled here.

With a monolith, you get the benefit of all the code being in one place. That way, you can get exposure to how the entirety of the system works and where changes need to be made. Setting up the infrastructure is straightforward because you’ll be deploying only one app. Debugging is also easy because you can check the backend holistically.

Building a monolith is a simpler approach, but it comes with drawbacks. For example, if a bug gets released to production and causes errors at runtime that didn’t happen locally, the whole app is down for users. It can also be hard to scale the app appropriately. You might have only one endpoint that gets a large amount of traffic, but with a monolith, you have to scale the whole backend, which can cost a lot. [Figure 2-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch02.html#architecture_diagram_for_a_simple_monol) shows an example of what a monolithic backend can look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0202.png)

###### Figure 2-2. Architecture diagram for a simple monolith

A _microservice_ is the complete opposite of a monolith. If you have a directory or set of files dedicated to one piece of business logic in your monolith, it becomes its own API in the form of a microservice. With microservices, all the backend business logic is split into different chunks based on functionality. Each microservice has its own codebase, database, and server.

One of the biggest benefits of microservices is scaling. Since all the business logic is separated into specific entities, you can ramp up resources for one endpoint while keeping the others the same. A unique thing you can do is use different programming languages for different microservices. You might have a microservice written in Rust for concurrency handling, a microservice written in Python to serve a machine learning model, and a microservice written in TypeScript because that’s the dominant language used by the team.

Microservices give you more flexibility throughout development and deployment. There are a few drawbacks, though. Debugging microservices can get tricky. It may be hard to figure out the source of an issue when you have to sift through all the microservices you have running. The issue of data integrity may also come up if you have references to the same data across multiple microservices. [Figure 2-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch02.html#architecture_diagram_for_simple_microse) shows an example of what a microservices backend can look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0203.png)

###### Figure 2-3. Architecture diagram for simple microservices

# Alternative Architectures to Consider

Another architecture you’ll hear about is _serverless architecture_. This can be used to handle all the backend functionality, but you have to be aware of the differences because it can make a huge difference in your cloud provider bills. If you need a few lambda functions, for example, to handle very specific behavior, this could be a good addition to your backend design. You might see serverless and monolithic architectures used together as the product grows.

Before microservices became a thing, there was _service-oriented architecture_ (SOA). The main difference is that with SOA, all the separate services are sharing the same database. This can still be a good approach if your app gets complex enough that it’s a good idea to break out functionality into smaller chunks code-wise to keep things maintainable. I’m not going to dive into the [details of SOA](https://aws.amazon.com/what-is/service-oriented-architecture/), but you can consider if it should be used as part of your architecture.

The beautiful thing about architecture is that you can take the bits and pieces that work for your use case and put them together. As you work with microservices, you might hear about service meshes, too. I encourage you to [learn more about service meshes](https://www.redhat.com/en/topics/microservices/what-is-a-service-mesh) and how they are implemented, but I won’t cover them here.

Remember that the process of designing a backend will take time. This isn’t something that should be rushed through, and you should seek out as much feedback as possible from the other developers and teams you work with. Selecting any combination of these approaches cements the future development of the project because it’s not an easy task to switch the entire architecture to something else.

# Selecting an API Design Pattern: REST and GraphQL

There’s something else you need to decide on during this phase of setting up the project: are you going to go with REST APIs, GraphQL, or something else? This will determine how the frontend and other services are able to interface with your backend as well as how you extend your code as new roles, permissions, and other features are added over time.

REST is the standard in backend development at the moment, and GraphQL has become a close second. There are other options, like Simple Object Access Protocol (SOAP) APIs, but hopefully you won’t have to work with those.

The main difference between REST and GraphQL is that GraphQL uses a single endpoint to handle all requests while REST uses multiple endpoints for requests. You may hear people talk about these paradigms like they’re very different, but they really aren’t. It’s more a difference in the way you can build the APIs and consume them. One thing to highlight is that GraphQL comes with subscriptions built in. The only way to handle that with REST is with [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) or a similar technology.

With REST APIs, you typically have one type of resource per endpoint. Take the `orders` endpoint in your project, for example. You have GET, POST, and PUT requests for the different ways you need to interact with your order data. If this were a GraphQL API, the endpoints would become your GraphQL schema like this:

```
type
```

With GraphQL, you can specify the operation you want with the `query` or `mutation` keyword instead of an HTTP method. There’s no difference in the data that you’ll receive compared to your HTTP requests. The GraphQL response will still be in JSON format with the same values as an HTTP response. The difference is in how you manage the code.

When you write HTTP requests in TypeScript, you specify an endpoint path, a method, and any data you need to send in the request. With GraphQL, you have more flexibility with your requests. Using queries and mutations, you can write resolver functions instead of endpoint paths. Here’s an example of what resolvers might look like for your orders functionality:

```
{
```

So when you’re ready to make a request, the query would look like this:

```
query
```

This is where some of the power of GraphQL comes in. You can query for the exact data you want without returning values you don’t need. When you make HTTP requests, you can’t ask for one or two values to be returned. You get all the data from that endpoint every time. GraphQL can save on response times for systems that handle massive amounts of interrelated data because you can ask for exactly what you need.

This book isn’t going to do a deep dive into GraphQL, but I wanted to make sure that you know this option is available and it’s widely used across the tech industry. You can think of GraphQL as the next evolution after REST. It’s like when TypeScript came along to become the standard over JavaScript. It’s worth taking some time to learn more about it, so check out [some resources directly](https://graphql.org/). You’ll find all kinds of communities, classes, tools, and docs based around GraphQL.

# Conclusion

In this chapter, you learned about the NestJS app you’ll be setting up to handle your backend. You learned about a few different architectures and API paradigms and their trade-offs. Now that you’ve done the initial setup for the NestJS backend and you know why each decision was made, it’s time to turn your attention to the data schema.