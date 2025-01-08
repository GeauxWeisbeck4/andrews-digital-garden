---
id: 01JGFP7S7JD67MRT802Q4W2ARP
modified: 2024-12-31T20:03:58-05:00
---
# Chapter 9. Backend Debugging

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 9th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

No matter what you do, you’ll have to have solid debugging skills and processes. Being able to sort through possible issues quickly is going to get you to the root cause efficiently. There’s not a strictly right or wrong way to perform your debugging, although there are some things you should definitely check for.

Debugging is an art of persistence with the discipline of a systematic approach because it’s going to take some twists and turns and lead to some dead ends. Often, the code causing the bug isn’t very complex. It might be a one-line change, but the path to finding that one line can take days. I’m going to give you some tools and strategies that will hopefully get you to the root cause as fast as possible.

In this chapter, I’ll cover:

- Detailed log messages
    
- Environment configurations
    
- Tools you can use
    
- Strategies for tracing bugs
    
- How to help others find bugs
    

Debugging is one area where you will be looked up to for help. When some of the less experienced developers run into issues with their code, you’ll be one of the people they turn to. This is a great opportunity to mentor another developer as well as learn from them. Some bugs are just too big for one person to track down because of how complex a system gets. Over time, all the team members will develop expertise in specific parts of the app, which means you’ll get farther faster when you help one another.

# Detailed Log Messages

One way you can start helping the team is by writing log messages with useful information. There isn’t a limit to the number of logs you can include in a method, so you’ll have to decide what information is most important for everyone to see. Let’s take a look at some of the log messages already in the project and see how you can strengthen them.

The _stripe.service.ts_ file is a good place for more logging. This time, we’ll go through the `createProduct` method because it has quite a few moving parts. You’re making a database update and calling different endpoints from a third-party service. If something happens in this chain of calls and updates, how will you be able to trace what happened? Let’s add some logs and improve the existing ones:

```
public
```

There are nine log messages here, and you could add more if you want. You’ll notice that each one that passes an object in the message is stringified. Without this, you end up with a lot of instances of `[Object object]` in your logs, and that’s not going to help you debug. You might even include the environment in your log messages specifically, but many tools like [Datadog](https://www.datadoghq.com/) or [Sentry](https://sentry.io/welcome/) have something like that built in. If you want to see all this information at a glance, it doesn’t hurt to include it, though. You’ll learn more about the specifics of monitoring tools in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide).

You can also see that different log levels are being used. The common log levels are trace, debug, info, warn, error, and fatal. The _debug_ level is used for diagnosing issues or when you’re working in a test environment to make sure everything is running correctly. The _trace_ level is like a more verbose form of the debug level. Trace logs give you all the info about what’s happening in your app and in third-party dependencies you use. These aren’t commonly used and typically come up only when you need granular visibility into a specific part of the app.

# HOW TO DETERMINE AN ADEQUATE AMOUNT OF LOGGING

Here’s some more advice from Jeff Graham on logs:

> Determining the “right” amount of log messages can be tricky. You’ll want as much info as possible to debug, but it can be cumbersome to scan through a long list of log entries. It’s good to test logs locally: could you diagnose a problem with your API by only reading the logs? There are also products to manage and search logs: CloudWatch, Splunk, Datadog, New Relic, and many more.
> 
> Logs can also unexpectedly contain PII or other secure details: credit card numbers, API tokens, addresses, etc. It’s important to make sure these details are removed or obfuscated from production logs.
> 
> Lastly, don’t be the engineer who doesn’t read their logs until the first production issue occurs. You’ll likely find yourself wishing you had better details to use in debugging.

The _info_ level is very commonly used in logs to show that something happened. This is how you can see authorization requests, determine what parameters were used in an API call, and keep an eye on the state of the app. Info logs shouldn’t contain important details that would be needed if an issue arises with the app.

The _warn_ level is used to show that something unexpected happened in the app, but it didn’t stop the app from running. This could happen if data is parsed incorrectly, for example.

The _error_ level should be used whenever something stops the app from functioning as expected. This typically includes 4xx errors and 5xx errors. An example of an error log would be if Stripe were down but a user could still try to place an order through a backup service or if one of the user’s login methods isn’t working. A _fatal_-level log is reserved for when a core piece of business functionality is down, such as if your users are trying to get their order info but can’t because the database is having issues or if a user can’t log in to the app because all the login methods aren’t working.

It helps to use logging at the correct level. Being able to quickly distinguish between error logs and debug logs will help you get answers faster than if everything is at the same level. Check for things like the API version in the request, any parameter issues, the source of the traffic, and the times and dates logs are generated. These are some quick-start points to figure out what’s happening. That way, you can catch patterns like certain errors being thrown when specific values are sent. This can also help you find concurrency issues because you’ll be able to see when a request call started, when it returned, and anything that happened in between.

Another thing to be cautious of is having nested `try-catch` blocks. Pay close attention to how you handle errors because it could lead to a case where errors are unexpectedly suppressed through these blocks.

Something to keep in mind is that when you log errors from somewhere else, you need to pass that original error within your log. It’s easy to hide the real error message when you might catch the error with custom handling. Anytime you use a third-party service, it will help to log the requests you make as well as the responses you receive. Here’s a snippet to highlight this from the previous example:

```
try
```

Since you have custom error handling here to log the error, you should be intentional about preserving the original error response that comes directly from Stripe and triggers the `catch` part of this block. That’s what the error log does in the example.

###### NOTE

This is a place where testing can really shine because ideally, you should have a unit test written for each possible path the code can take. That can help you catch bugs during development, too. You should have a test case for each error scenario, which can help you find if messages are suppressed under custom error handling.

When you go through these logs, you’ll get all kinds of good info. You’ll be able to filter logs based on the time when issues happen, which gives you insight into how code was executed. For example, if there are errors creating products on Stripe, you’ll be able to see when calls were made and what data was sent to cause the errors. You might find things like multiple calls happening at the same time, leading to race conditions that then lead to data inconsistency.

If you have a bug and nothing in the code looks wrong, go look in the logs. Here’s an example of what some log entries might look like:

[Nest] 90322  - 06/19/2024, 1:43:22 PM   DEBUG [OrdersService] Create order called with: {“user”:{“id”:5,”roles”:[“Customer”]},”order”:{“total”:23.54,”userId”:5,”products”:[{“id”:”aperiam”},{“id”:”ipsa”}]}}
[Nest] 90322  - 06/19/2024, 1:43:22 PM   ERROR [ExceptionsHandler] Cannot read properties of undefined (reading ‘includes’)
TypeError: Cannot read properties of undefined (reading ‘includes’)
    at OrdersV1Controller.create (/Users/milecia/Repos/dashboard-server/src/orders/orders.controller.ts:29:21)
    at /Users/milecia/Repos/dashboard-server/node_modules/@nestjs/core/router/router-execution-context.js:38:29
    at processTicksAndRejections (node:internal/process/task_queues:95:5)
    at /Users/milecia/Repos/dashboard-server/node_modules/@nestjs/core/router/router-execution-context.js:46:28
    at /Users/milecia/Repos/dashboard-server/node_modules/@nestjs/core/router/router-proxy.js:9:17

[Figure 9-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#logs_in_gcp) shows an example of logs from Google Cloud Platform (GCP) so that you can see a different tool, but don’t worry about the details of what’s in the screenshot. You’ll get a better look at logging tools later in the book.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0901.png)

###### Figure 9-1. Logs in GCP

Here’s what the details of one of these logs might look like:

```
{
```

You’re going to check the time that the logs occurred, the services they came from, and what information is contained in them. If you’re debugging, focus on the logs that are from the endpoint or service you’re troubleshooting. Check the status codes that are returned because that can take you to the exact location of the bug in the code. Look for any auth messages to see if that’s where the problem is. From there, see if any missing or invalid parameters have been passed. Also check the frequency of the bug in the logs and see if you can trigger the action that creates the log yourself. That’ll help you figure out how to resolve it with a systematic approach.

# Environment Configurations

I’ll go into way more detail about things like CI/CD pipelines, deployment tools and strategies, and other related infrastructure topics starting in [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments). For now, I’ll focus on specific environment configurations that may be in your codebase or somewhere that you have easy access to, like [CircleCI](https://circleci.com/). The following code snippet is an example of an environment config file, and [Figure 9-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#environment_variables_in_circleci) shows these environment variables in CircleCI:

```
# .env file
```

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0902.png)

###### Figure 9-2. Environment variables in CircleCI

When you add new third-party services or need new environment variables, make sure that they actually get added. This is an easy step to overlook in the process of getting everything else out. These types of errors will usually come up quickly in your CI/CD pipeline, and you can add the values in the correct places. This is a good place to make a technical ticket so that everyone on the team remembers it needs to be done, not just you.

See if any roles are missing from the request based on the access token. This might be something a little trickier to pin down, depending on how your logs and error handling are set up. Ask around and see if any new permissions were added based on security updates or new features. Also check that your services have the right roles and permissions being passed to in their requests.

Make sure your changes are getting deployed to the correct environment. This is especially true the first few times you do any deployments in new environments. Check out the deployment artifacts and see if file hashes match what you expect. A _deployment artifact_ is the actual build with all the bundled code and metadata that is produced when your code is compiled to what gets run on the server. You can see these in your cloud platform in a service like an Amazon S3 bucket. The _file hash_ is the unique string identifier for each artifact. Sometimes you might see that an artifact has been deployed, but when you check the hash, you find out it’s the same version as a previous artifact.

When you don’t see the changes you pushed, it becomes harder to debug the issue because it’s not related to your change. This happens occasionally, and it can take some time to dig through and get to this check.

If you have a complex deploy pipeline that moves your application across different parts of the infrastructure, check each stage. You might find success messages that still have errors being thrown by the infrastructure service. Go through the logs in each stage to ensure that no unexpected errors are sneaking through silently.

###### NOTE

As you get deeper into debugging issues, you will inevitably end up looking at some part of the infrastructure. That’s where some lines of responsibility can get blurred between the dev team and the DevOps team. Both should look through infrastructure logs to see if a command or something didn’t execute as expected and determine if that’s a code issue or an infrastructure issue. That will help decide who handles the issue and makes the updates.

This brings up another reason to make and maintain a system architecture diagram: you know exactly how everything fits together, and you can see where things might have gotten tangled. Seeing the architecture will remind you of things like having an updated access token for your background and cron jobs. It’ll also help you consider other parts of the system that could be causing the issue, which you wouldn’t have initially thought about.

# Strategies for Tracing Bugs

When you’ve been digging through logs and code for a while, it’s easy to get stuck in a certain place. You may have been looking at the same few files for hours, maybe even days. Go for a walk or work on something else for a while. It always surprises me how fast I can figure out a bug once I’ve had some time away from it.

If you can’t break out of the weeds, then shift your focus to another part of the app that might cause the issue. Ask a teammate for a few minutes to talk through the problem with them. A rule of thumb I’ve found useful is to try debugging on your own for about 30 minutes, and if you haven’t gotten past the initial error, reach out to someone else. Giving yourself distance and getting help from someone not as close to the problem give you a different perspective to approach debugging with. You could consider using online forums like Stack Overflow or a Discord server to help you get going. Just remember to make your code generic so that you aren’t leaking organization info. Try all of these to make your debugging super effective.

###### NOTE

Debugging is definitely an area where AI tools like ChatGPT can help. Sometimes you can type your question into the chat box, and the output it gives you is similar to talking with another dev. Depending on the rules your organization has around protecting the code, you may even be able to copy-paste a snippet in the tool and get some really good feedback on things to try.

Use _console.log_! This is the simplest debugging tool you have available. You can log every line of code if it helps you figure out what’s happening. Using console messages can show you any unexpected calls or data transformations as they happen. Also don’t be afraid to try some hacky code to reproduce the issue and test out a theory or a potential fix. When you’re looking for the root cause, you don’t have to keep code PR-ready. Test out your thoughts, and then you can clean it up.

Here’s an example of how you might write hacky code to figure out if an endpoint is working. You can slowly uncomment code and manually update the user and order inputs to see if one of those is causing the error to throw:

```
// if (!user) {
```

Knowing the tools to help you test your hypothesis will give you more confidence in checking more things. If you think there’s an issue with a request or response, try using [Postman](https://postman.com/) to make the request like a client would. When you want to check if data has been updated correctly, use a tool like [pgAdmin for Postgres](https://www.pgadmin.org/download/). It gives you a quick, graphical way to interface directly with the database, which can help in pairing sessions.

Make some fake data right there in the code and run it locally. Sometimes you just need to hack together some code to reproduce a bug. In the process of reproducing the bug, you may solve the problem. You can find some really interesting third-party bugs this way as you start seeing what the actual response is. You might even be surprised by some unexpected data transformation happening because of a bad function call.

Debugging the backend will take you into the infrastructure occasionally. You’ll run into cases where data didn’t get deleted because an event threw an error silently two days ago. This is a large process that touches many parts of the system, which can take more time to debug. Get comfortable with it and look through the services as you develop features and fix other bugs. That way, when a big bug comes up, you’re comfortable with what you see in the infrastructure.

Double-check the business logic. Sometimes the system is working exactly like it’s supposed to, but there are some business conflicts. As you talk through the issues with the Product team, confirm that the business rules are written in the code as expected. This one can be more subtle because it means that you have to have some intuition about how the product works; that’s what you need to truly understand what’s a bug and what’s a feature.

I’ll go over incident handling more in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide) and [Chapter 23](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch23.html#full_stack_deployment_setup), but it’s worth mentioning a bit about it here. Keep people up to date with what you’ve checked out so far. This will be especially important for large production-level bugs that are affecting users. Communicate your findings as you rule out things. This will help you keep track of what you’ve done, let the stakeholders know what is going on, and give everyone an idea of how things are going and who can help.

# Helping Other Devs Debug

Knowing how to help others debug their issues is a huge skill to have. This is where you can bring in a few approaches. First is the scientific method, where you come up with a theory about what the problem is, create a test, and then run the test. A good example of this is a forbidden error like the following:

```
if
```

You and the other dev could form a hypothesis that the user doesn’t have the `Customer` role. Then you could test it by calling the endpoint locally with Postman and passing user data with the correct role. After you run the test, you can check the terminal the app is running in to see what errors or messages come up. Then you can repeat this process with a different hypothesis to try something else. Or you can both come up with several hypotheses, try them individually, and report back with your results.

Another method is to construct a specific flow diagram that shows the exact sequence of events and code touched in the pathway of the bug. You and the dev can use a tool like Miro or even just a Google Slide to quickly diagram what’s happening. [Figure 9-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#debugging_flow_diagram) is an example of a flow diagram for the same bug as in the previous example.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0903.png)

###### Figure 9-3. Debugging flow diagram

The next approach is to start breaking down the problem space. See if you can trace the bug to the controller or service. If it’s in the controller, then you can stop breaking things down. If it’s in the service, see if you can break it down to either service code or something with the database or a third-party service. Keep breaking down the path of the bug until you reach the part of the app it’s most likely coming from.

The next approach you can take when helping someone else debug is “rubber-ducking” with them. Just let them talk through the issue while they have your attention. It’s surprising how many bugs are resolved just by talking it through with someone else. The act of explaining a bug to someone in normal language can trigger a new thought or reveal something that becomes really obvious when it’s said aloud. This can be done on a call or even through messages. The key is to get them to describe the bug and what they are trying to do. Sometimes you don’t even need to say or do anything, and they’ll figure it out in a few minutes!

The last approach I’ll cover is helping them use tools like [ChatGPT](https://chat.openai.com/) or [Copilot](https://github.com/features/copilot). These tools can act as automated rubber-ducking because they can analyze your code for inconsistencies. They also make you think about how to describe the issue so that you _can_ get help from them. AI tools like these can help simplify the issue for them and give them a blurb about what’s going on with the code.

# Debugging Checklist

This isn’t an exhaustive list of every possible thing to check for when you’re debugging the backend, but it is a pretty long one. Mainly, I wanted to point out the specific things you’re looking for at each check. It’s like cutting into a cake with precision. Hopefully, you can use this checklist as a good starting point if you get stuck in your debugging process:

- Check the application logs
    
    - Are there any errors? If so, where are they coming from?
        
    - Is there a certain time when the issue started happening?
        
    - Can you find any helpful stack traces?
        
    - Did you go to the line of code highlighted in the errors if there was one?
        
- Check the access token
    
    - Is it expired?
        
    - Does the requester have the correct permissions?
        
    - Is the token malformed like someone tried to edit it?
        
- Check the request
    
    - Are the correct parameters present?
        
    - Do the parameters have the correct types?
        
    - Is it a [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (Cross-Origin Resource Sharing) issue?
        
    - Have you checked the headers?
        
- Check the response
    
    - Did it send the expected error?
        
    - Was the data in the right format?
        
    - Are data transformations happening before the response is sent?
        
    - Are any third-party services being called before the response is sent?
        
- Check the database
    
    - Have the expected fields been updated or queried?
        
    - When was the last update for the data?
        
    - What happens when you run a raw SQL query compared to the query made from the app?
        
- Check the environment
    
    - Did you add the new environment variables?
        
    - Have you updated existing environment variables?
        
    - Did you add the environment variables to all the infrastructure tools and code that use them?
        
    - Has a third-party service rotated values?
        
    - Are you deploying to the correct environment?
        
    - Can you check the artifact version?
        
    - Have you asked someone on the DevOps team to pair with you?
        
- Check your third-party services
    
    - Have any new versions been released?
        
    - Did any fields change their names or types?
        
    - Can you go through the docs and check the request and response types?
        
    - Has anything changed in the dashboard?
        
    - Do you need a new API key?
        
- Check the code
    
    - Are all your package versions up to date?
        
    - Have you gone through the methods line by line?
        
    - Did you try breaking the code down to its simplest form, like in the hacky example, and slowly add code back?
        
    - Are there any `try-catch` blocks that might suppress errors?
        
    - How are database errors handled?
        
    - Does the business logic make sense?
        
    - Are there other parts of the code that touch this one?
        

As you go through your career, you can build your own personal debugging list. I have something like that, and it helps me cross things off a lot faster without taking as much mental energy. It’s tedious checking all these areas, but it’s a thorough way to rule out root causes.

# Conclusion

Debugging is going to be one of the areas of development that will require persistence. There may have been times when you wondered how another dev fixed a bug in 30 minutes that you’ve been looking at for days. It’s OK to ask for help because it means you understand how others can help you eliminate potential sources of the bugs.

As your app grows, so will the complexity of your debugging. There’s no perfect way to find bugs. Sometimes you’ll need to hack together some code, use a lot of console messages, or even change data directly in the database. Remember that this is one of the artistic parts of being a dev. Any approach is valid as long as it doesn’t break other things!