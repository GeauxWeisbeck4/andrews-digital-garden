---
id: 01JH1NYYDGYCF3GRAME743H7KC
modified: 2025-01-07T19:45:28-05:00
---
# Chapter 26. Integration Concerns

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 26th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You’ve gone through the process of getting the full stack app to production, and everything seems to be working smoothly. Getting the app to production isn’t a small feat, and you now have a good baseline to work from, so you can start looking at areas that can be improved. This is when you can really stress-test the app to see how well the frontend and backend are connected and how the third-party services really behave.

As you and other teams start to poke at the app in production, a number of concerns will come up. Some will be surprises to everyone, no matter how much planning you do. Someone might notice they don’t have access to a certain page or there’s an unexpected glitch on the frontend. While you and the team have done all you can to test everything in a preproduction environment, some things will just behave differently. This is where you start to hammer out those things you couldn’t test for in any other environment.

In this chapter, I’ll go over:

- Frontend and backend concerns
    
- Things you should check for with your third-party services
    
- Data and security management in production
    
- Containerization of your app
    

This is a point in the development lifecycle that takes grit and perseverance because you’re so close to being done! These last integration concerns are what will help you, the team, and everyone else know that you can automate your deployments with confidence that they will work consistently. Before, you were able to develop and test each layer in isolation to keep your development focused. Some of the issues that arise at this point will be trickier to pin down than any of the issues that came up before because they could be caused by anything in your environment.

# Frontend and Backend Concerns

You’ve tested the frontend and backend separately, you have unit tests for them, and you have some e2e tests as well. Now you need to start picking at things that can help you maintain the best user experience possible. One of those things will be double-checking your error handling. Consider the case where you’ve tested everything in preproduction and now you have a missing environment variable in production. How does the app respond? Or what happens to the app when your frontend and backend deployments get out of sync because you have a new team member who doesn’t understand the process yet?

Understanding your own deployment schedule will determine when and how features and bug fixes get released. Deploying the backend first is often a good move—unless there are breaking changes in the API. In that case, you need to coordinate the frontend and backend to ensure the least amount of downtime for users or do it during nonpeak hours.

Since the team works on features asynchronously, are there any changes in your code that come from another dev? Sometimes changes get merged into branches silently, which can cause issues for everyone. I’ll go over some Git branching strategies in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), but always make sure you know what code is on a branch before starting the deployment process. Multiple features might be getting merged to the same branch and have incompatibilities that take time to resolve. Talk to one another before deployments are scheduled so that everyone is aware of what’s going on outside of their work.

Have you already factored in browser and device testing? Sometimes older browser versions can change the layout of the frontend because they don’t support features you’re depending on. This is something that may slip past preproduction testing because everyone gets tunnel vision on getting the change released. You also have to check that responsive or adaptive design has been implemented to account for users viewing the app on different screen sizes and devices. Really push the edge cases of what a user might do.

A subtle thing to check is time zone handling in production, especially with any scheduled jobs. Sometimes a different server can throw your timing off by milliseconds, and that can cause an issue with a job running today or tomorrow. It can also lead to confusing data being shown to the user. Figure out what time zones your servers are in and test that your code is running as expected. Checking jobs in production is something that can get overlooked, especially when they run on an interval instead of immediately when an event happens.

###### TIP

Wherever possible, use UTC in your application. This includes your databases, APIs, scheduled jobs, and so on. It will keep everything consistent and avoid bugs related to a server’s local time.

Something else to do is clean up your config files. Go through your _tsconfig_ files to make sure you’re optimizing for production. You might have left some files for preproduction environments in your build, such as mock data or toggles to force switching between different views. You won’t need those in production, and you especially don’t want that kind of developer functionality accessible to users. They add unnecessary bloat to the build artifacts and potentially open attack vectors.

The key is to try to break the app in as many ways as possible before the user does. That’s why it can be a good idea to do some testing in production right after a deployment to make sure the changes for both the frontend and the backend are there. You do have to plan for production testing so that it doesn’t disrupt the users. That can be in the form of known fake user accounts and having fake data that is separated from real data. Be sure to communicate with everyone when you’re testing in production so that weird occurrences aren’t flagged as real issues. It can be uncomfortable admitting that there still could be bugs, even after your extensive testing and the team’s hard work to write good code.

# Third-Party Service Concerns

Not all of the concern is for your code because you’ll be working with third-party services and packages that have code maintained by other organizations. When you switch to your production credentials and turn off test mode in the third-party service, the app may behave differently. This is when you have to verify that you have the service configured correctly.

I remember one case of working with Stripe where we spent so much time testing in preproduction that when we did the release, there weren’t any products set up in production mode in Stripe. The errors we received in production were different from what we tested with, and we even noticed events we hadn’t accounted for. It took a good two months of continuous testing in production before we were confident that users could be notified that the new feature was ready. It required a combined effort among the full stack devs, QA, DevOps, and Product to finally get the feature in a place where all the edge cases were addressed and we had enough documentation to feel confident that we could adequately support our users.

###### NOTE

Just because a feature is in production doesn’t mean users have to know about it immediately. You can have it behind a feature flag, like we discussed in [Chapter 25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments), where only users with certain permissions have access to the new feature. A _feature flag_ is a way to enable or disable a specific feature. Feature flags are commonly used on the frontend to hide functionality for users until the backend has been updated or more testing has been done. This can give you breathing room to test in production without it affecting current users. Don’t forget to turn off the feature flag when you’re ready for all users to see the feature! Not doing so can lead to issues with expected release dates.

External services introduce a different type of uncertainty to your app. One of the things you can do to stay ahead of changes is to scan the websites of the external services you use for announcements of upcoming changes. Pay attention to emails from the services on updates that have been made or new features they are working on. Look through the external service docs at least once a month to see if a beta or alpha version of the service has been released silently. This will help you stay ahead of anything that could break your app.

Remember that not all the breaking changes will depend on version updates to any packages you have installed on the frontend or backend. Sometimes they just change responses or events regardless of the version you have. That’s why your logs and alerts are so important. They’ll help you track down those unexpected changes. Some third-party services are better than others at letting you know updates are coming.

If you go through your logs and trace an error to an external service, make sure the error message isn’t being suppressed by some log message you wrote in your code. You need the raw error from the service to figure out if the issue is on their end or yours. This is a key point when your third-party service requires some action from the user, such as giving your app permission to access their account in the external service. [Google Analytics](https://developers.google.com/analytics) is a good example of an external service that does this.

You could end up getting errors in your app because a user hasn’t given your app permission to access their data. This is another issue I’ve run into on projects, and it took three devs at least a full day to figure out that was the problem because we had custom error messages on the backend for everything. Or you might run into situations where your app uses services that depend on one another. If you don’t get data from one service, then the other service won’t execute as expected. Then you end up having to figure out what to do with the compound issues, and it can take days to make a workaround for it before you even consider how to handle it in the long term.

Even your cloud services will have updates that could potentially cause downtime. It’s up to you to stay on top of these changes and decide how to handle them because users won’t know or care why your app is down. The main thing is communicating with users that you know something’s wrong and you’re actively working on it. You can always work with the Support or Sales team to send out emails letting users know your app will be down for scheduled maintenance when you have to work through issues in production.

It may be hard to test your services in nonproduction environments because of limitations they have. Setting up test accounts or testing for new features with your services can be difficult to manage, especially if you need real user data. Keep that in mind as you prepare your production deployments. That’s why it’s essential to test things in production _before_ you announce they are ready for users.

Setting regular maintenance schedules for external services will help you stay on top of many issues, but sometimes production just brings out the unexpected. So you have to do your best to minimize the impact of your external services and keep alternatives in mind, even if that means building your own solution at some point. It’s an art of balancing flexibility, control, costs, and time. There are some things you can’t do without an external service, like get specific user data. One of the other trade-offs that really comes up in production is that third-party services can expose your users to vulnerabilities and attacks you have no control over.

# Data and Security in Production

Because attacks can come from anywhere at any time in production, you need to work with the Security team to see what they do for testing. Get the Product and Design teams and relevant stakeholders involved in production testing. The more everyone works with the app, the better the organization as a whole will understand it. Everyone being familiar with how the app works across the full stack is invaluable when you’re discussing the roadmap and upcoming features.

Lean into testing for security vulnerabilities or loopholes in the business logic. If you didn’t get a chance to add [security testing tools](https://owasp.org/www-community/Free_for_Open_Source_Application_Security_Tools) like we discussed in [Chapter 8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch08.html#backend_security_considerations), this is a great time to include them. Now that everything is connected, you can see exactly what any user is able to access. Try out different user roles on the frontend to see if the access token you get will allow you to access backend resources you shouldn’t be able to. Check your tokens to see if they expire like you set them to. Take actions that go against the business rules you know should be in place and see if you can break them.

For example, in the app you built, it shouldn’t be possible for a user to order an item that’s out of stock. But can you find a way to bypass that rule? You have limited the number of products you show, but is there a way to break that filter and see all the products? These are the types of business logic loopholes you want to find. They can get more complex as you start thinking of ways to manipulate the app with things like values from the URL, local storage, and session storage.

You should check to see if there are any workarounds for backend logic through the UI. Are strings parsed and validated on both the frontend and backend? It’s more critical on the backend because this is a layer that protects your database. In your forms, see if SQL queries can be submitted as string values and then check the backend to see what it does. This is one way to see if your app is vulnerable to [SQL injection attacks](https://owasp.org/www-community/attacks/SQL_Injection). Generally check if you can force any values that don’t conform with what you have planned for on the backend.

When the user authenticates themselves in the app, are there any [step-up authorization](https://auth0.com/blog/what-is-step-up-authentication-when-to-use-it/) calls that need to be made for further access? If so, is there anything a user can deduce from the step-up process? Things like code verifier strings and encoded user permission strings can be added to the URL in these auth calls, even briefly. These usually won’t be useful on their own, but depending on what you store in the browser, they could become useful info.

Double-check that your tokens expire how you planned and see if the database updates the status on those tokens. Also check that users are not able to manipulate tokens in API requests to give themselves different permissions. Anyone can decode your JWTs, see what the values are, and change them. Make sure they can’t use the altered token to manually call endpoints through a tool like Postman.

This is also a good time to look at the responses that come through in the browser to double-check that you aren’t revealing sensitive information. You don’t want to send things like users’ addresses or any other sensitive info in plain text. This should be handled through JWTs or by simply having HTTPS enabled on your server. Certain input fields, like passwords or Social Security numbers, should be masked so that people around the users can’t easily see the details they type in. The [`password` input element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/password) does this by default, and there are packages like [@react-input/mask](https://www.npmjs.com/package/@react-input/mask) and [react-input-mask](https://github.com/sanniassin/react-input-mask?tab=readme-ov-file#react-input-mask) to help with other fields.

# Containerization of Your App

Something that will come up, even during development, is that your app will run in certain environments (like your machine) but not others. Or you might notice the app doesn’t run the same across preproduction and production servers. That’s usually because there are configs somewhere in those systems that are slightly different. For example, you might run Node 21.2.0 locally, but the production server uses Node 18.19.1.

Differences like that can have a huge impact on the functionality of your app. That’s why running the app in a container is an approach many teams take. [_Containers_](https://cloud.google.com/learn/what-are-containers) let you bundle all the dependencies you need to run an app, such as the version of the runtime you need and any libraries that need to be installed. Containerization is very similar to virtualization. The big difference is that containers don’t fake the hardware they run on like virtual machines do. Containers just run in isolation on the existing hardware, but you can’t do things like install device drivers or interact with USB hardware. So a container is like having the app run in a different world on the server.

The most commonly used tool for this is [Docker](https://docs.docker.com/get-started/overview/), although there are others, such as [Podman](https://docs.podman.io/en/latest/), [Buildah](https://github.com/containers/buildah/blob/main/docs/tutorials/01-intro.md), or [runc](https://github.com/opencontainers/runc). If you can set up containers at the beginning of your project, that will make a consistent development experience for everyone on the dev team. I’ll walk you through the process of setting up containers for your frontend and backend apps using Docker.

Go ahead and [install Docker Engine](https://docs.docker.com/engine/install/) locally so that you can run it on your machine. This is how you can help get a consistent dev environment ready for the whole team. Understanding how you can unblock everyone is an essential skill. That’s why knowing some basics of how to use this tool will come in handy. You don’t have to be a pro at working with containers, but you might find that it’s interesting, and you can dive as deep as you want.

For now, you’ll need to make some updates to your _vite.config.ts_ file. The values you have in this file and the _tsconfig_ files are going to directly affect the build artifact you get for production. So if you run into issues with the build, check your config files first. Here are the updates for the _vite.config.ts_ file:

```
...
```

We’ll start by creating an image for the frontend app. An _image_ is a template that has the instructions for creating a container with all the libraries and dependencies required for an application to run. You create images by writing a [Dockerfile](https://docs.docker.com/reference/dockerfile/) that has Docker commands to build the container the app will run in. These are similar to the commands you would run in a terminal on the server to set up a container. Here’s a simple Dockerfile to run the production build of the app in a container:

```
FROM
```

Six instruction keywords followed by arguments are used in this Dockerfile. First, the [`FROM`](https://docs.docker.com/reference/dockerfile/#from) instruction is how you set the [base image](https://hub.docker.com/) that the container is built on. The argument for this instruction is commonly the runtime version you want to use. Next, the [`WORKDIR`](https://docs.docker.com/reference/dockerfile/#workdir) instruction sets the working directory in the container for the other instructions that are run in the Dockerfile. This is how you can set the location where your app files are copied to in the container.

An instruction that gets used a lot throughout a Dockerfile is [`COPY`](https://docs.docker.com/reference/dockerfile/#copy). This is how you’re able to move files from your computer’s directory into the container. The [`RUN`](https://docs.docker.com/reference/dockerfile/#run) instruction is how you execute commands that you would typically run in the terminal in the container. Next, the [`EXPOSE`](https://docs.docker.com/reference/dockerfile/#expose) instruction lets Docker know how to expose the container’s specified port to the host server. This is like the virtualization comparison earlier. The container is still isolated, but there’s a way for the host server to interact with it. Finally, the [`CMD`](https://docs.docker.com/reference/dockerfile/#cmd) instruction sets the command that will be executed when the container is running from an image.

# RUN ONLY THE FRONTEND IN CONTAINER DURING DEVELOPMENT

The `npm run preview` command is used here as an example since you’re running the container locally. The [Vite docs](https://vitejs.dev/guide/static-deploy.html#deploying-a-static-site) don’t recommend using this in production, so this is just for local development:

```
FROM
```

With your real production frontend, you should serve the app from an S3 bucket, Vercel, Cloudflare, or something similar.

To create an image from this Dockerfile, enter the following command in the terminal of your frontend app directory

docker build . -t “dashboard-ui:v1.0”

This command is how you build the image defined in the Dockerfile. The `.` between `build` and `-t` is very important and also easy to miss. This is how you specify that the command runs in the current directory. The `-t` option is how you set the name and tag for the image, which are `dashboard-ui:v1.0` in this command. You don’t have to name the image and can just use the generated ID, but I’ve found naming to be useful when you have a lot of images.

After this command finishes, you’ll have an image that you can turn into a container. You can see your current Docker images with this command:

docker images

And you’ll get an output like this:

REPOSITORY     TAG           IMAGE ID       CREATED          SIZE
dashboard-ui   v1.0          8179b8302716   `21` seconds ago   `1`.43GB

Finally, you can run this command to start a container based on that image:

docker run -d -p `8080`:8080 dashboard-ui:v1.0

The `-d` option is how you run the container in detached mode, which makes the container run in the background instead of displaying the process in your terminal. The `-p` option is how you map the host port (the port on the server) to the container port (the port inside the container) so that the app is exposed and running on the port you specified in the Dockerfile. Now you should be able to access your app via the container at _localhost:8080._

You can also see this container running in your terminal with this command and output:

docker ps

This is similar to the [`ps` (process) command for Unix systems](https://www.javatpoint.com/linux-ps#:~:text=The%20ps%20command%20is%20used,ID%2C%20command%20name%2C%20etc%20.). So you’ll see the currently running processes in the container, as in [Figure 26-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch26.html#docker_container_running).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2601.png)

###### Figure 26-1. Docker container running

You’ll have a similar Dockerfile on the backend. The main difference between this and the frontend will be the `CMD` instruction argument to run the app. Here’s what that will look like:

```
FROM
```

As you create new images, you should be aware of golden images. A _golden image_ is a template with the environment, tools, packages, and security settings predefined for all the other images you make. These are good to have because they are rebuilt to keep the latest security patches installed, they speed up app development because you and the team won’t have to worry about configs, they maintain consistency across all your images, and they can be used to automate deployments for new apps. If you have a dedicated DevOps team, they will typically be responsible for keeping the golden image up to date.

# Conclusion

In this chapter, I went over some of the integration concerns that will arise in production and some methods for solving them. Some of the more unpredictable issues will come from your third-party services. Since you don’t have control over when they make changes or what those changes will be, you have to monitor them. Regularly go through their docs and announcements and do some testing in production to stay on top of changes. Many of the integration concerns can be managed with coordination between the developers and the other teams you work with.

Most of this chapter explained setting up Docker and creating containers to keep your production deploys consistent. You should partner with the DevOps team to keep those Dockerfiles up to date with both security and app needs—we’ll talk more about that in the next chapter. It will benefit you to understand some of the basics of Docker so that you can help maintain the images for your apps. That way, when small updates need to be made, you can help your team stay unblocked because they won’t need to rely on the DevOps team for things like version updates or changes to the scripts that are executed.