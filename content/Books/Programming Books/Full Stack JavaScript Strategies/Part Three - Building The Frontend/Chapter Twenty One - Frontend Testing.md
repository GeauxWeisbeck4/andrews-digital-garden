---
id: 01JH1NT0YNJKQ0J08MGRFGZMMG
modified: 2025-01-07T19:42:46-05:00
---
# Chapter 21. Frontend Testing

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 21 chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

In previous chapters, I mentioned that the best practice for writing any tests is to write them at the same time as you write any new functionality or do any refactors. Testing deserves its own focus, and that’s what I’ll cover here.

When you’re building this app, you need to make sure you aren’t releasing regressions in existing functionality. A _regression_ is when new code unintentionally causes errors in existing functionality anywhere in the app. The QA team, if there is one, won’t have time to run regression testing on every single release, but as a developer, you can take the initiative to ensure your code is solid. Your tests for the new code can bring up questions about how something works or what happens when it doesn’t.

In this chapter, I’ll cover:

- How to determine which parts of the frontend to test
    
- Unit testing
    
- End-to-end (e2e) testing
    
- Useful testing tools
    

The two goals of test writing are preventing unexpected broken code from ending up in front of users and documenting the app so that everyone knows how it’s supposed to work. Testing also gives you more confidence with future development because you aren’t worried about your changes breaking something unexpectedly.

Testing will bring your dev team, the Product team, the Design team, and the QA team even closer together as you all come up with different scenarios. Always remember that QA is not the enemy. Their job is crucial to deploying with confidence. They aren’t telling you that your code is bad, just that it doesn’t work as expected compared to the feature specs.

# Determining Test Scenarios

If you aren’t sure where to start, look at your components closely. Here’s a list of things to look for:

- Anything that is conditionally rendered should probably have a test.
    
- Anywhere you make API requests should probably have a test.
    
- If an error might occur, there should be a test for it.
    
- If you manipulate data in any way, a test should likely be written.
    
- Pretty much anything that causes a change in the way a component is rendered should have a test.
    

It’s not unusual for a single component to have a large test file as the functionality changes. When you do PR reviews, look for new test cases to make sure as many lines of code have test coverage as reasonably possible. When Product brings you requirements, you should think about test cases.

There’s also a balance to this. Some features will take longer to write tests for than to actually implement. This is where you need to consider trade-offs between the effort it will take to write the test, the time it takes to write the implementation, and any upcoming release dates. Personally, I’ve never seen a frontend app written in any framework with 100% test coverage. That’s completely different from the backend, where I’ve seen very close to 100% coverage in some projects.

This difference happens because of the complex and changing nature of the tools we use. It’s hard to get tests to work with the way apps are rendered because of things like timing inconsistencies, triggering events, and external dependencies, such as third-party services. As you start writing tests, you’ll notice when a test is taking up more time than the value it will provide. There’s no hard rule, so lean into your experience, intuition, and the team to make sure no one is getting stuck on putting up a PR because of a complicated test.

# Unit Tests

Whether you decide to use [BDD](https://www.geeksforgeeks.org/behavioral-driven-development-bdd-in-software-engineering/) or [TDD](https://www.geeksforgeeks.org/test-driven-development-tdd/), as long as you write testable code, write tests for it, and make sure the tests pass, you should be good. _Unit tests_ are the core of the testing you’ll do on the frontend. This is how you check if components work as expected and account for edge cases. These are small and targeted tests to catch things like conditional rendering in a programmatic way. They help ensure that your code works with every deployment and give everyone more confidence in doing small deployments frequently.

## Jest and React Testing Libraries

If you want a more established library to work with, [Jest](https://jestjs.io/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) (RTL) are solid choices with plenty of documentation, examples, and community support. You’ve already used Jest for testing on the backend: it’s the underlying framework for the NestJS testing suite. Using it on the frontend is different, though. That’s where RTL comes in.

Any projects that are initialized with [CRA](https://create-react-app.dev/) have RTL included out of the box. However, the React team [deprecated CRA](https://github.com/reactjs/react.dev/pull/5487#issuecomment-1409720741) in early 2023, which is why you bootstrapped this project with Vite. Even though RTL doesn’t come out of the box with some of the alternatives to CRA, they are still useful to initialize your project. The way you write tests won’t be much different from the way you’ll write them with the tool you’ll actually use in this project.

## Vitest

Because this project has been initialized with Vite, you’ll go ahead and use [Vitest](https://vitest.dev/) for unit tests. It’s compatible with Jest, so even if the team is more familiar with Jest and RTL, the transition will be pretty straightforward. This is a newer tool that has quickly growing support in the React ecosystem. It can dramatically speed up the time it takes to run your entire test suite compared with Jest.

###### NOTE

If you ever migrate from CRA to Vite or any other tool, check the benchmarks on how fast things run. You’ll notice differences in test speeds, cold starts for the app, and maybe even response times. The differences in test speeds will help when it’s time to deploy the app to different environments. We’ll get into deployment pipelines in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), but since you’re working on them now, it’s worth noting how long your unit tests take to run. Having fast tests will also speed up local development, so look at the options and see which is best.

Let’s write some tests for the `UserInfo` component so that you can establish some examples for the team in your repo. This will give you an idea of how you can break down a component into different test cases. First, let’s start with the mocks for the tests:

```
import
```

Setting up all the mocks for your tests is going to make writing them easier as you run into more complex scenarios. In this example, you’ve set up mocks for the `useQuery` hook, which fetches your data, and some mocks to handle the error boundary functions. These mocks will allow you to return different values or responses as you write tests without having to call the real method in the component. This is important so that you don’t have to test the functionality of the method you’re calling—you can just focus on the way the code works.

If you check the imports, you’ll find the `orderResponseData` and the `userResponseData` in separate files. These are the mock responses you have defined for the API requests you make in the component. The reason they’re not in the test file directly is because they might get used in other test files for different components. Keeping them in separate files makes them more maintainable because you can keep one source up to date, as opposed to having to update duplicate data across multiple files.

Inside the `describe` block, initialize the mocks to the state the component expects. That means making API requests, returning the mock data as responses, and setting any other values. You do this in the `beforeEach` block, which executes before every test in the `describe` block. At some point, you’ll modify responses or data, and you don’t want those to carry over to the next test case. That’s why it’s common to reset the values in the `afterEach` block.

Finally, you’ll get into the test cases where you check for rendering states. It’s a good practice to at least have a test that renders the component correctly. When you’re deciding what values to check for in your test, make sure they reflect what should be rendered based on data or user interactions. Testing for static text can be misleading because it doesn’t change regardless of what state the component is in. Here are a couple of tests you can write for this component inside the `describe` block, just to get you started:

```
it
```

In the first test, you can see that the component gets rendered and then you check for values that come from the API requests as well as one of the interactive components. Change the values slightly and you’ll find that the test fails. That’s how you can check that your test works. The second test checks that the loading spinner shows when the API response is still loading.

###### TIP

To make sure I’ve covered all the test scenarios, I like to have the component file open right next to the test file in a split-screen view. That way, I can go line by line and add things to my test file to match what’s in my component. Doing this has led me to understand features in a better way and to work through some confusing rendering logic.

Another type of test you might consider is a [snapshot](https://vitest.dev/guide/snapshot.html). _Snapshot tests_ compare the output of your code to a snapshot file that has the exact expected HTML or values that should be rendered on the page. This can be useful to make sure the UI isn’t changing unexpectedly or to confirm updates. Snapshot tests are usually easy to maintain since you don’t have to write code for them: you just [run a command](https://vitest.dev/guide/snapshot.html#updating-snapshots) to update the snapshot file when needed.

After you’ve been in the details of writing tests for a while, it’s normal to notice areas where the code can be written more clearly. Refactoring code to be more testable is a good thing. You can show the team areas that can be simplified or abstracted to helper functions or hooks that make the code more reusable and maintainable.

Sometimes a test can be difficult to write because the implementation is hard to understand. Unit tests will help you find those areas and improve them. Here’s an example of some code that’s a little hard to test although very straightforward to implement:

```
const
```

When you try to test this, it gets tricky because you’re working with the current date every time. That can lead to situations where your tests are flaky and can fail by a number of milliseconds. Here’s how that code can be refactored to make testing easier and the code give consistent results:

```
const
```

Now you can pass a specific date to get a reproducible result. Once you and the team get into the practice of including unit tests for everything you add to the project, you’ll find that it catches regressions in development and leads to other conversations about how code should be broken down.

## Mock Service Worker

There are a lot of other testing tools that can optimize your flow across the team and across other processes in the development lifecycle. One of those tools is [Mock Service Worker](https://mswjs.io/docs/getting-started/) (MSW), which can mimic your endpoints. When you need to check that an endpoint is being called with the correct parameters and returning the expected data, having mock data helps; this tool takes it to the next level. This is great if you need to deploy the app to a develop or feature environment for dev testing or for the Product team to play around and see how the app works in different scenarios. Alternatives like [Nock](https://github.com/nock/nock) or [JSON Server](https://github.com/typicode/json-server) are also worth researching.

###### NOTE

I’ve seen tools like MSW used in organizations where the backend and frontend are separate teams: the frontend team develops against MSW based on the expected data. Be careful here, though. Sometimes the frontend can be developed before the backend is ready, which isn’t a great practice if you aren’t in close contact with the backend team. It can lead to redundant work if the backend team makes changes to the data structure.

MSW works by creating a service worker in your public directory. You then wrap the `<App />` component in [a function](https://mswjs.io/docs/integrations/browser#conditionally-enable-mocking) to determine whether the app will use real APIs or the mock ones you’ll create. So the _main.tsx_ file in your project will look like this:

```
export
```

Now anytime you’re running the app in `development` mode, you’ll get the responses from MSW. The only thing left to finish the implementation of this tool is to create the handlers that will be called instead of the actual API. In _src/mocks_, create a new file called _handlers.ts_ and add this code to it:

```
import
```

This is how you establish mock endpoints and [their responses](https://mswjs.io/docs/network-behavior/rest#response-resolver). Can you see how having the mock responses in their own files comes in handy now? From here, you can add more endpoints and more conditions to handle auth tokens and parameters.

# E2E Testing with Cypress

The last type of testing I’ll go over is _e2e testing_, which you can use to test the full user flow of an app in an automated way. With a tool like [Cypress](https://docs.cypress.io/guides/end-to-end-testing/writing-your-first-end-to-end-test), you can run automated tests from the user’s perspective. Cypress will open its own browser and perform actions in the way a user would. It will click buttons, type values into forms, and determine if the expected changes appear on the page.

Collaborate with the Product team to get all of the scenarios. Using tools like [Cucumber](https://cucumber.io/docs/cucumber/step-definitions/?lang=javascript) and [Gherkin](https://cucumber.io/docs/gherkin/reference/#steps) will help the Product team understand the purpose of testing and get them more involved. Gherkin and Cucumber let you write test scenarios in the given-when-then format, which can help you get Product to define requirements with less ambiguity as you work through specs and mocks together.

Depending on the organization, e2e tests might be work for the QA team. This is especially true if you have any _software development engineer in test_ (SDET) members on the team: people who are dedicated to writing automated test suites with tools like Cypress. SDETs are typically found at larger organizations, although it’s possible to work with them at startups.

Here’s an example of what a Cypress test can look like:

```
describe
```

When you run this with Cypress, you’ll see the process exactly as a user would. The tool will run in a separate window, as shown in [Figure 21-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#a_cypress_test_running_in_separate_wind).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2101.png)

###### Figure 21-1. A Cypress test running in separate window

Now look at an example using Gherkin and Cucumber. You can request that the Product team write requirements in the Gherkin format:

```
// cypress > e2e > user-info.feature
```

This code will be in a file called _user-info.feature,_ and it will go in the _cypress/e2e_ folder. You can take this feature and use the keywords `Given…When…Then` to define the steps in your e2e test, in a new file called _user-info.ts,_ which you’ll also store in the _cypress/e2e_ folder. It will have this code:

```
// cypress > e2e > user-info.ts
```

These are a couple of ways you can implement e2e tests. You don’t need to use Gherkin and Cucumber to get the most use out of Cypress, but it can be effective if the teams agree. Some developers will say that using Gherkin and Cucumber actually makes it _harder_ to write and maintain Cypress tests because Cypress is pretty expressive out of the box; however, at enterprise organizations, it can be a useful way to include the Product team in more technical discussions.

There’s no right or wrong answer to this. It just depends on what you and your team along with the other teams and leadership decide is the best approach. Personally, I prefer just writing e2e tests with Cypress directly because setting up Gherkin and Cucumber to work in a deployment pipeline can get a little tricky with configs.

# Conclusion

In this chapter, you learned about different ways to test your frontend app. Sometimes pushing for technical work to be included as part of the feature requirements is a skill you’ll need. Tests are one of those things, even in crunch times. Unless production is on fire and you need to release a hotfix immediately, make the argument for writing tests as part of your postmortem checklist. (We covered what that is in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide).)

It’s easy to write tickets and say you’ll come back to them, but there’s never time, so it often doesn’t happen. Not all organizations or developers value tests equally, so encourage the practice as early as possible. Even if you’re working on an existing app, it’s better to have some coverage than none at all. Tests don’t speed up new feature development, but they definitely help keep the code stable from a technical perspective.