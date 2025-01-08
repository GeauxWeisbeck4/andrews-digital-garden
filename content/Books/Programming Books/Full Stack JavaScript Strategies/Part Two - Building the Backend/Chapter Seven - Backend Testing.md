---
id: 01JGFP40KV0QACP74P50N6SKG5
modified: 2024-12-31T20:01:54-05:00
---
# Chapter 7. Backend Testing

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 7th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

At this point in the project, you’ve got quite a bit of code. There are services, controllers, and integrations. While you and the team will do your best to not introduce code _regressions_ (where previously working functionality is broken from an unrelated change), that will happen at some point, and it’s normal as the code grows. That’s why you write tests for the major functionality of the app.

On the backend, you’ll test for things like errors being called in the correct scenarios, data being returned in the correct format, and the correct methods being called with the correct parameters. Writing unit tests like these will help keep you from making regressions, understand the way the code should work, and make the code maintainable because you are writing more concise, modular code.

In this chapter, we’ll cover:

- Trade-offs between having tests and not having them
    
- How to write tests using Jest
    
- The importance of mock data
    

Regardless of whether you are starting a project from scratch or inheriting one, tests can be a great help. This is also a great time to collaborate with the Product team on the current and future state of the product. Writing tests will often bring up more detailed questions about exactly what is expected from some functionality.

# Why Spend the Time on Tests

You’ll find that developers have differing views on how tests should be implemented. Some strong opinions can come up here because, just like with everything else, there are trade-offs. A major consideration is the level of test coverage for a project. I’ve seen many projects where getting close to 100% test coverage on the backend isn’t far-fetched, but aiming for 90% or higher is more reasonable. There comes a point when you and the team have to decide if more tedious tests are worth writing by looking at the value they add compared to the amount of time they take to write.

When you take the time to write tests, it does slow release cycles initially. Tests can take a while to run in your CI and take time away from feature development or refactoring. I think that the benefits outweigh the costs, but you have to maintain a balance between them. While you may sacrifice some speed in the beginning, tests can prevent a lot of headaches and give the team more confidence in their code as the app grows. Having tests improves the developer experience for the team because you all can make changes without as much worry about regressions and things breaking in unexpected places.

Tests give you a chance to think through and document the way the code should work. They can even bring up more product-related questions. You may run into cases where tests lead to questions that shake up your architecture. Having automated regression tests in place also helps you keep preventable bugs from getting to production.

This is a good place to pair with the QA team, if you have one. If you don’t, then you’re building in some level of QA for your team. Most of the time, the dev team will be focused on writing unit tests. _Unit tests_ are what you use to make sure the code is implemented properly and running as expected for different use cases. The QA team can come in and write some automated end-to-end (e2e) tests to run against your development environment. An _e2e test_ is more focused on testing the user flow than the code. Some good tools for this are [Postman](https://www.postman.com/) and [Thunder Client](https://www.thunderclient.com/). You can write collections of tests to hit your development environment to make sure endpoints are really working as expected.

Having a way to validate that your endpoints are manipulating data correctly becomes a necessity as the product grows and new business rules come up. Sometimes things can slip through manual testing as people come and go. Having test cases set up for both unit tests and e2e tests is a form of documentation that is kept up to date consistently.

Whenever a new change breaks a test, you can see what the initial assumptions were and then double-check with the dev or Product team that those assumptions are still valid. Again, writing tests with this level of thoroughness can slow releases because you have to account for the time it takes to write the tests _and_ get them to pass. So it’s important to consider how long this takes in your ticket estimates.

Implementing code and deploying it without tests can be useful when you need to get features and bug fixes out really fast. The trade-off is that you don’t have the assurance that you didn’t break something seemingly unrelated. For example, you may update the data schema for products and rename a field. This can break things you wouldn’t expect, like functionality for retrieving orders.

You may think you can come back and add tests later as long as you get this next release out on time. This is a risky assumption to make. It’s best to write the tests along with the implementation so that you don’t miss anything before releasing. Plus, tests tend to get pushed to the backlog as bugs and new features come along.

Work with the Product team to help them understand why writing tests is helpful for getting a better product to the end user. Show them how it will help prevent the need to do hotfixes for unexpected regressions. Once they see the value that tests add, they will start asking about different scenarios that will help you write even better test cases.

# How to Approach Test Writing

Although approaches like test-driven development (TDD) or behavior-driven development (BDD) can be helpful, you don’t need to follow them. The important thing is to make sure you know what to write tests for. That’s a big part of testing that tends to get left out in most courses or blog posts. When you write unit tests on the backend, you’re checking for specific functionality.

When you make a request to an endpoint, what exactly is supposed to happen? You may need tests for the potential errors that could be returned in the response. It also helps to make sure that the functions you expect to get called actually do, they get called with the correct parameters, and they return the expected data.

Go through your code line by line. Is an error being thrown? Write a test for it. Is a function being called? Write a test to see if it gets called with the correct parameters. There’s a balance between knowing when to mock functions or letting the actual function be tested. If you find that you’re trying to test a package you’ve installed, then that’s a good case for mocking the functionality. If it’s internal code that you and the team have written, it’s usually OK to let the tests call the actual code. By having these tests to start with, you can save yourself a lot of debugging time when you and other devs start adding new functionality and code gets refactored.

Any time a change is made that makes the tests fail, that will help you get to the root cause faster. This also helps you think through the way the code should work as it grows over time. You may even find areas where the logic can be improved as you handle failed tests and think about how the code should really work. Your tests should focus on where the logic is, such as conditions in your code, when HTTP status codes will be returned, how user permissions affect the results of API requests, and anything else that will change the outcome a user or service receives.

Let’s take a look at the `create` method in _orders.controller.ts_ and see how you would write unit tests for this:

  `@Post``()`
  `public` `async` `create``(``@Body``()` `user``:` `User``,` `order``:` `CreateOrderDto``)``:` `Promise``<``Order``>` `{`
    `if` `(``!``user``)` `{`
      `throw` `new` `HttpException``(``‘``Unauthorized``’``,` `HttpStatus``.``UNAUTHORIZED``);`
    `}`
    `if` `(``!``user``.``permissions``.``includes``(``‘``create``:``orders``’``))` `{`
      `throw` `new` `HttpException``(``‘``Forbidden``’``,` `HttpStatus``.``FORBIDDEN``);`
    `}`
    `if` `(``!``order``)` `{`
      `throw` `new` `HttpException``(``‘``No` `order` `data``’``,` `HttpStatus``.``BAD_REQUEST``);`
    `}`
    `if` `(``order``.``products``.``length` `===` `0``)` `{`
      `throw` `new` `HttpException``(``‘``No` `products` `in` `order``’``,` `HttpStatus``.``CONFLICT``);`
    `}`
    `if` `(``!``order``.``total``)` `{`
      `throw` `new` `HttpException``(``‘``No` `order` `total``’``,` `HttpStatus``.``CONFLICT``);`
    `}`

    `try` `{`
      `const` `newOrder` `=` `await` `this``.``ordersService``.``createOrder``(``order``);`
      `return` `newOrder``;`
    `}` `catch` `(``err``)` `{`
      `throw` `new` `HttpException``(``‘``Something` `happened``’``,` `HttpStatus``.``NOT_FOUND``);`
    `}`
  `}`

You can see the errors that may get thrown before making the call to the service method. If any of those conditions are met, then an error will be sent to the frontend in the response. You can be explicit in the error message or leave it more generic. As long as you aren’t revealing any PII data or anything specific enough to give attackers useful info, you can say just about anything.

It’s a good practice to keep the message short and send the associated status code. Now you can look at this from a testing perspective and go through the method line by line. By reading the code, you can identify five test cases from the errors being thrown, a test case for a successful service call, and a test case for an error being thrown from the service. Let’s look at how you would write these seven tests with Jest:

```
import
```

###### TIP

This is a great time to mention some popular AI tools currently available that could help you write tests, like [Copilot](https://github.com/features/copilot) or [ChatGPT](https://openai.com/chatgpt/). Tools like these can help speed up your process for writing tests or generating scenarios.

All these tests came from scenarios described in the controller code. When you’re writing tests in general, you typically don’t need to worry about testing the implementation for things like helpers and service functions because they will have their own unit tests. That’s one of the benefits of a modular approach like this. It makes writing tests clearer because you have a separation of responsibilities.

For each scenario, you have test data, and you have the expected response. This is a good way to ensure that you get the main cases. While you should try your best to anticipate edge cases when you talk with Product and QA, you can also add edge cases to these tests as you find them.

###### TIP

One thing that helps me a lot when writing tests is to have the test file and the code file open side by side. That way, I can see if I’ve tested each line of code as I move through the file.

There are a few ways to approach e2e API testing. You can use a tool like Postman because it’s friendlier to work with for nondevs. Or you can use a tool like [Cypress](https://www.cypress.io/) or [Playwright](https://playwright.dev/). [Figure 7-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch07.html#etwoe_tests_in_postman) shows what the tests would look like in Postman. You can import the [JSON file for these tests](https://github.com/flippedcoder/dashboard-server/blob/main/test/product-api-e2e.postman_collection.json) into Postman and check out the tests yourself. In [Chapter 24](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#integration_testing), I’ll go over running e2e tests in your CI/CD pipeline.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0701.png)

###### Figure 7-1. e2e tests in Postman

###### NOTE

_E2e testing_ is an ambiguous term—it depends on whom you talk to about it. Some would argue that e2e tests are only when you test both the frontend and backend together. Others will say that e2e tests can be divided between the frontend or backend if they focus on functionality instead of code implementation, similar to unit tests. Just keep in mind that there isn’t a specific definition for e2e tests that’s more correct than another. You have to understand that the distinction may depend on the organization.

If you decide to go the programmatic way with something like Cypress, the tests will look something like this:

```
it
```

So the syntax is comparable to what you’ll be writing your unit tests with. With the programmatic approach, make sure to factor in the time it will take for the team to get ramped up on how to use a particular library. But the scenarios will be very similar across your unit and e2e tests. The unit tests will check code implementation and data manipulation, and the e2e tests will check for the flow of the data at a higher level similar to what the frontend or another service would do. You’ll learn more about e2e testing in [Chapter 24](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#integration_testing).

# Mock Data

You may have noticed that there was some mock data in a few of those test cases. Your mock data should account for different scenarios that can happen with the project. For the highest level of documentation, it helps to have the mocks in separate files with very specific names, although sometimes the difference between one test case and another is a single field. This is where some of those code conventions might need some updates. As long as the team agrees on the best approach for the scenario, then that’s what works.

You can get the mock data directly from your database or by using a tool like [Faker](https://fakerjs.dev/guide/) or [Falso](https://github.com/ngneat/falso). This is a great time to double-check your types as well. As you make this mock data, work with the Product team to decide what should happen in different scenarios. It’s OK to make assumptions, but verify that they are valid for what’s expected of the product.

Something else you should consider with your mock data is creating test data in your database. This is how you can have mock data available for your e2e tests, which is great for demos to the Product team and any other stakeholders because you can go through the entire flow without having to do code hacks. It also gives them a chance to test things for themselves and see if anything needs to be changed.

# A DIFFERENT TAKE ON MOCKING DATA

Here’s another great tidbit from Ethan Brown:

> I think having a robust mock data infrastructure is an underrated principle in software development. Most real-world applications deal with all these very specific data types, and developers are always spending time reinventing the wheel of test data for every little thing they do. Having robust and realistic test-data factories—indeed, building it into the very process of class/type creation—can be a game changer for productivity.
> 
> For example, if I have a dev starting on some piece of functionality involving a “user” object, I don’t want them to either (a) create a bare-bones user object that might miss something that would break in production or (b) spend a lot of time mocking up user objects. I’d rather they just call `FakeUserFactory.create(10)` to get 10 users as if they had been created in the real world. This also has follow-up benefits for product demos, trial accounts, or simulations.
> 
> It also helps having some robust test data to choose from that covers several real-world situations. Even if you just have a file with a list of 10 or 15 fictitious users and devs get to pick which one(s) they want to use for testing, it may remind them to handle messy real-world data.
> 
> In the case of users, I would have a user with a ridiculously long name (“Sir Charles Xavier Tester Testalot III”), one-word names (“Prince”), and names with non-Latin characters (“Dvořák”). As a junior dev looks through that list, even though they were just looking for some test data, they might think, for example, “Oh shoot, I just did something that’ll break if there isn’t a last and first name.”

A good way to start mocking data is to make a complete object for each model. For example, you made the user mock data here:

```
const
```

This has everything you need based on the user schema. Now if you need to modify fields, you can just update this object in the test to account for that specific case. You have an example of how this works in the user permissions test case where the `permissions` field is modified.

###### NOTE

Another good way to start mocking data will actually be from your database seed data! This is a good time to note that your database seed data should be updated along with any schema changes. That way, when you need to set up the project on a new computer, you won’t run into missing data or incompatible types.

# Conclusion

In this chapter, we covered testing on the backend and why it’s important. You’ll run into some developers who haven’t written tests with this much detail before, and they might push back. Just keep in mind that every time you deploy a bug and it gets found, you cause a mini panic within the organization and create the need for hotfixes. Tests help you avoid that even if they do take extra developer time. We’ll get into deployments and integrating tests there in [Chapter 24](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#integration_testing) and [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments).

We also covered several different ways to test your code. Unit tests are there for the details in the code implementation, and e2e tests are there to make sure the flow works as expected from a user standpoint. You can even say that any testing done by the Product team counts as user-acceptance testing because they are the ones checking functionality from a user perspective. Testing is a group effort, so a lot of discussion with the Product team and any other stakeholders may be necessary to make sure everything makes sense.