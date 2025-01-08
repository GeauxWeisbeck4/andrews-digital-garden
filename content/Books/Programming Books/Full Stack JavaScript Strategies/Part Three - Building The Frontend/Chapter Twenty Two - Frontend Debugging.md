---
id: 01JH1NTQSJC16ZM7CWRMJC5Y8F
modified: 2025-01-07T19:43:10-05:00
---
# Chapter 22. Frontend Debugging

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 22nd chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You’re finally at the point in your frontend app where you have everything ready for production. You’ve had to debug issues during development, so you already have some strategies or approaches to take. Now that everything’s integrated and your app is in production, you need a more systematic approach to finding and resolving bugs.

Although there isn’t a single strategy that will guarantee you find a bug the exact same way every time, there are some common things you can do to give yourself and the team a good starting point. So as you and other teams start to work together with the app on production, you can help everyone by learning how to quickly track down root causes.

In this chapter, I’ll go over:

- Places to start your debugging journey
    
- Tools you can use to help
    
- Other areas to consider when you get stuck
    
- What to do when a bug is happening because of another team
    

Debugging is an art that takes time and exposure to numerous projects to get better at. It’s one of the most underrated and valuable skills you can have because it will be applicable to any app you ever work on and across all programming languages. Even though we try our best not to introduce bugs to the code, it happens. As you encounter more causes of bugs in your work, you’ll be able to add more debugging approaches to your toolbox.

# The Debugging Process

One of the first things you need to do is become aware that a bug exists. Someone from the Support team could bring it up because they’re getting a lot of users writing in about it. You could also find out something is happening in production based on your monitoring and alert tools. You might even be debugging a separate issue and notice a lot of the same errors happening in the logs. There’s also the chance that you, someone on the dev team, or someone in QA notices something as you’re testing other things.

Once you’re aware that a bug is in production, it’s time for you to start collecting information about it from different sources. That includes talking to Support to figure out the steps to reproduce the bugs users are seeing, looking through the logs more thoroughly, and going through the most recent changes that have been deployed to production. After you have enough info to reproduce the bug, then you can jump to the next step in the process, which is figuring out how to fix the bug and write tests for it.

You’ll find that in many cases the bug fix can be a simple one-liner or a quick code change. It’s the process of finding where that code change needs to happen that takes the most time. After you’ve made the fix, you can finish writing test cases to make sure you have coverage for this scenario. Finally, you deploy the fix to production and validate that it works there.

This is a typical debugging process that you might follow. Now we’re going to go through how you might find and reproduce the issue once you’re aware of it.

# Looking Through Logs

After you’ve become aware of the bug, checking logs is one way you can start searching for the root cause. You can find what triggers the errors and look for the records in the logs that match with that. You should also check the backend logs to see if any API errors occurred that may have caused the frontend errors.

Depending on the logging tool you use, you should be able to search through the logs. If you know what page the error happened on, you may be able to find a root cause by searching for that specific page. There can be a lot of data to sift through in the logs, so keep your search very specific. You want to focus on things like:

- Patterns in the errors
    
- Certain events that cause errors each time
    
- Data being captured
    
- Users who experienced the errors
    

This is an area where you can really shine because going through logs can be a daunting task. Something to keep in mind as you go through logs is that having more logs from the app would be useful. Go ahead and add that to the code and deploy it! Your backend logs should be inaccessible by outside parties for security reasons anyway, so you can log more of the user data that you have available to get even more details about what’s happening. If the error involves an API request, you can add more logs on the backend to see if it’s getting the request and the response you expect.

Your logs can tell you if a certain component sent the correct parameters for a request. You can make the log message even more specific and include things like the user’s auth token to check their permissions. Or if you notice the error messages aren’t completely showing in the logs, you might need to update that specific logger. For example, if your log says something like `“error from orders.createOrder”`, that doesn’t tell you what happened. It can also mean that the underlying error is being suppressed by custom error handling that you and the team have implemented on either the frontend or backend. Go into the code, search for that error message, and see what you have for the logger and why.

###### TIP

A small thing to keep in mind when you are looking at logs is how you handle objects. If your log messages have `[object Object]` in them, that probably means the data needs to be stringified.

This is especially true for third-party service errors. You might call a function directly from a third-party service on the frontend, such as [Okta](https://auth0.com/docs/quickstart/spa/react). You don’t have control over what info you receive if that package throws an error, but you can pass the info along in the raw format. That can help you trace bugs back to some settings you created in the dashboard for a service.

When you’re reproducing the bug locally, don’t hesitate to add loggers all over your code if they will help you track down a difficult bug. If you think they are crowding the code, you can always remove them once you’ve found a bug. Sometimes bugs won’t occur when you run the app locally because you aren’t connected to certain services or the environment has special configs.

Another thing is that you don’t have to strictly log errors. You can have debug, info, and warn logs, like we discussed in [Chapter 9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#backend_debugging). These will help you identify patterns in how the app typically behaves so that you can target deviations from normal behavior. That’s why you can take the lead and create a log checklist for the team to go through and amend as they learn new things. Here’s a little checklist I use when I’m looking through the logs to find a bug:

- Figure out what time the bug was reported and filter the logs based on that.
    
- Look for specific messages that are related to the issue. That might be a page, a component, an API call, or a user ID.
    
- Read the full log object to see what’s included in the message.
    
- Go through the call stack if it’s included in the log object because this helps find a specific file where the error is coming from.
    
- Look for other logs that are happening around the related issue. Check if there are logs before or after the issue to get more info.
    
- Go to the code and see if you can find the problem area.
    
- If you can’t figure it out, add more logs around the problem area in the code and recheck the logs.
    

###### NOTE

To trace an error through the call stack, you’ll need to have your [source maps](https://web.dev/articles/source-maps#why) configured correctly. This isn’t typically enabled by default in many modern tools, like [Vite](https://vitejs.dev/config/build-options#build-sourcemap), [Rollup](https://rollupjs.org/configuration-options/#output-sourcemap), and [esbuild](https://esbuild.github.io/api/#sourcemap). They have a specific config value to enable source mapping.

This is just a starting point. When you get into your app’s logs, you have to see where they lead you. There’s no normal for what should show in the logs for an error, and the error might not even get logged. But understanding what you’re looking for will help you figure that out faster so that you can move on to other debugging options.

# Checking the Code

As you do your due diligence and check the logs, you’ll end up in your code. This is when you can run the app locally and see if you can reproduce the bug. Keep in mind that as you debug, you’ll jump across all the different debugging methods in this chapter. So if your logs immediately lead you to the problem in the code, don’t hesitate to get right into some of those suggestions. You may also prefer to start in the code and then check the logs. Any approach that gets you to the root cause of the bug is the right one.

If you’re just coming to the code with an error message, you can search the project for that message. If the message is in the code, this should show where it is. Then you can go to that file and start tracing what causes that error. This is where you have to keep track of where you’ve been in the code because it’s easy to lose your place as you switch between files. Something that might be useful is opening a split view of files so you can do side-by-side comparisons.

## Using console.log Messages

As you go through the code with the app running locally, toss in some `console.log` calls around areas that you think might help. This is similar to the logs you looked at earlier except it happens in the browser console, so you don’t want to leave these in after you find the bug. You can have `console.log`s to help you figure out when a component gets rendered and what data it’s rendered with. These programmatic checks can show you when you’re getting data in a format you weren’t expecting or when you aren’t getting data at all.

For example, you might notice that your component isn’t rendering. By using `console.log`, you can check in real time if the conditions for that component to render are being met. With a `console.log`, you might find that a response is stuck in a loading state, it’s returning an empty array, or you aren’t accessing an object value correctly. You might even catch a side effect happening where the component loads correctly on the initial render but something with a `useEffect` hook is making the component rerender incorrectly.

The best part about your `console.log`s is that you don’t have to format them in a certain way because they’re going to get deleted after you find the problem. Tools like [missionlog](https://www.npmjs.com/package/missionlog), [pino](https://www.npmjs.com/package/pino), and [winston](https://www.npmjs.com/package/winston) are also good frontend debugging packages. Here’s an example of how you could mark up your component for debugging with `console.log`:

```
useEffect
```

This is a small example of how `console.log` can be used to help you debug code locally. These are temporary checks that don’t need to be logged in production. This is also how you can validate incremental changes in the code. When you get into a debugging flow, changing one line of code at a time can help you pin down the changes you need to make.

For example, maybe you need to update the URL with query string parameters that change based on dropdown selections users make. You notice that the query string doesn’t update like you expect, but the page reloads anyway. You can change one of the dropdown values to use a state variable and see if that helps. If it does, then you can change another in the same way and see if the update still works. When you’re debugging, you don’t want to get to a point where you’ve fixed the bug but you have no idea how. Or worse, you fix the original bug and introduce a new one. Unit tests help to protect against this, but they aren’t foolproof. That’s why you want to do the changes incrementally and see what effect they have. If it means changing one line of code at a time, that’s fine.

## Using Breakpoints

Now that you’ve seen how you can use `console.log` to debug your code incrementally, you can take it a step further with breakpoints. When using a debugger like we will in the Chrome DevTools, a _breakpoint_ is a place where code execution is stopped. This is how you can check values in real time without having `console.log`s everywhere. Breakpoints also allow you to see the call stack up to the point where the code execution stops, which may reveal where the real problem is.

You’ll be able to step through the code line by line until you trigger the bug. That means you can see what data is getting passed as it’s generated, and you can see the order that functions are being called in. If a function is called, you can usually step into it to see what values it receives and the output it returns. A caveat to stepping into functions is when you have async tasks, like API calls. You’ll need to put the breakpoint in the callback itself because you can’t directly step into that function.

This usually ends up being very efficient and succinct because you can keep track of what new code does. For example, maybe you refactor a function parameter to be an object instead of an array. Now you know what the real fix for the bug was, and you can clean up any code related to it. Next, you can start making incremental Git commits so that you don’t lose your progress. Something that feels worse than fixing a bug and making a new one is fixing a bug and deleting the code.

## Using Unit Tests

Because you already have tests in place like we covered in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing), you can use these to help you debug as well. Try running the unit tests and see if they pass to start. Then try changing the conditions for your tests and see if they still pass. If you notice a test is passing even though it shouldn’t be, this is a sign that you need to take a closer look at the code for it. In some cases, there could be an issue with an async function, or you might not be rendering components as expected.

This is a good time to add more tests. Sometimes the exercise of going through the code line by line to look for new areas to test can highlight conditions. You can use this line-by-line review to try some refactors to simplify the code and reveal issues.

## Using Git

Git can be your friend during any debugging session. Not only can it help you keep track of the changes you’re making to fix the bug, but it can also help you quickly find the commit that introduced the bug. This is where a command like [`git bisect`](https://www.metaltoad.com/blog/beginners-guide-git-bisect-process-elimination) comes in; we’ll look at that more in [Chapter 28](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch28.html#git_management). Looking through the Git history to see what changes were pushed before a bug started happening can take you right to the change that created it. You can look at the PRs that have been merged or use your IDE to see a comparison between the current code and the last commit, as in [Figure 22-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#git_history_in_vs_code).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2201.png)

###### Figure 22-1. Git history in VS Code

You can see who made the changes and when. Keep in mind that the purpose of this isn’t to blame someone for a bug. At some point, we’ve all gotten some shaky code through the PR review process and caused production issues. You know what that feels like, and you can coach other members of the team through these stressful times.

Once you know who wrote the buggy code, you can get on a call with them to figure out what they were trying to do and come up with a better solution. Some of the best debugging sessions come from a pair-programming session. Having two people looking at the issue lets you “rubber-duck” with each other to figure out what could have happened and the best approach going forward. [_Rubber-ducking_](https://careerfoundry.com/en/blog/web-development/rubber-duck-debugging/#:~:text=Rubber%20duck%20debugging%20is%20a,duck%2C%20in%20its%20literal%20sense.) is when you talk through each line of code with an inanimate object or another developer. As a rule of thumb, if you’ve been debugging for more than 30 minutes, see if you can bounce ideas off another dev.

Even typing out the question can help bring you out of the depths of the code. This is where an AI tool like [ChatGPT](https://openai.com/chatgpt/) can come in handy. When you have to explain the problem to someone else, you might find that _you_ get a different view. You never know what someone else on the team has experience with. You might have an issue with datetime conversions, and that’s what one of your fellow devs worked on in their last role. It can be tempting to get caught up in the idea that you should know how to fix everything quickly, but nobody knows everything, no matter how long they’ve been programming. So if you get stumped, reach out to others.

# Using the Browser Devtools

When you’re trying to narrow down why a bug is happening, use the browser devtools to see what’s happening directly on the page. I’ll be referencing the Chrome DevTools, but all the browsers have similar tools. You’ve already looked at the browser tools for performance and accessibility reasons, so we’ll focus on four tabs: Elements, Sources, Network, and Application.

## The Elements Tab

If you’re trying to fix a style issue, but you’re having a hard time finding the incorrectly styled element in your code, the Elements tab can help. You can go to the page and right-click the element you want to check the styles for, which will open a view like in Figures [22-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#page_view_of_checking_styles_in_the_ele) and [22-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#chrome_devtools_view_of_checking_styles).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2202.png)

###### Figure 22-2. Page view of checking styles in the Elements tab of the Chrome DevTools

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2203.png)

###### Figure 22-3. Chrome DevTools view of checking styles in the Elements tab

In the panel on the right in [Figure 22-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#chrome_devtools_view_of_checking_styles), you’ll see all the styles applied to the highlighted element. You can edit the styles live in the browser by changing values in the right panel. If you’re dealing with deeply nested elements that have styles applied from parent components or third-party packages, such as MUI, this can really help you get to the root of the styles and what needs to be updated. You can even check for hover states and mobile views in the Elements tab.

To check mobile views, you can click the second icon on the left in the header nav of the DevTools as shown in [Figure 22-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#access_mobile_views_in_chrome_devtools). Then your browser will update the app to display in a mobile or responsive view that you can change based on the options in the dropdown, as shown in [Figure 22-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#responsive_options_in_mobile_view).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2204.png)

###### Figure 22-4. Access mobile views in Chrome DevTools

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2205.png)

###### Figure 22-5. Responsive options in mobile view

## The Sources Tab

If you need to check for things like states and rendering or even when values get set, the Sources tab is going to be helpful. This lets you go through the files in the app and set breakpoints so that you can step through the file execution. You can push Cmd–P on Mac or Ctrl+P on Windows in the Sources tab to search for the file you want to step through. Any file that’s in your repo should be accessible in the Sources tab when you run the app locally, as shown in [Figure 22-6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#searching_for_a_file_in_the_sources_tab). When the app is in another environment, like production, you might not be able to find the exact file because the code has been bundled and minimized.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2206.png)

###### Figure 22-6. Searching for a file in the Sources tab of the Chrome DevTools

After you’ve found the file you want to debug, you can click on the line number next to the code you want to check to set a breakpoint, as shown in [Figure 22-7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#applying_a_breakpoint_in_the_sources_ta). Now when you refresh the page, you’ll see if your breakpoint gets hit or not. If it doesn’t get hit and you’re expecting it to, then you have a new place to start tracking down the bug. If the breakpoint does get hit, you can see all the data for the component at that point in time and step through the code using the panel on the right.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2207.png)

###### Figure 22-7. Applying a breakpoint in the Sources tab of the Chrome DevTools

[Figure 22-7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#applying_a_breakpoint_in_the_sources_ta) shows how to check if something unexpected is happening with your state variables, API calls, or any other functions that get called. Maybe you need to see what’s causing a bug when a user tries to submit a form. Then you discover the input isn’t being parsed correctly by stepping through the process with breakpoints. If you have a breakpoint enabled and refresh the page, it might initially show that no data is available. That’s where your loading states will keep the app from crashing. If you continue to trigger the breakpoints, you’ll get to a point in the lifecycle where the data is loaded. Then you’ll be able to check what the issue might be, such as something directly with the data or a function getting called incorrectly.

## The Network Tab

If you’ve narrowed the problem down to an issue with the data, you should use the Network tab to check your backend requests and responses. This is where you can see all the requests the app makes when a page loads, including components, packages, and API calls. If your API request is a place of concern, you can check the headers that were sent with the request and make sure that you have all the correct parameters. You can see an example of what an API request might look like in the Network tab in [Figure 22-8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#checking_the_api_request_in_the_network).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2208.png)

###### Figure 22-8. Checking the API request in the Network tab

If you see that the bug is an authorization error like a [403 status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status), then you can check the credentials being sent in the request. You might notice that the URL being called isn’t the right one because something should be in camel case or another format. You can see if the API is called or if there’s a status code being returned that the app doesn’t have a way to handle. The Network tab is going to show you all the traffic for a page.

###### NOTE

The DevTools work on any website, so I highly encourage you to go to a site you frequent and look at what’s happening in the Network tab. It’ll probably look like a jumble of indecipherable chunks and calls, and that’s what you want to see on production in most cases. That’s because of performance considerations. If anybody can access the APIs and their data the same way you do locally, that’s a huge problem. That’s why I’ve tried to bring up security throughout this book so that your app isn’t in production with everything transparent for anybody to see.

Once you find the API request you’re curious about, you can look at the response in either the Preview or the Response window. The Preview window will let you see the response in JSON format. The Response window also displays the data in JSON, but in a more expanded view. This will let you look at the data straight from the API response without ever touching your app code. You can see if the data comes in the format you expect or if fields have changed.

That can happen unexpectedly when you’re depending on third-party services. They don’t always announce when they’ve pushed a change to the API, so you might find out at the same time the users do. [Figure 22-9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#checking_the_api_response_in_the_networ) is an example of what a response from a third-party API might look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2209.png)

###### Figure 22-9. Checking the API response in the Network tab

When you’re debugging and you notice that a component gets stuck in a loading state or consistently throws the same error, try checking the Network tab. It might take you in a new direction or at least rule out a hypothesis. The last tab that you should look at usually ties in with the Network tab; that’s the Application tab.

## The Application Tab

The Application tab is where you’ll find your `cookies`, `localStorage`, and `sessionStorage` values. `localStorage` and `sessionStorage` hold key-value pairs for things like access tokens, user language preferences, app IDs, and any other info that helps customize an app for a user. If your bug is showing that you constantly get 403 errors, check this tab to see if some kind of token is set. If it’s in JSON Web Token (JWT) format, you can copy it and decode it with a tool like [jwt.io](https://jwt.io/). Sometimes the bug is as simple as a user not having the right permissions to get a response from an endpoint.

Or you might find that a required value is missing from one of the storage options. Keep in mind that `localStorage` can persist even when a tab is closed, but `sessionStorage` is cleared when the tab is closed or the user ends a session by logging out. That could also be the cause of your bug. Things like user preference settings are kept in `localStorage` while things like whether or not to show a popup on a page or other state-tracking variables are kept in `sessionStorage`. The reason a user can have multiple tabs for the same website open at the same time and have different interactions is likely due to `sessionStorage` being used, so keep that in mind for bugs. [Figure 22-10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#checking_values_in_localstorage_in_the) is an example of what `localStorage` might look like in a production app in the Applications tab.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2210.png)

###### Figure 22-10. Checking values in `localStorage` in the Applications tab

The browser devtools are an incredible debugging option, and they can tell you more about your app than you might find in logs or even in the code. They help you see what’s happening at runtime when everything’s been bundled and minimized. Sometimes it’s the only way you can catch what’s happening, especially with style issues. Once you have located a few places that might be causing the bug, you can start homing in on the details you may find in the logs.

# Debugging in Other Environments

After you’ve checked all the tabs in the DevTools and checked out the mobile view of your app, it’s a good idea to test your app in other environments. Maybe you couldn’t reproduce the bug locally, so see what happens in the develop environment or in staging. If the bug is dependent on production resources, like connections to third-party services, you may need to debug there. If you’re debugging in production, make sure you’re using a test account and not a real user’s account. That way, if you change settings or values, it doesn’t affect anyone.

You could also recruit other devs to help you take a look at the bug in these shared environments. Using other environments like staging can help you loop in the Product team so that you can confirm the expected behavior of the app. Something else you can do is have people on your team check the shared environment in different browsers and at different responsive sizes. That’s definitely helped me uncover some hard-to-find bugs when I get overly focused on a desktop view in Chrome. Remember, your users aren’t going to always use the app like you do. So try to debug from the user perspective when you find bugs.

# Unexpected Places

Bugs don’t always come from the code, the data, or even feature specs. Sometimes, the most unexpected places are the source of the issue, and it can take longer than expected to find the bug. This is where you’ll bring in people from other teams and start looking outside the application your team is responsible for.

You’re going to be exposed to many issues like these. These are the bugs you’ll remember and tell stories about because no one suspected the true source of the issue. Here are a few that I’ve run into at several companies over the years.

Users will always find new ways to interact with your apps that you weren’t planning on. There was an instance where a user was accessing the site on an older tablet, and the page didn’t render parts of the view because our responsive breakpoints didn’t account for that screen size. It took days for us to figure that out and make an update for it. One other case was a user who was trying to interact with the app in the Safari browser and the styles on the page weren’t rendering correctly.

Another time, users weren’t able to see their data because a third-party update had revoked our app permissions without warning and they all had to give us access again. Another case was when certain users were experiencing inconsistent updates when they ordered products because the app server, the server our inventory system was on, and the server for our third-party payments were all configured with different time zones.

A different case was when our CI/CD pipeline was showing that deployments were completing successfully, but the app wasn’t being updated for users. After a ton of digging, we found that the file hash where the artifact was stored wasn’t being updated. That meant something was silently failing in our pipeline.

You might run into things as unexpected as the files not loading in one environment because of an issue with the filenames. One time, production was brought down because a server needed files to start with a capital letter and use underscores compared to the same files working in the development environment.

Don’t forget to clear the cache and check if the issue is happening in other browsers. Also make sure your UI isn’t being rendered server-side. That can make debugging tricky because SSR components don’t rerender.

If you have an issue where an old version of the app runs fine and the new one doesn’t, check the build artifact. Or if you notice that the changes you just released aren’t showing in production, check that build artifact. Going through artifact hashes can be tedious, but if you know there’s nothing wrong with the code, it’s worth checking. There might be some underlying issue where the artifact is cached or it gets uploaded to a different location.

You might also check the CI/CD pipeline to see if anything in the deployment scripts changed. Maybe a new version of Node or your runtime of choice has been installed. Or some packages could have been upgraded in the pipeline. I’ll go over some of the scripts you might have in your CI/CD pipeline in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), but this is an area that can go overlooked and unmaintained until something breaks. If the bug crosses over into the infrastructure, you should check with the team responsible for maintaining it. We discussed handling critical bugs in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide), so if you run into them, you can refer to your incident playbook with more insight from the frontend.

While we used the Chrome browser as an example of debugging, make sure to check your app in other browsers. Sometimes your app will behave differently in other browsers because of the support they offer for things like new CSS rules or web technology updates. You also need to be comfortable debugging across window sizes because there is a huge range of devices that users will access your app with. By checking these areas, you’ll find some of those trickier bugs.

# Conclusion

In this chapter, I covered a few ways you can debug code. One thing I want to mention is that sometimes stepping away from the bug is the best solution. Go take a walk or eat lunch or do literally anything else. When you’ve been focused on an issue or a piece of code for too long, it’s easy to get stuck on one aspect of the problem. I can’t count the number of times a 30-minute walk has helped me come back and fix a bug in the fastest and simplest way possible after I’ve been at it for hours or days.

Reach out to others when you’ve tried everything you can think of. Many times, the bug and the solution end up being simple, but finding the bug source is the hard part. When you work with your team and others, you bring together a lot of different experiences and exposure. While you may have never seen a particular bug or worked with a certain tool, someone else on the team could have expertise in that area. Debugging isn’t always a solo journey, especially when you need to get a fix to production because the bug is affecting customers. Don’t feel like you have to figure it out on your own.