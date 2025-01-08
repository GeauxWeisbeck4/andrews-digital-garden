---
id: 01JH1NWABTRSXV84CHYEZ4ZJ3C
modified: 2025-01-07T19:44:02-05:00
---
# Chapter 23. Full Stack Deployment Setup

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 23rd chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Congratulations! You’ve made it through the initial challenges of setting up a backend and frontend app from scratch! Once the app is in a stable place with architecture decisions, third-party services, code conventions, package choices, and all the other pieces you’ve implemented throughout the rest of this book, you and the team are finally in a place where you can churn out features quickly and efficiently. Anything you do from here on will be to help the product grow and accommodate user needs over time.

Now it’s time to shift your focus to setting up your deployment pipelines and making sure everything connects correctly from the database to the backend to the frontend and all the jobs in between. This is where the Security team will do more testing because the complete full stack app is ready. Before I jump into topics like CI/CD pipelines and integration concerns, it’s important for you to take a step back and see how you and the team performed as full stack devs.

In this chapter, I go over:

- Other teams involved in the deployment process
    
- Checks for the frontend and backend
    
- Demos and team retrospectives
    

Although you’ve made it through this greenfield project, it’s essential that you do some self-reflection and encourage the rest of the dev team to do the same. After working on getting a full stack app deployed, you may have some notes on what could be done better on your future projects. Let’s start by discussing the other teams you’ll work with to handle all your deployments.

# Teams Involved in Deploys

There’s usually at least one other team you’ll work with to get all the infrastructure built in your cloud, but sometimes there are multiple teams. These teams commonly include Infrastructure, DevOps, and site reliability engineering (SRE).

The Infrastructure team typically manages the cloud platform itself. This team handles things like hardware, networking, storage, and server resources. They choose the best service options based on conversations they have with you and the DevOps team. The Infrastructure team has more experience managing Linux-based systems, and they help implement security practices at that level.

The DevOps team focuses on automating manual tasks like moving artifacts around or writing scripts to execute repetitive processes. This team works on implementing CI/CD pipelines with tools like [CircleCI](https://circleci.com/) or [GitHub Actions](https://docs.github.com/en/actions/quickstart); we’ll discuss this more in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline). DevOps also tries to foster collaboration across development and operations teams. When it comes to infrastructure automation, configuration management, and tooling for fast and reliable software delivery, you’ll reach out to the DevOps team.

If you’re at a bigger organization, you may have an SRE team. SRE teams focus on the reliability, performance, and availability of your applications. They work with monitoring tools, incident responses, and performance optimization. When you hear about service-level objectives (SLOs), it’s usually with respect to metrics that this team measures and tries to maintain.

All these teams work with the dev team to ensure that apps have adequate resources to run reliably. They each manage different parts of the deployment process. It usually starts with the Infrastructure team setting up the foundation in the cloud platform. Then the DevOps team adds automations and pipelines to get the app to the correct services in the infrastructure. Finally, the SRE team monitors the deployed apps and makes sure that any changes to ensure uptime, performance, and incidents are handled.

# FULLY UNDERSTAND THE DEPLOYMENT PROCESS

Here’s some advice about deployments for all devs from Jeff Graham:

> I encourage all engineers to learn and own their app’s deployment process. While Infrastructure or DevOps teams might help (depending on company size), it’s important that you fully understand every detail. You best know your code, tests, dependencies, build commands, etc. and can therefore make the best decisions. Besides improving your ability to troubleshoot, it will also increase your knowledge about DevOps, cloud services, and more.

These teams help you and the dev team deploy apps without worrying about the underlying systems. You’ll have discussions with all these teams many times as you deploy to production while you slowly expose the app for more users to access it.

###### NOTE

If you’re at a small organization, all of these tasks (DevOps, infrastructure, and SRE) could become part of your workload, and you’ll have to do your best to figure it all out. This can seem like an overwhelming undertaking, but take your time, read through docs, and reach out to people who work on these systems. Be clear with your organization that these are not areas you specialize in so that they have the correct expectations.

# Backend and Frontend Connection Steps

At this point, you have the backend built, it’s been tested, and you’ve been adding calls to it from the frontend. What you’re doing now is double-checking everything you have in place to make sure the rollout to all the users goes smoothly. This is when you really put the infrastructure to the test and iron out the remaining wrinkles. You already have some stuff deployed to a staging environment and maybe even production, but with limited traffic.

Everything might be working perfectly, but as soon as you get more users and add more features, you’ll notice some unexpected behavior. To help make this process as smooth as possible, here are some steps I use when I’m doing any full stack connections.

## Backend Steps

Some devs will argue that you can start with the frontend, but I’ve found it much easier to have all the layers that support the frontend in place first. You can start with the frontend if it’s more comfortable for you, but you’ll likely end up doing some redundant work. There isn’t a concrete rule about what should be handled first, and it’s going to vary depending on the nature of the app.

Check your database connection with a database tool, such as [pgAdmin](https://www.pgadmin.org/). It’s one thing to develop against something locally or in a develop or staging environment. But when it’s time to go to production, you need to make sure you have the right connection string, credentials, and data schema. Make sure the product you’re using is configured correctly so that it doesn’t return errors. If you expect this app to work at scale, this is a good time to consider [connection pooling](https://www.prisma.io/dataguide/database-tools/connection-pooling#what-is-connection-pooling). You might need to involve the Infrastructure or DevOps team here because they usually provision these resources.

Seed the production database to make sure data is populated correctly. It doesn’t have to be a lot of data, but it should be enough to check that all your relations, tables, indexes, and values write how you expect. Since you’ve already done this in the initial setup, you should be able to run the same script with maybe a few modifications to the data you add.

Query the seeded data. Try out a few queries just to make sure you fetch the data you expect. Also try to update the data to check that any constraints are applied correctly and that invalid data types throw errors.

Set up the production database connection in your backend app. Once you’ve tested with the database tool directly, you know the database is stable and shouldn’t be the source of issues. So you can confidently connect the app to the production database with the environment variables you set.

Double-check that the environment variables are set to the right values in the production environment. This is easy to miss when you’re excited about everything working during a manual test.

Work with the Infrastructure team to get the backend server configured correctly. That will include setting the security for accessing the backend, handling SSL certificates, determining where parameters for scripts are stored and the regions the app will get deployed to, and invalidating your cache if you have one in place. This will also include setting up any user roles and permissions you’ll need for cloud service access, which is separate from the roles and permissions in the app.

Set up logging, monitoring, and alerts for the database and backend API. You want to have these in place before you open the app to users so that you can adequately support them. Understanding how your app works in production by monitoring it is going to help you and the team find the places where the app can be improved over time. You can trigger alerts based on certain errors in the logs or based on monitoring things like CPU usage and memory usage. That will keep the team ahead of potential attacks or let the SRE and DevOps teams know it’s time to scale the resources.

Check that the backend API is working with a tool like [Postman](https://www.postman.com/). At this point, your backend should be deployed to its production environment, so make sure that requests are returning the expected responses. Look for things like header configurations, permissions, and any encoding and decoding that should happen. This part can be automated with tests so that you have an established baseline for the expected behavior.

Check that your third-party services are working correctly. This is a critical point because you’re switching from test credentials, which might have limits on your access to the third-party functionality. Take your time and go through as many scenarios as possible with the Product team to make sure you’ve got everything configured in the app and in the service dashboard. You’ll also want to check the environment variables again to make sure you’re using production credentials now.

Don’t forget about your background jobs and cron jobs because when they are running in production, they can cause a lot of issues and shouldn’t be an afterthought. Trigger each job and see if it updates data how you expect. Some jobs may not be easy to test if they depend on external services, so write a ticket to come back and check them as soon as data or events are happening in the external services.

Make a plan for when things go wrong. We’ll go over this in more depth in Chapters [25](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch25.html#making_deployments) and [26](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch26.html#integration_concerns), but you should have an incident plan ready for when deployments go wrong. The plan will include some type of rollback strategy and communicating time frames to customers and stakeholders. You’ll want a plan ready because in the midst of an incident, everyone wants to put the fire out as quickly as possible, so it’s hard to do things systematically.

Document all the steps for the database and backend. When it’s time to do more deployments, having a detailed document that outlines what needs to be done will keep everyone on the same page. While a lot of this process will be automated, sometimes automations break. This is one way you have a backup available for all the teams. You should consider making a backend-specific architecture diagram to highlight the connections you have between the app and the infrastructure services. An example of this is shown in [Figure 23-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch23.html#production_backend_architecture_diagram).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2301.png)

###### Figure 23-1. Production backend architecture diagram

## Frontend Steps

With the backend in production, you can really get to work on connecting the frontend. All the data should be available for you via the API, so let’s go through the steps to deploy the frontend to production. Some of them will be similar to what you did on the backend.

Work with the DevOps team to get the location for the frontend app configured correctly. This may be a little different from your backend, and DevOps may do both at the same time. It just depends on how they like to work. You still have the same considerations as the backend in terms of environment variables, regions, and cache busting. The cache might be handled differently because you’ll probably want a CDN on top of the frontend for performance and uptime.

Implement a CDN to improve page load times, as discussed in [Chapter 20](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#frontend_performance). Since you’re already working with the DevOps team, you might as well get this in place. [Cloudflare](https://www.cloudflare.com/application-services/products/cdn/) is a popular CDN service that lets you manage your frontend apps securely, and it comes with a lot of built-in functionality to make it easier to handle any issues that come up. Another popular one is [CloudFront](https://aws.amazon.com/cloudfront/) if you’re working with AWS.

Optimize your bundle. Make sure the code is minimized and the assets are compressed. This will likely happen in your CI/CD pipeline, which we’ll build in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline). This is the artifact that will sit on the frontend server and is the code that users will interact with. That’s why you want to make sure to have a small bundle size.

Check that logging, monitoring, and alerts work. The frontend is what represents the product to many users, so you want to be just as aware of what’s going on with it as you are on the backend. You may find it helpful to keep separate logging and monitoring dashboards for the frontend so that you can quickly find information. Same with the alerts because different people may need to be notified so that issues can be handled as efficiently as possible.

Verify that the frontend works on all the browsers you plan to support. We often focus on the browser that we develop in and don’t check the others. [CanIUse](https://caniuse.com/ciu/comparison) lists the most commonly used browsers and the features they support, so check out the app in these other browsers once the app is deployed. You should also use tools like [SauceLabs](https://saucelabs.com/) to make sure the app works across different devices and browsers. This should be part of your testing and debugging during development, similar to what we discussed in [Chapter 22](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#frontend_debugging), but if it got overlooked, you definitely need to do it now.

Check that all your forms work as expected. You’ll get edge cases, but you need to make sure the core functionality is working. The UI should update accordingly, and you can use tools like the Network and Application tabs to see if your requests and responses are what you need them to be.

Use your app from the frontend and see the database changes in production. Go through several of the user flows with the Product team to make sure everything’s working. This is another key time to have demos. Have Product sit on a call with you as you go through the app. Then have them take over and go through the user flows as well. Your app should be usable by anyone who hasn’t been involved in the development process. As developers, sometimes we get so used to our workarounds that we forget nobody else will do that.

Create the documentation for the frontend. You can go as deep as getting into the details of how the app is structured and how it works on the component level or have a high-level diagram depending on what is useful for the actual integration visualization. Creating a diagram of your component tree might be a task you did earlier, or it could be something you help someone else on your team do. Keeping it at a higher level at this phase can make things clearer as you integrate so that you don’t start mixing in things like component state with infrastructure. [Figure 23-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch23.html#production_frontend_architecture_diagra) is an example of a high-level diagram.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2302.png)

###### Figure 23-2. Production frontend architecture diagram

## Cleanup Steps

After you’ve gone through all the steps of getting the entire stack deployed, there are still a few more things you need to check just to make sure you’ve accounted for as much as you can. After everything has been deployed, take a little break. Those 30-minute walks or just time away from the computer will give you a chance to come back with a fresh mind.

Now that the full app is in production, you want to make sure you can support it. One way to do this is to check the logs and see if your connection tests are showing there. Hopefully, you don’t have any errors yet, but you should see some activity. Check if your monitoring is working correctly by looking for the traffic from your connection tests. Trigger those alerts to make sure they’re going to the right emails and channels. You should also test your incident and rollback plans to see how they really work.

You don’t actually know if your rollback plan works until you’ve tested it, but this can be a disruptive process. In the disaster recovery (DR) world, there’s a saying that you don’t have a backup plan if you haven’t restored anything. Until you’ve tested your DR measures, you can’t have any confidence in them. Incident response and rollbacks can be disruptive in production.

Something else that will help you is to familiarize yourself with the services you use. It can be easy to leave things up to the DevOps team and not really understand how they work—in this case, learning just enough to do basic tasks is a good thing. At least know where to go in the cloud provider to look at configs and then go through the cloud provider docs to get a high-level understanding of what’s being set up.

The cleanup steps in this section should help you with the bulk of what you need to connect and what you should look out for. Every project is different, so add your own steps to this list based on your experience. Many devs have their own checklists that they’ve developed over their careers and that they take with them to new jobs or projects.

## Documentation and Maintenance Steps

At this point, you should update the architecture diagram to its production state because you know how everything works together. [Figure 23-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch23.html#production_architecture_diagram) is an example of what that can look like for this app.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2303.png)

###### Figure 23-3. Production architecture diagram

Now that you have the diagram for production, keep it up to date as things change. You can add as much detail to the diagram as you like, even having lines that represent all the endpoints you have available and including the database schema. After you’ve taken these steps and made your own checklist, document it for the team and see what gaps they find to fill in.

Remember that your app might be in the production environment, but it’s only public once you have users working with it. Until there are actual users, production may not be much different than your other environments. Go through the release process to move the team’s changes from the other environments to production numerous times and involve every dev on the team so that they’re comfortable with it. By the time you have users, it should be a routine thing to deploy to the production environment. You should also test your incident plans while there is minimal traffic to make sure they actually work.

Even though you’ve gone through this process from scratch, as you do this more, you’ll learn new ways that things can go wrong. This is a constant learning process, and you will encounter scenarios that leave you surprised.

# Conclusion

In this chapter, I gave you an overview of some general steps you should take as you get everything ready for production. There are a lot of moving parts after the code is written and tested in your preproduction environments. Whether this is the first time your full stack app is going to production or the hundredth time, be ready for issues. It’s usually smoother after the deployment process has completed several times, but stuff happens. Test out the changes you’ve deployed before you announce to everyone that they’re ready and stay ready to do rollbacks or handle incidents.

The main thing is that you’ve had exposure to the different areas, so you know how they work together and understand the process behind connecting them. At this point, you’ve coordinated across several teams, including the dev team. Now you can clean up the process, document it, and help others on the dev team go through production deploys. The more comfortable everyone gets with production deploys, the more smoothly they’ll go over time.