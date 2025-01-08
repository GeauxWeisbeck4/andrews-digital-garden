---
id: 01JH1NNHCJHKA8XNY43W4N6RF0
modified: 2025-01-07T19:40:19-05:00
---
# Chapter 16. Data Management

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 16th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You already have some state management in place, so now it’s time to fetch and update data. Talk with the devs who are working on the backend so that you can communicate expectations around API calls. Since you’re a full stack dev, you can also note any changes that need to be made and write tickets to do that work yourself. For now, you and the backend devs have agreed on expected endpoints and data schema, so you can get started.

It’s a good practice to limit the number of API calls that the frontend needs to make for performance reasons. So you really need to understand how the data will affect the page layout and when it needs to show in the UX flow. Users need a consistent experience or else they think something is wrong with the product or some data they need is missing or incorrect. Considering this will help you figure out how data should flow.

In this chapter, I’ll cover:

- API calls
    
- Async handling
    
- When to check on backend functionality
    
- Tools you can use
    

There are some nuances to calling data endpoints because you have to think about latency and what happens when your backend requests don’t return what you expect. You also have to think about when data should be requested and why calls are being made. This is going to show you if the backend returns data in the format you need. You will likely need to make minor adjustments on the backend as you put together the interface.

Before you jump into the code and request data, it’s good to evaluate the tools you have available and review them with the team.

# Potential Tools for Fetching Data

The ecosystem for JavaScript tools is constantly changing, so you have to do research to make sure you’re staying up to date with the latest tools. There are currently several you should mention in the discussion with the other devs to make the selection: [TanStack Query](https://tanstack.com/query/latest/docs/react/overview), [SWR](https://swr.vercel.app/), [RTK Query](https://redux-toolkit.js.org/rtk-query/overview), and [React Router](https://reactrouter.com/en/main/guides/data-libs#loading-data). You’ll find that some of these tools work well together and pair nicely with [Axios](https://axios-http.com/docs/intro) or the built-in [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

Some specific metrics you want to look at for data-fetching libraries include:

- Cache management
    
- Devtools
    
- Retry handling
    
- Error handling
    
- Query and mutation capabilities
    
- Supported protocols
    
- API definitions
    
- How well it integrates into the frontend framework
    

You also need to consider standard package metrics like:

- Bundle size
    
- Community support
    
- Documentation
    
- Examples
    

The team’s experience is going to help drive the decisions here because each dev will be able to highlight some of the practical aspects of the tools they’ve worked with. If you already have Redux in your app, then RTK Query will be a great option because it’s part of the Redux suite. Having caching, memoization, or refetching implemented out of the box could lead you to choose a full-feature tool like TanStack Query or SWR.

###### NOTE

_Memoization_ is when functions that are time or computationally expensive have their results cached. When you pass a function the same argument values, it will return the same result. In these cases, memoization will return a cached result when the same arguments are passed to the function without running the function again.

It’s important to have discussions and demos for all the options you all come up with. The packages you choose at this point will greatly affect speed and user experience as the app grows and functionality becomes more dependent on data updates. As with state management, you _can_ switch tools later, but that will be a large refactor that you and the team will have to do gradually.

If there’s a reason to switch to GraphQL or [tRPC](https://trpc.io/) on the backend—for instance, you have very complex data relationships or deeply nested values—then your frontend tool will also likely change to be compatible with the response you’ll get. That’s when you might consider something more specific like [urql](https://github.com/urql-graphql/urql) or [Apollo Client](https://www.apollographql.com/docs/react/). You can find [comparisons](https://tanstack.com/query/v4/docs/react/comparison) of a number of [tools](https://formidable.com/open-source/urql/docs/comparison/) that have been done by [different people](https://redux-toolkit.js.org/rtk-query/comparison), but it’s good to do your own benchmark testing for at least the top three choices from the team.

Once you’ve figured out the tools and combinations you want to use to get the best metrics, you can install and set up the tools that you and the team agreed to use for data handling. Then you’ll get into some of the details, like loading states and configuring headers. We’ll take a deeper dive into error handling in [Chapter 18](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#frontend_error_handling) because that will involve more than just API errors.

# Handling API Calls with Axios and TanStack Query

You’ll use Axios for your API requests because it comes with many built-in configs, so the syntax you’ll use is a little more straightforward than with the standard Fetch API. You’ll also use TanStack Query as the tool to handle your API responses because it’s very feature rich. Caching responses, pagination, and reducing the number of requests you need to make are just a few of the huge tasks this package handles for you out of the box.

###### NOTE

One quick thing to take note of is that the Fetch API can do essentially the same things as Axios. So it might not be necessary to install the package. Axios does offer a streamlined developer experience, though. This is another thing you’ll want to check in with your team on because some devs have a preference for one tool over the other.

Install Axios, TanStack Query, and the TanStack Query plug-in to help with debugging and catching potential issues with these commands:

npm i @tanstack/react-query axios
npm i -D @tanstack/eslint-plugin-query

TanStack Query works like a context, so you’ll need to create the [query client](https://tanstack.com/query/latest/docs/eslint/stable-query-client#stable-query-client) and wrap the entire app in the [query client provider](https://tanstack.com/query/latest/docs/react/reference/QueryClientProvider). In the _App.tsx_ file, make the following updates to initialize and use the query client and provider:

```
…
```

The whole app is wrapped in the `QueryClientProvider` component, and it has access to everything in the `queryClient` cache. You can do more advanced things, like set options for the cache on the query client. It has a lot of methods that you can use to interact with the cache as well, so make sure to read through the docs as you go through the initial setup to see if there’s anything you need up front. Some of these options will come up organically as the app grows and user needs change.

Before you move on to updating the `UserInfo` component to fetch data with Axios and store it with TanSatack Query, there’s a dev task you need to do. In the root of the project, add a new file called _.env_.

## The .env File

On the backend, Node introduced first-class support for _.env_ files in [Node 20](https://nodejs.org/en/learn/command-line/how-to-read-environment-variables-from-nodejs). Before that, devs used the dotenv package. As with the backend, you can use environment variables (env vars) on the frontend. One big reason to use env vars is that they are more secure because they live in memory, not on disk. This means that it’s generally harder for an attacker to get sensitive data because it gets reset instead of persisting somewhere like a database, which is true for the backend.

On the backend, you can use env vars liberally to provide secrets or sensitive configs. You just have to make sure you don’t pass anything to the frontend accidentally. On the frontend, env vars are simulated and should _never_ contain sensitive info. When you build the code, your env vars will be present in the compiled output in plain text. So frontend env vars are only good for things you don’t mind being public, like endpoint URLs or public API keys.

This is a very extensive topic, and it’s not something you need to dig too deep into unless you want to get into the details of security mechanisms and hardware. Just know that when you use env vars, your secrets are typically not stored in your repo (check the _.gitignore_ file for more details) and get pulled from your CI/CD pipeline based on the environment the app gets deployed to.

When you’re developing locally or working in a develop environment, it’s OK to load environment variables from disk in an _.env_ file because the values here shouldn’t affect your production data or billing for services. Depending on the tool you choose to build your app, you’ll access env vars similarly to how you do on the backend with code like the following:

```
const
```

Since you’re using Vite on the frontend in this example app, you’ll [access env vars](https://vitejs.dev/guide/env-and-mode) with this code:

```
const
```

So go to your _.env_ file because this is where you’ll put the URL to call your API. The reason you define the API URL in this file is because the value might change depending on what environment the app gets deployed to. Some teams have a develop environment for the backend that’s used for testing both backend and frontend functionality. This environment is separate from production, so the API URL will be different. In that new _.env_ file, add this line:

```
VITE_API_URL
```

For local feature development, it’s good to run the backend code locally from the state it’s in at [Chapter 11](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch11.html#scalability_considerations). That way, you can just connect to the local API instead of a deployed API and make adjustments to the backend as needed.

Now you can create a new file in _src/page/UserInfo_ called _useUserInfo.tsx._ This is a [custom hook](https://react.dev/learn/reusing-logic-with-custom-hooks) that will handle the API calls for you. You’ll add the code to make the backend calls using the `VITE_API_URL` you just defined. The reasons you’re making a custom hook are to have a clear separation of concerns, to make testing easier, and to keep your component code readable. So add the following code to your custom hook:

```
import
```

The first thing to note is the [`useQuery` hook](https://tanstack.com/query/latest/docs/react/reference/useQuery) being called to fetch the data with Axios and cache the response. The `useQuery` hook includes a number of values you can use to make the app more responsive, but for now you need only the loading states (`ordersAreLoading`, `userIsLoading`), the error states (`ordersErrors`, `userErrors`), and the data from the requests (`orderData`, `userData`). The API URL you defined is being used in the requests. All of these are returned as values from your custom hook in _UserInfo.Container.tsx._

In _UserInfo.Container.tsx,_ you’ll need to do the following refactors to your code to use this hook and render the component based on the values returned from it:

```
…
```

You can note where the states get updated in the `useEffect` hooks based on their individual dependencies. So whenever the value for `orderData` or `userData` changes, the respective state will get updated as well. Finally, there are two conditionally rendered components for the loading and error states. This provides a better UX, so users aren’t left wondering why they can’t interact with the page.

## Handling Loading States

Something you should create are actual components for your loading states. You can have different loading states for each component on the page that receives data dynamically. This is where your component library can help. You should discuss this with the Design and Product teams. MUI has different [loading state components](https://mui.com/material-ui/react-progress/) that you can customize for your app, which can give you a starting point for the discussion.

For example, you will have separate calls for your orders search in the table on the user info page and the general user data that gets loaded. You could block the whole page and wait for all the responses to return, but this will negatively affect the UX because users won’t be able to do anything on the page. One of these responses will likely return before the other, and you’ll need to show a slightly different loading state for each.

As a simple example, you can have two different loading components for the user header and the orders table that get conditionally rendered instead of one check for both loading states, as in the following example and [Figure 16-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch16.html#screenshot_of_the_multiple_loading_comp):

```
// UserInfo.Container.tsx
```

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1601.png)

###### Figure 16-1. Screenshot of the multiple loading components in the code

This removes that shared component that returned when both data requests were loading and gives you more flexibility with what users see and experience. Your loading state helps you take care of the asynchronous way API calls are made from the frontend. You can [chain promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises), use [`Promise.all`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all), or manually create a loading state to handle data. TanStack Query handles that with the `isLoading` value that gets returned until you get a response. You can combine multiple requests into one, depending on how you configure TanStack Query options.

## Handling Error States

We’ll go into much more detail on error handling in [Chapter 18](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#frontend_error_handling) because it’s a huge discussion, but you can add a bit to your component now to cover it initially. Think of this implementation as scaffolding to unblock the team for now. When you get an error from any of your API requests, you want to handle it gracefully so that the app doesn’t crash for the user. Having a simple error message that lets the user know something happened is better than nothing:

```
…
```

Now the users will see this message whenever there’s an error from either request. In [Chapter 18](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#frontend_error_handling), you’ll see how to split these into separate messages for different error levels and how to handle them across components.

## Configuring Request Headers

As you see in this example, you will get to the point where you need to make multiple requests to provide the functionality the user expects. This will involve sending requests to the backend that need specific values in the header, like access tokens or expected data types. So you need to configure your data-fetching tools to handle headers to avoid things like [CORS errors](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors) or sending data in invalid formats. You can configure the headers on each request, or you can do it on a global level in the app to ensure consistency across all your requests.

For this app, you can assume that you and the team decided to configure the [headers with Axios](https://github.com/axios/axios?tab=readme-ov-file#config-defaults) at the global level for better maintainability and easier debugging. One of the most common headers to configure globally is for authorization, although you can configure headers for endpoint patterns as well. You’ll do this by creating a new file called _axios.config.ts_ in the _src_ directory. In this file, you’ll define the base URL for all the requests being made, the content type, and the authorization:

```
import
```

For now, the `AUTH_TOKEN` is a placeholder value that may or may not come from the `localStorage`. That’s something I’ll get into in [Chapter 19](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch19.html#frontend_security_considerations) as we cover more security concerns. Since the base URL is defined at the global level, it will be used for every request you make in the app. If you need to make requests for different base URLs, you can make those per request, or you can have different instances of Axios that you use.

After you talk it over with the team, you may choose an approach like making hooks for your requests or making an SDK for the backend. An _SDK_ is when you make a package that exports methods that make the API requests along with types for the methods and parameters and any documentation for the methods. This is an approach you might take if you’re making these API requests in multiple frontend apps or other backend apps. [SendGrid](https://www.twilio.com/docs/sendgrid#send-your-first-email) and [Google Maps](https://github.com/googlemaps/google-maps-services-js) are examples of widely used SDKs.

You also need to discuss the caching strategy your app will use, which I’ll go over in [Chapter 20](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#frontend_performance). As you set up the rest of the frontend requests, you’ll need to make sure the frontend isn’t handling backend functionality.

# When to Check on Backend Functionality

Anytime you start doing calculations on the frontend, it’s time to examine the backend more closely. While the frontend can handle calculations, doing them on the backend provides more consistency across browsers and client machines. So when you get to a point where you need to do calculations with the data on the frontend, double-check if it would make more sense to move that logic to the backend. That way, you can do all the calculations directly on the server one time instead of thousands of times across clients, or you could cache the result and send it to the client.

If you do a lot of data formatting on the frontend, you should check if there’s a reason for this. If you have other services that are calling this endpoint and that’s a reason, consider making a different endpoint. Split the business reasons so that the user functionality doesn’t get affected by changes meant for other services.

Another thing to look for is when the frontend receives large payloads. This might be for pagination, sorting, or filtering if it wasn’t included in the original API specs. Go over this with the team because it could be something that needs to be added to the backend for multiple endpoints. Having a standard format to request data for different pages is going to help with code quality as the app grows, so really discuss this with the team.

Something else to note is when you need to make multiple requests to fetch data and then merge it on the frontend. Talk it over with the team and decide if there is a way to make a single endpoint for this data or if you really do need the separate endpoints. It’s not always a bad thing to make multiple requests if it makes sense for long-term development. There might be optimization reasons to have separate endpoints, like keeping a separation of concerns on the backend.

# Conclusion

In this chapter, you learned more about handling the data requests and responses with tools that can make the process easier to manage. Caching is a huge undertaking on the frontend, so having a package that handles that for you out of the box with a few configs is very helpful. Also having tools that automatically tell you the status of your request helps you show the user meaningful messages on the page. When this is paired with state management, you’ve created a large part of your frontend.

Establishing consistent, well-thought-out data management strategies now will help keep any future code secure. You have the core functionality in place, and you’ve enabled your team to really dive into feature development. There will be options that you’ll update and add or even remove as the product becomes feature rich. This is when you’ll need to be stricter in your code reviews so that the quality of the code remains high.