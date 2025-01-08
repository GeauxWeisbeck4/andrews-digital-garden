---
id: 01JGFP312QGG0MCFS51T3CTNJ7
modified: 2024-12-31T20:01:22-05:00
---
# Chapter 6. Background Jobs

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 6th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Once you have some third-party integrations, it’s likely that you’ll end up needing automated ways to keep the company’s data in sync with the third party’s data and to trigger other events like sending emails. That’s where background jobs and cron jobs come in.

A _background job_ executes code or actions that need to run outside the flow of an API in a systematic way. This will include tasks like sending emails or doing complex calculations after a user has made an update. Background jobs are going to work alongside your server-side application code and are typically triggered directly from an endpoint call. These jobs are usually defined at the service level because they typically have business logic around them.

A _cron job_ is a task that is executed on a schedule. It will run at some time interval you set based on the business needs. Syncing data from or to a third-party service on a schedule is a prime example of using a cron job. Cron jobs can be in your application, but it’s more common and better practice to have them handled somewhere else, like an Amazon Elastic Compute Cloud (EC2) instance or just another server. That way, you won’t use up resources that your application is running with.

One of the hardest things about working with background jobs or cron jobs is that, since they run on their own schedules, they can affect the state of the whole system in unexpected ways. That’s where your senior dev skills come in. In this chapter, you’ll dive into:

- Incorporating background and cron jobs into your app architecture
    
- Setting up alerts, monitoring, and logging for background and cron jobs
    
- Handling data-sync issues with third-party services and cron jobs
    
- Handling task-execution issues with background jobs
    
- Future-proofing your background and cron jobs
    

The first thing to note is that your jobs can be written in any programming language. So you aren’t doing anything drastically different, you’re still going to use TypeScript. This is just code that gets run somewhere aside from your endpoints or service logic. This is why understanding the overall architecture of the app is important because jobs can get lost over time as developers come and go and the product changes.

You’ll be expected to be able to make and document those architecture changes and explain how they will affect resource allocation. You may even be responsible for the initial research around the tools that work best with your infrastructure. This is where it’s great to talk to the DevOps team or whoever manages the infrastructure.

Find out what cloud providers are used to handle all the apps for the company. This will give you an idea of the tools you have available and how you should approach writing the code for your jobs. The tool you use will have a direct impact on your implementation because you have to use its SDK to interface from its resources to your code. You’ll learn more about this as we go through this chapter, but let’s start with the architecture before going too deep.

# Updating the Backend Architecture

For this project, you’ll implement a background job to send emails and a cron job to sync data from Stripe to your database. To make sure you and everyone else understands where these fit into the backend, you need to update the architecture diagram. Whenever there is a question about how things go together and why resources are used a certain way, the architecture diagram should be the source of truth. It’s essential for you to make sure this document is maintained. This is also a good place to see if there are any initial concerns with how a new feature will get integrated into the backend. [Figure 6-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch06.html#updated_architecture_diagram_with_backg) shows what the diagram will look like now.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0601.png)

###### Figure 6-1. Updated architecture diagram with background and cron jobs

###### NOTE

Updating docs is an incredible opportunity for you to really see if you know what you’re talking about. An important skill for you to have is being able to mentor more junior devs; this is so important that it can’t be overstated. Take architecture updates as a way to introduce new concepts to the team and make sure everyone, including you, remembers how this whole product works. You might even run into some areas you need a refresher on!

Now you have a visual representation of the flow of the jobs. The background job to send emails will get triggered only when something happens in the Stripe integration. The cron job to sync Stripe data, like products, with your database will happen on a schedule.

# A DIFFERENT OPINION ON STRUCTURING YOUR CRON AND BACKGROUND JOBS WITH THE APP CODE

Ethan Brown has some great advice about this. What follows is from Ethan:

> Increasingly, I’m preferring monorepos, and on most of my projects, I do colocate cron/background code with my application code. It’s organized in appropriate folders, of course, but it’s not in a separate repo or project, and it’s certainly not a microservice. The way I like to think about it is that it’s all application code.
> 
> Some of it is invoked as a direct response to user requests, some of it is invoked on a schedule (cron jobs), and some of it is invoked directly or indirectly from a user action, but long-running enough to have a push mechanism for informing interested parties that the job had completed (background jobs).
> 
> In general, I think this approach offers a powerful paradigm that can demystify cron/background jobs. Think about it all as part of the problem domain (i.e., application code), which is primarily event driven: the bulk of the events may be direct user requests (HTTP, XHR, WebSockets, usually), but some events are driven by schedules and some are driven by queues (Amazon’s SQS [Simple Queue Service] or SNS [Simple Notification Service], in my case).
> 
> I’d be more inclined to make an exception to this paradigm for read-only background jobs like analytics or reporting. If this job isn’t changing system state, just monitoring or doing business intelligence work, that can be a microservice or separate app.
> 
> But if a job changes system state, it is application code, regardless of how it was invoked.
> 
> I extend philosophy to system diagrams: they show not only the application endpoints but what kind of events trigger them.

Since you have an idea of how these jobs will work within the existing architecture, it’s time to take a glance at the infrastructure.

# Using Cron Jobs

This is where things get really specific, so make sure you have input from the DevOps team. For this project, we’ll assume that your infrastructure is built on Amazon Web Services (AWS). You won’t get a deep dive into the services in this book other than some quick implementation. If you want to learn more about a specific cloud provider, I suggest starting with their docs, blog posts, and free training materials.

Cron job scheduling is often handled through the cloud configurations, which you usually won’t touch. You’ll be responsible for the code that runs the job, though, and you should know the schedule it runs on. The example cron job you’ll implement to sync the Stripe data will be in the repo with the rest of the project for now. This is a good way to test that it works as expected before you start updating the infrastructure.

Just to see what a cron job can look like, here’s a code snippet from the project you’re working on ([you can find the complete file on GitHub](https://github.com/flippedcoder/dashboard-server/blob/main/src/jobs/crons/data-sync.cron.ts)):

```
…
```

Let’s break down some of the key things. The first is the cron expression `‘5 00 * * *’` that defines the schedule for your sync job. You can play with different values on this [cron expression site](https://crontab.guru/#5_00_*_*_*). The expression is set to run the code at five minutes after midnight every night. That way, your data syncs when you can get the most info with the least impact on your users. Keep time zones in mind here because that can have a surprising impact on your system. If you have the code running on a server in UTC-5 and the user lives in UTC-8, that can cause some issues with data syncing on time. You can see the [full code in the repo](https://github.com/flippedcoder/dashboard-server/blob/main/src/jobs/crons/data-sync.cron.ts).

# MORE TO NOTE ON YOUR CRON JOBS

Jeff Graham has some additional insights on what to watch for with your cron jobs:

> It’s common that cron jobs encounter issues as your traffic or usage increases. You might find it takes longer and longer for a job to complete, which can lead to timeouts, long queries, and failures. Worst case, your job can’t complete its list of tasks, but restarts at the beginning over and over throughout the day. Disaster!
> 
> Monitoring your jobs to prevent these situations is crucial. This way, you will see the problem coming, and you can implement changes ahead of time. All jobs are inherently different: some may run quietly in their original form, while others will need improvements and maturation.

## Alerts and Monitoring

There’s another almost-hidden thing happening with this cron job. As any errors are being logged, certain ones may be more urgent than others. That’s where your monitoring tools will come in. This is probably something that the DevOps team will handle because they are responsible for resource management and infrastructure-level incidents. When programmatic issues come up, though, it’s on your dev team to solve them and figure out the root cause.

You’re watching for things like a number of errors being thrown in a specified time range, a specific type of error that occurs, or something else you and the DevOps team have agreed on. Monitoring can be used to trigger alerts, which help the team address issues as soon as they come up. Monitoring usually involves agreed-upon metrics, and it can also help establish baselines for how the app should perform.

Most of the time, alerts will be configured in the infrastructure. If you have a job that starts to trigger a lot unexpectedly, the DevOps team may have an alert that lets them know when a certain resource-utilization level is reached. Or if you start seeing a lot of 403 errors in your monitoring tool, that could indicate an attack on the system. There are also alerts that the dev team will need, such as code continuously throwing errors. You’ll go into more depth on monitoring and alerts in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide).

Between logging, monitoring, and alerts, you should be able to start triaging an issue as soon as it comes up and have the info you need to find a root cause.

## Logging

The next important thing to note is the logging that’s in place. When jobs are running automatically, they will have errors from time to time. Debugging scheduled jobs can be hard for a number of reasons. Issues like the service throwing errors or being down, records getting skipped, errors happening only at a certain time of day, or database issues happen.

Sometimes you can’t replicate the errors because they occur under very specific conditions. That’s where going through logs comes in; we’ll get to that in [Chapter 9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#backend_debugging). There are a number of logging lines in here because these are the places in the job where errors may happen and cause other issues. As you think about what to log, think about the info you might need to debug errors. IDs, statuses, names, and the like are all things you can look up across different parts of the system.

Something else that can help is to put specific searchable terms in your logs. As the product grows, a ton of logs will be generated every time an endpoint is called. So being able to quickly filter the logs down to a specific event is essential for concise debugging.

## Issues with Data Sync and Task Execution

Once you have a _root cause_, or the source of the error in the code or environment configs, and you’ve figured out the fix, you need a way to retrigger the job that failed. It’s good backend practice to have a retry mechanism for any cron or background jobs. When you run into data-sync issues, usually retrying to fetch the records will fix them. In your retry function, you might do a quick count to see how many records were updated so that you know the impact of the issue before fixing it.

I’m not going to walk through the code for the background job that will send emails, but [you can find it on GitHub](https://github.com/flippedcoder/dashboard-server/blob/main/src/jobs/background/emails.service.ts). For jobs that are time sensitive and communicate important info to the customers, your alerts will be helpful for the dev team. This can involve an automated response to the customers or the dev team that also lets the Support team know something has happened. You may even get to the point where your monitoring metrics can be used to trigger retry events automatically. Most of the time, you’ll work closely with the DevOps team to debug issues with jobs because of the way they integrate with the infrastructure.

## Future Considerations

More errors will occur as the number of jobs and the amount of data they handle increase with the number of users. You’re at a great point in the project to build a stop mechanism for your jobs. Sometimes jobs just execute unexpectedly, and you need a way to stop them quickly. You can have stop mechanisms for each job or for specific jobs that are more impactful on the overall system. If you start by building this functionality into your first job, when it inevitably gets copy-pasted to create a new job, it’ll already have this handling baked in.

When you create jobs, there’s a chance that the code won’t get updated as often as it should as long as the jobs run successfully. To prevent the code for your jobs from becoming legacy code, always have a ticket somewhere in your backlog to update the packages and do refactors for the jobs. Doing this at least once a quarter should be often enough to keep the code maintainable.

# Conclusion

In this chapter, you were able to write a cron job to handle syncing your Stripe data to the company database and a background job to handle sending emails to users for orders. This is where having a solid understanding of the architecture becomes important so that functionality doesn’t get lost or forgotten about. You also can see how much the dev team and the DevOps team work together.

You don’t have to be a DevOps engineer to get a basic understanding of how the infrastructure works and how to use the tools that have been set up. You may feel like you should know how to do everything. You don’t. Learn how to lean on other teams for their expertise to help your team get things done more efficiently.