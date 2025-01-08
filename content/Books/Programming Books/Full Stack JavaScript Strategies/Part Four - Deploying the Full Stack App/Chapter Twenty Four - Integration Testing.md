---
id: 01JH1NXAME9AVY9JRAX353121J
modified: 2025-01-07T19:44:35-05:00
---
# Chapter 24. Integration Testing

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 24th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

One of the best things you can do for the long-term maintainability of your apps is to write integration tests. When I discussed backend and frontend testing in [Chapter 7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch07.html#backend_testing) and [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing), I mentioned how we would go into greater details on these tests. E2e tests are a great way to ensure that the changes you make don’t break the full stack functionality. This is incredibly helpful when you do things like upgrade packages and make changes to global components so that you can catch unintended side effects. That saves QA time on regression testing, and it helps prevent bugs in unexpected places from getting to users.

These tests also help document how features should work as the app grows. If you change something, such as what happens when a button gets clicked, these tests are more robust than unit tests. E2e tests can be used to perform actions like a user so that you get the real flow consistently. That’s the biggest difference between unit tests and e2e tests and why they give you more assurance that functionality hasn’t changed.

The e2e tools that I’ll talk about in this chapter are generally stack agnostic, so they aren’t limited to just React projects. I’ll go over writing the same tests using three of these packages so that you can compare how they work:

- Cypress
    
- Nightwatch
    
- Playwright
    

E2e tests typically take more time to write than unit tests, so while they offer more thorough flow testing, they are more expensive to develop and maintain. To keep the e2e test cases manageable, it helps to define them during feature development or product roadmapping.

# The Test Cases

Before we jump into the code, let’s start by defining the three test cases we’ll write with each of the tools.

The first test case is making sure the order table loads. This functionality will require a few responses from different endpoints, waiting on a loading state to change, and waiting for the data to load in the table component. This feature has a lot of moving pieces, so the Product team will likely help you write out the scenarios. Maybe they’ll use [Gherkin](https://cucumber.io/docs/gherkin/reference/), like we discussed in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing).

The second test case is to check if an order has been submitted correctly. In this case, you’ll check that the inputs have valid values, the endpoint is called correctly, and you get the expected status code once the endpoint sends the response. This will help ensure that when users try to submit orders, they can do so successfully.

The third test case is making sure a user can’t submit an incomplete order request. Now you’re checking that the correct error message shows on the page if one of the inputs has an invalid value. This scenario helps ensure that users are provided with relevant, actionable feedback if they type in something wrong.

Now that we have the test cases, let’s write the tests for them with the different e2e tools.

# End-to-End Tests with Cypress

The first e2e tool I’ll cover is [Cypress](https://docs.cypress.io/guides/overview/why-cypress). This is one of the most widely used tools for e2e testing, and it uses your app in a real browser the same way a user would. Think of it as like giving a person instructions on how to do something in your app. It has methods that let you target components on the page, and it will call your real APIs. You can also mock the API calls if you have dummy data that will be returned to keep consistency in the tests, but remember that takes away from the e2e part of these tests. To get started, go ahead and install Cypress if you didn’t in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing):

npm install cypress --save-dev

You’ll also need to add some configs to your _cypress_ directory by updating your _tsconfig.json_ file. Take a look at the [Cypress docs](https://docs.cypress.io/guides/tooling/typescript-support) to determine what you need to add because these configs tend to change often.

You already have some tests in _cypress/e2e/user-info.cy.ts,_ and we’ll add on to them. The first thing you need to do is refactor the tests with a `beforeEach` hook because this will automatically run repetitive code for you instead of having it in each test. You’ll be making three API calls to get orders, get users, and post an order, so those can be mocked and intercepted for each test. Here’s how you implement that:

```
// user-info.cy.ts
```

Next, you can add the first new test to the _user-info.cy.ts_ file below the one where you click the “Actions” link. This test will check that your orders table is loading correctly, that you’ve called the correct endpoint, and that the data has finished loading. Keep in mind that the way I’m showing is just one of many ways to write any of these tests:

```
it
```

This uses the `getOrders` intercept you defined in the `beforeEach` hook. You get the orders table based on its `aria-label` and check it for one of the expected product names to make sure the data was returned from the response. Then you double-check that the loading icon for the table isn’t visible on the page. Just a reminder: we added the [`data-testid`](https://github.com/flippedcoder/dashboard-web/blob/ch-18/src/screens/UserInfo/UserInfo.Container.tsx#L79) attribute earlier.

Now you can run the test to make sure it passes and you’re getting the expected results. If it runs and passes successfully, go in and change one of the assertions to make sure it’s not a false pass. Purposely make the test fail by changing the check for the loading circle to be visible to make sure the test is correct.

The next test is a little more involved because you get to fill in a form programmatically. You have to target all the inputs and then type valid values into them. After that, you have to find the Submit button and click it. Then you can check that the API requests return successfully by using the `createOrder` mock intercept you defined earlier. You also check to make sure the success message is rendered on the page. Here’s how you can write this test:

```
it
```

The last test is to make sure your form validation works correctly and doesn’t allow invalid form submissions. This will fill out all the form fields and keep the email in an incorrect format, which should display the form error on the page:

```
it
```

Now you have a set of test cases you can expand on. Try running these tests to see if they pass. If they don’t, take a look at the component code, the test code, and the Cypress docs to figure out where the issue is. Also try running these tests in your CI/CD pipeline to see how long they take.

# End-to-End Tests with Playwright

The next e2e tool we’ll implement is [Playwright](https://playwright.dev/docs/intro), another popular testing package. It has some similarities with [Testing Library](https://testing-library.com/docs/); Playwright has a [migration guide](https://playwright.dev/docs/testing-library) that shows you how they implement similar functionality. It automatically runs tests in Chromium, Firefox, and WebKit, so you have three browsers covered. The configs are a little different from Cypress. It’s all set up when you install and initialize the package with the following command:

npm init playwright@latest

You’ll see multiple prompts, and there will be a list of options you need to go through to set the [appropriate configs](https://playwright.dev/docs/test-configuration). Using the default values is fine for this example, but experiment with the other values to see what they change. With the initial setup done, you can go to the Playwright test file and rename it: _user-info.spec.ts._ Then add the following code to start your test file:

```
import
```

Similar to what you did in the Cypress test, you have a `beforeEach` and an `afterEach` hook that navigate to the local URL and set up your API mock. With Playwright, you use the built-in context for [API testing](https://playwright.dev/docs/api-testing). You can add specific headers and make this a utility object that you expand on for things like authorization and token validation. The _[_https://api.teststore.com_](https://api.teststore.com/)_ URL is the domain for the backend, but you can replace this with your own backend URL, especially if you’re running the e2e tests locally.

There’s one test case we wrote in Cypress in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing) that we’ll add to our tests just for thoroughness. The case is to check basic navigation to make sure a link works:

```
test
```

Then you’ll add the same three test cases in this file that you wrote for your Cypress tests:

```
test
```

Here, you set the context to return the mock response from the orders endpoint and then make sure the page loads with it. Next is an example of making a POST request with a form submission:

```
test
```

The form field inputs are similar to what you did in Cypress where you get the field and type the value you want. How you make the POST request and determine if the request was successful are a little different than Cypress.

The final test case is checking for the form field validation error on the email input:

```
test
```

This is the same assertion you made in the Cypress test with different syntax. Now you can run the tests with the following command and see if the tests are passing:

npx playwright `test`

[Figure 24-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#overview_of_test_results_from_playwrigh) is an example screenshot of the test results in the browser. Playwright generates an _index.html_ file that you can commit in your repo, or you can exclude it from the repo and still show it in your CI/CD pipeline.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2401.png)

###### Figure 24-1. Overview of test results from Playwright

The test results show you which ones passed and failed and the browsers they were tested in. If you click one of the test results, you’ll get a detailed view of why it failed or passed. [Figure 24-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#detailed_view_of_a_failed_test) is an example of what the details for a failed test might look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2402.png)

###### Figure 24-2. Detailed view of a failed test

Now you can go to the part of the test that’s failing and start debugging. Playwright has some [good debugging tools you can find in the docs](https://playwright.dev/docs/debug), such as breakpoints and verbose API logs. The breakpoints are similar to what you see in the browser.

# End-to-End Tests with Nightwatch

The last tool you’ll implement is [Nightwatch](https://nightwatchjs.org/guide/overview/what-is-nightwatch.html). This tool is based on [Selenium WebDriver](https://www.selenium.dev/documentation/), which is one of the oldest browser-automation packages around. Many testing tools are based on Selenium for multiple types of projects. You also have the option to run your tests on a remote Selenium server, similar to [Cypress Cloud](https://www.cypress.io/cloud). So if you or anyone on your team is familiar with Selenium, this is a good choice because those skills transfer. Another thing Nightwatch does well is integrate with [SauceLabs](https://saucelabs.com/) and other cross-platform tools if you need to test on a wider range of devices.

This command will install Nightwatch and take you through some setup questions:

npm init nightwatch

It’s fine to accept the default values for the setup question in your terminal. Again, take some time to explore the [config options](https://nightwatchjs.org/guide/reference/settings.html) in the Nightwatch documentation.

After you have the initial setup ready, you’ll have to install the following Nightwatch package so that you can test the API requests as with the other e2e tools:

npm i @nightwatch/apitesting --save-dev

Unlike the other tools, Nightwatch has a separate package to keep the package focused on one task because often you’ll want your e2e tests to hit your actual endpoints. In our case though, we’re using mock data.

Now you have to update the _nightwatch.conf.cjs_ file with a new plug-in and the `api_testing` object as described [in the Nightwatch docs](https://nightwatchjs.org/guide/writing-tests/api-testing.html). This was the last bit of setup you needed to do before starting on your test cases. You’ll create the same four test cases as you did in the previous examples with Cypress and Playwright and do some initial setup that will cover all the cases. Start with setting up the mock API server, the requests, and the data:

```
import
```

This does the same as the other test packages but with a different syntax and system under the hood. If you’re familiar with [Express](https://expressjs.com/en/starter/hello-world.html), this is similar to the way you would write API endpoints. That could be handy if you have team members who are stronger on the backend but do some frontend work and need to maintain code coverage for their changes. Then you can dive into the first test case of navigating to another page:

```
it
```

This syntax is similar to what you would write with a Selenium test because Nightwatch is also based on the [W3C WebDriver specification](https://www.w3.org/TR/webdriver/). This spec is what lets you write code across all browsers, which is great for an e2e tool because you need to test on multiple browsers. WebDriver doesn’t need to be compiled with your code, so just like with the other e2e tools, your automated tests run the same way as if a user were performing the actions.

The next test checks that the endpoint to get the orders data is being called correctly:

```
it
```

This accesses your mock API by checking if the `client` made a call to the route you’ve defined, and it checks the screen to determine if one of the ordered products has loaded.

The next test is for the form submission request:

```
it
```

You can see that the format is similar to the other two tools because you find an input and set a value to it. Then you find the Submit button and click it to check if the POST request was made successfully.

The last test is the invalid email submission to check if the error message is on the screen:

```
it
```

Now you’re ready to run these tests to see how they look when they pass or fail. One unique thing about running Nightwatch tests is that you have to specify the location of the tests in your command by default. You can set up a script in your _package.json_ to help automate the process for you and the team locally and in the CI/CD pipeline. Here’s the command that you can run to get the results from your test suite:

npx nightwatch nightwatch-tests

Then you’ll get messages in your terminal and an HTML version of the test report saved in your repo. Now you’re done trying out all these testing tools, and since you have written the same four tests with all of them, you can do some comparisons between them.

###### NOTE

One thing to note is that all of these e2e packages have been installed as dev dependencies. This is important so that your testing tools don’t get bundled into the code that gets served in production. Remember, having unnecessary packages in your bundle makes your app run more slowly on the server and in the browser, and it opens the app for more malicious attacks. You can use any of these tools to perform API testing if you want to use the same testing tools across the frontend and backend.

# Comparison Between Packages

Now you see how you can run e2e tests using different packages, how you set up the environment, how they run in your CI/CD pipeline, and how they help you check functionality from a user perspective. This is where you can really work with Product to nail down requirements and how to handle edge cases. A good rule of thumb is that if you struggle coming up with good names for tests, that means you might not be writing the best test cases. That’s something to keep in mind as you review PRs from other devs and continue adding more test cases.

When you compare Cypress, Playwright, and Nightwatch, a key metric you should look at is how long it takes the developers to write and maintain the tests. Sometimes one package is easier for the team to pick up than the others, and that’s a strong reason to choose it. If the team can write tests with one tool 10% faster than with another, that will save time over the long term as you have to update tests to address code and feature changes.

The time for the tests to complete in your CI/CD pipeline is another metric you should look at. That could be affected by the package size because the package will have to be installed before tests are run. Although if you notice that the test runtime for your app is similar across all the packages, then you should look at other metrics.

As with other packages, you need to decide which one of these has the best documentation for you and the team. Community support is also an important factor because that will determine how quickly issues are resolved and if you can find help when you run into the quirky aspects of any package. Take a look through the GitHub issues on these packages and see how well they were handled.

# THE TESTING PYRAMID

The “testing pyramid” is a cool concept that helps you decide how and when to implement different types of tests, such as unit tests compared to e2e tests. Here’s a brief explanation from Ethan Brown:

> The conventional wisdom is that you have the fewest e2e tests because they’re so expensive. I think tools like Cypress are slowly changing this conventional wisdom, but I think it’s a valid big-picture framework all the same: invest in a technology in proportion to its cost (in either dollars or person-hours) and value. We could argue about how Cypress is changing the cost-value ratio of e2e tools, but it’s harder to argue that the principle isn’t correct.

Cypress is arguably the most established e2e testing package in the JavaScript ecosystem, so it’s usually the first choice because more devs have had exposure to it. But if you and the team agree that you would rather use Playwright or Nightwatch, they are also widespread. Consider the long-term use of the package you choose. That will affect how quickly new devs can get ramped up on the project as the package you choose becomes a standard in your development process.

# Conclusion

Now you have everything you need to test the full stack of the app from the frontend to the backend and maybe a little bit of the database. This is something that QA engineers will usually handle, but if you’re working on a team without QA, now you can use your skills to get something in place. You should have an opinion on how to write tests, what tools to use, and why spending time on e2e tests along with unit tests is helpful. Remember, unit tests are more focused on the code handling specific scenarios whereas e2e tests are simulating scripted user interactions to make sure parts of your app are working as expected. This is a holistic look at your processes, and you aren’t concerned about the code behind it here.

Something else you can do with e2e tests is demo them for Product and stakeholders so that they can see how the tests run and the value they add. This can bring up more conversations for future work on the roadmap once the stakeholders see the automations run. Keep in mind that these tests are more time-consuming to write and maintain, so this might not be something you do immediately. But don’t wait too long because then you’ll end up with lots of features that need to be tested.