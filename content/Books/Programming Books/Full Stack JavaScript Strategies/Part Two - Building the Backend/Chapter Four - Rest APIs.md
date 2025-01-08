---
id: 01JGFP12DNKDMJW10AMZZ93R99
modified: 2024-12-31T20:00:18-05:00
---
# Chapter 4. REST APIs

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 4th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

With the data schema defined, it’s time to get into the details of exposing that data. You’ll build an API with multiple endpoints that perform various operations, such as getting all the orders or submitting new ones. This is one area of the backend where we’ll get very into the details.

To create an API that can be maintained by different developers as the team changes, you need standard conventions for the team. Standard conventions are usually outlined in a document that defines the way the team agrees to approach code implementation, ranging from everything to naming conventions to the error codes and messages used in responses.

This set of conventions will make it easier to check for deviations in PR reviews for any code changes. Your PRs are changes you submit for other developers to review before they get merged with the central code and deployed. Since there may be multiple PRs for a piece of functionality, this rule enforcement is important for maintaining code consistency across codebases.

This chapter will go over how to address this convention while actually building an API, covering these areas:

- Working through data formatting with the frontend and other consuming services
    
- Writing an example of code conventions
    
- Writing the code for the interface, service, and controller for the API
    
- Tracking errors with logs and custom error handlers
    
- Ensuring that validation is in place
    

These are some of the concerns that will come up as you build out your API and the different endpoints. You’ll run into a lot of different approaches to API development and architecture decisions, and most of them are valid. As with everything else, it depends on the needs of your project and the team’s preferences.

For this API, we’re going to follow these conventions:

- It will send and receive data in JSON format.
    
- Endpoint logic shouldn’t reference other endpoints to keep separation of concerns.
    
- Use endpoint naming to reflect relationships of data and functionality (e.g., _/orders/{orderId}/products_).
    
- Return standard error codes and custom messages.
    
- Version the endpoints to gracefully handle deprecation.
    
- Handle calculations, pagination, filtering, and sorting on the backend.
    
- All endpoints receiving data should have validation.
    
- Endpoint documentation should be updated with all changes.
    

# Making Sure the Frontend and Backend Agree

There is always a partnership between the frontend displaying data and the backend processing it. When the frontend has to make multiple calls to fetch data, that can make the view render more slowly for a user. When the backend sends more data in responses, that can lead to unnecessary information being gathered and sent, which makes the response take longer. This is a trade-off you have to balance.

Typically, any type of pagination, filtering, sorting, and calculations should happen on the backend. This is because you can handle the data more efficiently on the backend compared to loading all the data on the frontend and making it do these operations. It’s very rare that you’ll ever want to load all the data for an app on the frontend. Generally, the engineers work together to come up with a way to handle data that provides a good UX.

This might mean introducing a microservice into the architecture to help both the frontend and backend performance. If there’s a specific endpoint that is called much more than the others, it’s worth researching if it makes sense to separate it out into a microservice and what type of work that would involve.

One area where you might have to push back is how endpoints send data to the frontend. For the frontend, it may make more sense for multiple pieces of data to get sent back on the same endpoint to help with performance and reduce the number of calls made. If it crosses data boundaries, then you need to double-check if the data should be combined. Data boundaries are how we keep a separation of concerns between different functionalities [Figure 4-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch04.html#example_of_data_boundaries_and_how_they) is an example of keeping a boundary between any products and orders calls.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0401.png)

###### Figure 4-1. Example of data boundaries and how they don’t cross

Remember, the frontend can filter the data out in the view, but anyone can check the developer tools in the browser to check the network response. It’s always the responsibility of the backend to enforce security when it comes to data handling. The frontend can have some security implemented, but users can bypass the UI and use the endpoints directly. We’ll address some security concerns in [Chapter 8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch08.html#backend_security_considerations), but this is one reason why you want to enforce data boundaries.

Now that the engineers understand the expectations of the frontend and backend, let’s work on the conventions doc for the backend.

# Creating a Document for Conventions

You can tighten up your code conventions even more with a doc that you share with the team and use to help new devs onboard with the team’s code style. This doc will evolve over time as you encounter new scenarios. It serves as a resource for the team when a new endpoint or even a new API needs to be created. That way, everyone knows how to build the code so that consistency is maintained everywhere.

Tools like [Prettier](https://prettier.io/), [ESLint](https://eslint.org/), [EditorConfig](https://editorconfig.org/), and [Husky](https://typicode.github.io/husky/) can help you automatically enforce the conventions with every PR. Sometimes, nitpicks like spacing, tabs, quotation marks, and tests can get tedious to check for manually. Using these types of tools requires devs to meet the conventions before a commit can even be made. Now every PR will have these little checks, which makes reviews faster and code more consistent.

The following are examples of some conventions:

- Microsoft Azure: [RESTful Web API Design](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
    
- [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)
    
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript?tab=readme-ov-file#airbnb-javascript-style-guide)
    

Docs like these can be starting points for your in-house conventions that you extend to match team preferences. For example, your conventions doc might contain an additional section like the following:

—-
For pagination, responses should return this structure:
{
 “page”: [
    {
      “id”: 4,
      “first_name”: “My first name”,
      “last_name”: “My last name”,
      “email”: “myemail@server.com”
    },
    {
      “id”: 5,
      “first_name”: “My first name”,
      “last_name”: “My last name”,
      “email”: “myemail@server.com”
    },
    {
      “id”: 6,
      “first_name”: “My first name”,
      “last_name”: “My last name”,
      “email”: “myemail@server.com”
    }
 ],
 “count”: 3,
 “limit”: 3,
 “offset”: 0,
 “total_pages”: 4,
 “total_count”: 12,
 “previous_page”: 1,
 “current_page”: 2,
 “next_page”: 3,
}

For error handling, responses should return this structure:
{
  “errors”: [
    { “statusCode”: “111”, “message”: “age must be an int” },
    { “statusCode”: “112”, “message”: “email is mandatory” }
  ]
}

—-

This is another way you can get feedback from others and make the convention doc the best it can be for everyone.

###### NOTE

Some things that you’ll set in your conventions will be annoying to stick to sometimes, especially during a time crunch. That’s why it’s important to implement a few features and see how things go in practice before you really enforce conventions through linting and other scripts. Once your conventions are in a good place, though, don’t deviate from them unless there’s a significant change to the direction of the project. The consistency throughout your codebases is what will make projects easier to maintain over the long term.

# Making the API and First Endpoint

As discussed in [Chapter 2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch02.html#setting_up_the_backend), NestJS is the framework you’ll build with. We won’t go through writing all the code tutorial-style here. I’ll address underlying reasons for why functionality is implemented a certain way. This is the type of thinking that spreads to any framework. I’ll leave it up to you to look through the NestJS docs to understand the syntax of the code.

There are so many things to consider when you start coding on the backend. The secret is to just pick an area and focus on it first. You’ll come back through and address security, performance, and testing concerns. For now, though, you need to get some endpoints working, so the frontend can start connecting the UI to the API. You’ll start by writing the basic CRUD operations you know the app will need. In this project, you’ll need CRUD endpoints for:

- Managing products
    
- Managing orders
    
- Administrative functions
    

The first two sets of endpoints are based on what you already know about the app. The last set of endpoints will come from other discussions with Product. There will be actions the Support team will need access to that no user should ever be able to touch. You’ll learn how to handle these different user permission levels and access control in [Chapter 8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch08.html#backend_security_considerations). Remember, these endpoints will likely change. The main thing is that you have to start building somewhere.

###### TIP

You can delete some of the boilerplate files, specifically _app.controller.spec.ts, app.controller.ts_, and _app.service.ts_. Also go ahead and update _app.module.ts_ to remove the references to those files. This is to keep things as clean as possible as you start to make changes and add new code. Those files are just there as examples of how to implement endpoint functionality. It’s normal to remove some of the example files when you use a scaffolding tool so that you only have what you need. This is something that will come from experience as you work with scaffolding tools and start to understand which files can be removed and which ones are useful starting points.

## Creating the Orders Endpoints

You can start by working on the functionality for orders. In the _src_ directory, make a new subfolder called _orders_ and add the files to handle the types for this endpoint, the tests, the service, and the controller.

The _orders.controller.ts_ file is where you define all the endpoints for this specific feature. You can learn more about [how controllers work](https://docs.nestjs.com/controllers) in the NestJS docs. So anytime you need to fetch orders or make changes to them, the frontend will reference the endpoints here. This is a great place to do your initial validation on data received in requests. Here’s an example of an endpoint:

```
// orders.controller.ts
```

Here you can see some custom error handling, and it sends a message and status code like you defined in the conventions. The controller shouldn’t contain any business logic because that will be handled in your service. Controllers are just there to handle requests and responses. This makes the code more testable, and it keeps the code separated based on what it should do.

Controllers will also do some of that validation I mentioned. Let’s take a look at an endpoint that will update orders:

```
// orders.controller.ts
```

Note how the `try-catch` statement is used for both the endpoints. This is a clean way of making sure your errors are handled. The code in the `try` block is always run first. If any errors happen in this block, the `catch` block will be triggered. Then, you can focus on how to handle the errors that are caught. This is something that can get overlooked when devs are in a hurry, so it’s a prime candidate to include in your code conventions.

The validation here is happening through the `UpdateOrderDto`. Here’s what it looks like in _orders.interface.ts:_

```
// orders.interface.ts
```

###### NOTE

DTO means _data transfer object_. DTOs are used to encapsulate data commonly used by the service layer on the backend to reduce the amount of data that needs to be sent between the backend and frontend. In a Model-View-Controller (MVC) framework like NestJS, these are also useful as the models for application. They mainly define the parameters passed to the methods on the backend.

NestJS handles validation under the hood with the [`class-validator`](https://github.com/typestack/class-validator) [`package`](https://github.com/typestack/class-validator), so if the `userId` is empty or the `total` isn’t a number, an error will be thrown to the frontend telling it the body data was in the wrong format. Sending proper validation messages to the frontend will help the devs know what to do and give users helpful information.

## Working on the Orders Service

The last file is _orders.service.ts._ This is where the business logic for the orders functionality is handled. Refer to the NestJS for more details on [service files](https://docs.nestjs.com/providers#services). Any calculations, sorting, filtering, or other data manipulation is going to happen in this file. Here’s an example of a method to update an order:

```
// orders.service.ts
```

Now you’re adding even more backend best practices and following the conventions because you have error handling and [logging](https://docs.nestjs.com/techniques/logger) happening here. There will also be errors that come from the service level, which is why you have the `try-catch` statement to bubble those errors back up to the controller.

You should also add logging like this in your controllers. One thing you’ll find is that logs are invaluable when you’re trying to debug the backend. Make your logs as descriptive as you need to in order to track values across your database and other endpoints or third-party services.

Regardless of the framework you decide to use on the backend, you’ve seen the core things you need to implement: validation on inputs, logging for the crucial parts of the flow, and error handling for issues that may arise. As long as you remember these things and keep the code conventions in mind, your team is on the way to a strongly built codebase.

It’s time to look at some other parts of the backend that will ensure that the endpoints and services work as expected.

## Checking the Database Connection

Many projects have a folder called _utils_ or _helpers_ or something similar. You’ll need to create one of those folders to hold the service you’re going to use to instantiate Prisma Client and connect to the database. This was discussed in [Chapter 3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#building_the_data_schema), so now you’re expanding on that base you already have. In the _src_ folder, make a new folder called _utils_. In this folder, create a file called _prisma.service.ts_ and put [this code from the NestJS documentation](https://docs.nestjs.com/recipes/prisma#use-prisma-client-in-your-nestjs-services) in the file.

You don’t have to worry about writing everything from scratch most of the time if you spend a few minutes reading and looking through docs. That’s a thing you’ll find senior devs doing all the time. Also, don’t be afraid to add more things to this _utils_ folder! When you see small functions that are repeated in numerous parts of the app, like data formatters, move them here so that they are easy for other devs to find and use.

If you haven’t stopped to make a Git commit, this is a good time to do so. Now you have the backend in a state where other devs can come in and add more functionality or configurations. One of the hardest tasks is to set something up that others can improve. That’s what you’re doing right now.

As you build on this application throughout the book, you’ll start to add calls to third-party services, handle data from different sources, and work on security concerns. All of these will involve endpoints and other service methods that you’ll add on as you move through the tasks on your sprint.

It’s important to get some practice in, so try to add error handling, logging, and validation to the remaining endpoints in the orders controller. Of course, you can always check out the [GitHub repo](https://github.com/flippedcoder/dashboard-server/tree/main/src/orders), too.

# Conclusion

We covered a lot in this chapter, and we will dive even deeper in [Chapter 5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch05.html#third_party_services)-[Chapter 10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#backend_performance)! The main takeaways from this chapter are making an agreement with the engineers working on the frontend, setting up strict conventions for your API as soon as possible, creating some initial endpoints to get the frontend devs moving, and error handling, validation, and logging. Now that you have a few endpoints up, you can start building on top of them.