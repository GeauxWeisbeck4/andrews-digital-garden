---
id: 01JH1NQFMSE61WHZX0BN6MVV6D
modified: 2025-01-07T19:41:24-05:00
---
# Chapter 18. Frontend Error Handling

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 18th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You’ve already implemented much of the functionality that is common to all frontend apps. Up to this point, you have created an infrastructure that should allow the app to be expanded to accommodate any new features that the Product team can throw at you. Now you can dive into more of the details on the frontend that make the app more user-friendly.

I’ve briefly mentioned errors in other chapters of the book, but we’ll finally take a closer look here. Error handling was left out before so that you could focus on best practices for the other parts of the frontend. In practice, though, error handling should be included in the requirements for your tasks. You have to deal with all the errors that could potentially happen to the user. Error handling needs to be thorough so that you don’t leak any sensitive information to users or malicious parties who are trying to attack your app and so that the app keeps a consistent UX.

In this chapter, I’ll cover:

- Error components
    
- User validation errors
    
- API errors
    

These are all standard errors that will eventually come up, so I’ll go over common ways to handle them. You’ll need to work with the Product and Design teams to define and create useful error states that will lead the user down the path they need to take. As you and the team build the app, take note of any edge cases you run into during development, which could bring up things no one had considered before.

# Error Boundary Approaches

[_Error boundaries_](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) are components that display a fallback UI, such as an error message, when errors are caught in your component tree. These error boundaries are unique to React, but it’s generally a good practice to have something like them. If you don’t have an error boundary, users will typically see a blank screen in production, and you’ll see the React error overlay that tells you what the error is during development. This will happen anytime there’s an issue with a component rendering or with any runtime errors. Usually, it’s because of issues with API calls or something the user is doing that the app hasn’t been tested for. This can also cause network errors if you’re depending on a value to make a request.

You might think that using `try/catch` blocks like on the backend will work, but they don’t capture errors like you might expect. That’s because unlike on the backend, React is calling the functions instead of you, which is a more declarative approach. This type of error handling is OK for functionality that is imperatively implemented, but it’s not the safest option for rendering errors.

# THE DIFFERENCE BETWEEN DECLARATIVE AND IMPERATIVE PROGRAMMING

When `try/catch` blocks are discussed in terms of error handling, you’ll hear about declarative and imperative programming. _Declarative programming_ is when you tell the code what you want it to do without getting into the details of how it happens. _Imperative programming_ is when you tell the code how to do what you want. To make sure the difference is clear, let’s look at array handling as an example. With declarative programming, you would use the built-in array methods like this:

```
const
```

With imperative programming, you would do something like this:

```
let
```

We usually do declarative programming in React because it’s easier to read as you gain experience and it’s faster to develop.

To address this restriction with `try/catch`, you can apply several common strategies to your app to create error boundaries. This includes boundaries at the:

- App level
    
- Layout level
    
- Component level
    

Regardless of what level you implement the boundary on, one thing to keep in mind is that the error boundary won’t handle things like async errors, errors that happen in event handlers, and errors thrown in the boundary itself. You’ll end up using a combination of these strategies to manage errors throughout the app.

The _app-level_ boundary is at the top of the component tree in the project and wraps everything. This should be in place in every project so that you know that all errors from any level of your component tree are handled. This is also how you can keep your app from crashing. App-level boundaries aren’t granular, so they really are a catchall for anything that happens. This is similar to having a `try/catch` at the top level of the app with additional `try/catch` statements for more specific errors.

Next are _layout-level_ boundaries. When you have a group of components, like on the user page in this project, you might want to show more specific messaging to the user. If you have a subset of components that have a shared state, consider adding a boundary at this level. Note that if anything happens in one component, then the error boundary shows for all the components in that layout.

Finally, there’s the _component-level_ boundary. This is the most granular level because you have an error state for an individual component that doesn’t affect anything else on the page. When you build components that are more isolated from the others—they have separate states or make their own API calls, for example—consider creating a boundary on that component.

When you implement the error boundary in a React app, you can [build it from scratch](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary), or you can use a package like [react-error-boundary](https://github.com/bvaughn/react-error-boundary). In this project, you’ll use the react-error-boundary package because it’s lightweight and gives you a nice wrapper so that you don’t have to use class components with your functional components. Install this in your repo, and then in the _App.tsx_ file, import it and make the following updates:

```
…
```

This wraps the entire app in the error boundary. That way, any unexpected errors that occur will bubble up to the top of the app and be caught. You need to create the `ErrorFallback` component and the `logError` function that are being used in the boundary. You’ll also use the [`useErrorBoundary` hook](https://github.com/bvaughn/react-error-boundary?tab=readme-ov-file#useerrorboundary-hook) when the error occurs in async code or lifecycle management hooks, such as `useEffect`.

The `onReset` function handles retries from the error state. This will vary based on the level where you have the boundary. Often, it will reset the state of a component to trigger a rerender. If an error occurs at the app level, you can use this function to refresh the whole page. This way of doing a refresh will fetch data from the cache, so if the issue was with an API call, you need to implement a lower-level boundary with that state being captured.

## Error Components

You’ll have to work closely with Design and Product to decide what the errors should look like. Some teams like to use modals with retry buttons, full-screen error messages that direct the user to take a different action, toast components to temporarily alert the user of an error, or a combination of things. For this project, you’re going to make dedicated components that display on the page in place of the expected layout.

You’ll build components for a few specific cases, including when a value is undefined, when an error is returned from the backend, and generic mishaps. It’s good to start with the generic “Something went wrong” component so that you know the app has something for anything that comes up. To accommodate the way React Router v6 currently handles errors, you need to do a refactor in _routes.tsx_.

###### TIP

Double-check the React Router docs to see if this is still the case. Since packages change so frequently, there’s a chance the behavior could be different by the time you read this.

Right now, _routes.tsx_ looks like this:

```
const
```

You’ll update it to the following where you use `useRoutes` instead of `createBrowserRouter`:

```
import
```

Then, you’ll have to update _App.tsx_ and replace:

```
…
```

with:

```
…
```

###### TIP

It’s important to be aware if any of the packages you use have built-in error boundaries that will prevent the errors from being caught at the app level or the level you’re expecting. For example, React Router currently catches errors at the `<RouteProvider>` level. So if you notice during development that the error screen you’re getting isn’t right, start looking at the parts of the code that can trigger errors to see if anything is blocking them.

Now that you have the error boundary in place and you’ve refactored the code to handle it, you can build the generic `ErrorFallback` component. In _src/components_, create a new directory called _ErrorFallbacks._ In the new directory, add three new files: _index.tsx, ErrorFallback.tsx,_ and _ErrorFallback.test.tsx_. In _ErrorFallback.tsx,_ add the following code:

```
import
```

Remember that you will need to update the _index.tsx_ file to use this component as a module. It’s the same as you’ve done for every other component file. Tests should be written at the same time as the component, but we’re going to come back to that in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing) so that we can go over all testing best practices together.

You now have a component that will show any time an error is caught at the component or layout level. It takes the error object and displays the message on the page so that developers can know what happened. It will look similar to [Figure 18-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#generic_error_component_with_the_error) when a runtime error is caught.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1801.png)

###### Figure 18-1. Generic error component with the error message

## Logging Errors

The second part of handling errors is logging them so that you have a record of what’s going on with the app when you need to debug production issues. A third-party service like Sentry, LogRocket, or Datadog or using a monitoring tool from your cloud provider, such as Amazon CloudWatch, will help out here. You can create a small helper function to do this. In the _src_ folder, make a new directory called _utils,_ and in that folder, make a new file called _helpers.tsx._ Add the following function to this file:

```
import
```

# KEEP A CONSISTENT FRONTEND FOLDER STRUCTURE

The frontend has arguably reached a point where deciding on the architecture and folder structure is more complex and opinionated than the backend. Some teams will use something like a _utils_ folder, or it might be called _helpers_ or something else. You can choose to have a separate folder for your types or include them in the same folder as the component. It can get very convoluted and difficult to find functionality if the team doesn’t remain consistent as the app grows.

While you are using a specific folder structure for this project, and it’s a good starting point for any greenfield app, that doesn’t mean it’s the one you should use for every project. Just keep in mind that as you get exposure to more projects and products, you’ll see vastly different structures and develop your own preferences. There isn’t a right or wrong way as long as the whole team agrees to enforce the structure you all put in place.

This logging functionality can be as robust as you and the team want to make it. The better your logs are, the faster it will be for you to track down the root causes of the errors. You can also use a logging tool to give you metrics around which errors happen most often. That can point out places that need to be refactored or that don’t give a good UX. One area this comes up with is when users need to enter information with ambiguous instructions.

# User Validation Errors

When your users interact with your app, you need a way to give them feedback on their input. You can show inline errors as soon as the user focuses on an input. You can wait to show any errors until the user tries to submit the form or changes focus from an input. You can show errors in one block. You’ll likely use a combination to give the user the most useful information.

You also have to account for the component library you’re using. Since you’re using MUI in this project, there’s already validation on your input fields, and you may have noticed that when building the search form. With MUI, you can make a field required and pass the validation requirements to it like this:

```
const
```

Now the user can’t submit the request without entering at least three characters in the field, and the input is restricted to 15 characters, so the user won’t be able to type in more than that. If they try to submit the form and the input doesn’t meet the validation requirements, then they will see a specific message for their error, such as in [Figure 18-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#required_input_error_message) and [Figure 18-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#required_character_length_error_message).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1802.png)

###### Figure 18-2. Required input error message

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1803.png)

###### Figure 18-3. Required character-length error message

These messages come straight from the default Chrome HTML validation on input fields. You can take advantage of the [other error message formats](https://mui.com/material-ui/react-text-field/#validation), such as highlighting the input within a form with a customized message. The way you handle error messages will vary quite a bit depending on which components you use to accept inputs.

Another widely used approach to handling user input validation is at the form level with something like React Hook Form. You can add validation that will check all the fields when the Submit button is clicked. The implementation might look similar to this:

```
<
```

Or you can use a validation schema with a tool like [Zod](https://github.com/vriad/zod), [joi](https://github.com/hapijs/joi), or [Yup](https://github.com/jquense/yup) with React Hook Forms to pass a schema to `useForm` as an option. It will validate the inputs against the schema and show the errors. Here’s an example of that with the search form using Yup:

```
import
```

Having validations written as functions or schemas makes them more reusable across different components and gives you more control over customization. As part of the React Hook Form package, any `<input />` that is registered with validations will be checked when the `onSubmit` function is triggered. When you show the user messages at this point in their flow, it can be easier to make them understand that they need to make changes before clicking the button again. The advantage of this is that the user won’t see errors until they make the submission action.

There are varying views on what is the clearest way to show validation messages, which is why it’s essential to work with the Design team because they should have a stronger idea of what the user expects from the app. That leaves you to help them figure out what errors might come from the backend and what that means to a user.

# API Errors

React Query, which you’re using to fetch your data, already comes with built-in error handling, which makes it easier to deal with any errors that come from the backend. Depending on the API design, the status codes can be used to show meaningful error messages to both the user and the developers. These errors might not have anything to do with an action a user took, but the user may need to refresh the page or do something else to try the request again. Your error components should guide them to the corrective action without revealing exactly what happened on the backend.

For example, if the API responds with a 422 status code, that could be an issue with how the code was written to format user input for the request. The frontend could have an incorrect parameter name that it submits to the backend. Or if the API server is down, the frontend might receive a 500 status code. There’s nothing for the user to do in either of these scenarios, but they still need to be shown something. This is also where your logging will come in handy for any developers debugging the issues. You and the dev team might even tie log messages to each API error screen. [Figure 18-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#example_of_a_fourzerofour_error_from_an) is an example of a 404 being returned from a backend request in the browser tools.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1804.png)

###### Figure 18-4. Example of a 404 error from an API request

Here’s what the JSON response from that request looks like:

```
{
```

These screens or components will be similar to what you’ve already built for the generic error component. They will be modified versions of this with different wording and maybe even action buttons. The main differences are that they will show only under the specified conditions and they will happen at the component or layout level in the app. Some common ways to handle the content in these components include showing the error message that’s coming from the backend or showing a custom message directly from the frontend strictly based on the status code and having the specifics in the logs.

Just to make sure you are handling the API errors appropriately, you can use a tool like [httpstat.us](https://httpstat.us/) or the [tweak browser extension](https://tweak-extension.com/docs/intro). You can also mock errors from your server with a tool like [Mock Service Worker](https://mswjs.io/docs/getting-started), or you can add a property to your requests that forces a certain error and payload. An example of that additional property could be called `responseStatus`, such as the following:

GET /order/72?responseStatus`=``404`

This goes back to your API design. The error messages sent in the responses might only apply to the developers debugging the issue, and you don’t need users or malicious parties having access to the exact error. One strategy is to log the error message from the backend and display something different to the user. Be cautious with this approach because a malicious or curious user will be able to see the response in the browser tools. You typically want to send a redacted version of the error to the frontend so that this situation doesn’t come up and you can focus on giving the users friendly and useful messaging.

# Conclusion

In this chapter, you learned about the different types of error boundary levels, how to implement a boundary, user validation messages, and API error handling. Error handling is important for user experience, developer debugging, and ensuring app security. Once you have your app in a stable state and there are discussions around releasing it to production, double-check that you have something in place even if it’s just the generic error component. This part can get overlooked as you focus on feature development unless Product, Design, or you bring it up.

Add this to your app fitness checklist. You may have to dig into the React docs and the docs for the packages in the app to fully understand how errors are handled. Understanding where errors are triggered from, how they bubble up in the app, and where they will be displayed on the page will affect the way the entire team writes code. This is another good time to have a demo or explainer meeting with the team to bring up any considerations that others have encountered. Error handling can be a little tricky, but once you have it implemented in the app, you and the team can iron out any ambiguities.