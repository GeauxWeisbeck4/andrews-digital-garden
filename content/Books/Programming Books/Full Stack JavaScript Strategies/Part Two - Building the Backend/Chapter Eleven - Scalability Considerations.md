---
id: 01JGFP9N30BRKAS8J75WV12MSQ
modified: 2024-12-31T20:04:59-05:00
---
# Chapter 11. Scalability Considerations

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 11th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You’ve covered everything from the architecture down to the endpoint logic and all the way through security and debugging. You’ve made some performance updates to ensure your app is running smoothly, so one of the last touches you can make is getting it ready to scale for more users. That means your backend application is “done.” You’ll add more features, make optimizations, and change things as the product develops and matures, but it’s in a state where the core functionality is stable.

Scaling is something that will start costing your company more money because you’ll need different tiers of third-party services and maybe even more team members. You know it’s time to start considering scaling when your performance enhancements aren’t enough to keep up with usage.

In this chapter, you’ll get more insight on:

- Types of scaling
    
- Scaling strategies
    

Deciding to scale is a huge undertaking because you might need to split existing data or significantly change resources and services. Take a considerable amount of time to research your options and verify what you have available as well as any business limitations you have. The decisions you make for scaling are an important part of how your systems will work going forward because migrations are harder at scale.

# Types of Scaling

There are a couple of different ways you can tackle scaling a backend app. You can scale _horizontally_, which is when you increase the number of instances your app runs on. In an AWS environment, this could mean adding more EC2 instances. Or you can scale _vertically_, which is when you increase the power or size of a resource. That would be like increasing the number of CPU cores or RAM for a single EC2 instance. Let’s look at each of these in more detail.

## Vertical Scaling

Vertical scaling is usually the more straightforward option if your app isn’t already written in a distributed manner because you don’t have to change anything in your code. It’s the same server you already have with better resources. Many companies will go with this type of scaling first because it usually means changing some configurations quickly. It’s also generally a cheaper option because you’re still only paying for one resource.

There are some trade-offs with vertical scaling. Since you can have multiple servers that get vertically scaled, this can increase your cost and possibly your susceptibility to points of failure. If something happens to a server, a part of your application can be down with no other resources for the app to try to run on. If you do run into any downtime, that means you don’t have anywhere else to divert traffic to. With vertical scaling (shown in [Figure 11-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch11.html#what_vertical_scaling_looks_like)), you also have to consider upgrades because they can become more tedious and lead to required downtime.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1101.png)

###### Figure 11-1. What vertical scaling looks like

## Horizontal Scaling

With horizontal scaling (shown in [Figure 11-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch11.html#what_horizontal_scaling_looks_like)), you have more resilience and a higher guarantee that your app won’t experience as much downtime. You’ll usually take the current resource settings for your server and duplicate them across multiple instances. With this approach, you’ll need to configure a load balancer so that you can spread the traffic across the instances. This makes upgrades easier because you can divert traffic as you make the upgrades. It’s usually a more sound solution for commercial products.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1102.png)

###### Figure 11-2. What horizontal scaling looks like

However, it will take longer to implement horizontal scaling compared to vertical scaling. This is where a DevOps or infrastructure team would come in to manage and orchestrate all the resources. This is one of those areas where you get to decide how much depth of knowledge you want to gain. You can learn all about provisioning resources and scaling tools (like [Kubernetes clusters](https://www.vmware.com/topics/glossary/content/kubernetes-cluster.html), [Docker Swarm](https://docs.docker.com/engine/swarm/), or [Rancher](https://www.rancher.com/)), or you can stick to a higher-level understanding, which will still help you converse with the DevOps team.

# LOOK AT YOUR CLOUD PROVIDER’S SCALING OPTIONS

Jeff Graham offers additional tips when it comes to horizontal scaling:

> Depending on your app, most cloud providers have ways to manage horizontal scaling for you. Often called “auto scaling,” these services can monitor traffic or resource usage and add/remove servers (or other resources) as needed. This can save you both time and cost—you won’t need to manage your clusters as often, and it will remove unneeded servers during low-traffic hours (at night, for example). A version of this is usually available for other cloud services too: databases, queues, containers, etc.

# Hybrid Scaling

As you learn more about your needs, you may choose to implement a hybrid of vertical and horizontal scaling. There may be some instances that need more resources because of the region where they’re hosted. Load balancers can help you avoid downtime by distributing the requests across the different server instances you have. If you know this is going to be a highly utilized app, then it’s best to include a load balancer in your initial architecture.

## Resource Scaling

Scaling isn’t just for your API requests. You might need to scale resources to handle the jobs you run in your queues. A sign that you need scaling here is when data isn’t available after the jobs have been running at their normal time. If you check your queues and see that jobs are still waiting, it’s time to start looking at scaling. In some cases, that means your jobs may need to be split out to a separate repo. Then you can run them on a separate server and see how vertical scaling works. Another resource you can try scaling is your database. That will allow you to support higher data throughput or accept more database connections.

You have to consider if you want to scale your resources manually or automatically. Typically, autoscaling is the way to go when you’re working with cloud services. Something to keep in mind is that when you use autoscaling, it’s referred to as _elastic scaling_; that’s when your resources increase and decrease in response to the number of requests or events. Elastic scaling is usually for horizontal scaling, but you might find a rare instance of this with vertical scaling.

This is worth exploring if you know your traffic spikes at certain times of the year. This happens a lot on ecommerce and financial platforms during the holidays because people are shopping more than usual. Then after a few months, the traffic goes back to normal. Or services will experience large spikes in usage after a major event happens. Any website that has gone viral unexpectedly could probably benefit from cloud elasticity more than scaling because the increase is short term. When you’re scaling an app, there’s been long-term consistent usage at this higher level, so you know it’s not temporary.

# Scaling Best Practices

Once you’ve made a decision on how to approach scaling, it’s time to actually do it. By now, you should have done at least a few performance improvements to make sure scaling is really the route you need to take. Here are some steps you can take to scale your apps.

## Make a Plan

I have a template that I’ve created from doing different scaling projects, which you can [find in the project repo](https://github.com/flippedcoder/dashboard-server/blob/main/docs/scaling-strategy.md). This is a good starting point if your company doesn’t have a procedure for how scaling should work. Hopefully, it will help you ask some good questions as you start to make a plan with all the other teams you’ll work with. A big question that comes up early is if you should stay with your current cloud platform or migrate to another. Scaling is something that should be considered before you fully commit to a cloud platform.

The platform you choose usually depends on a combination of the DevOps team’s experience, the tech stack you’ve chosen, how well the platform works with your suite of external services, and the services it has available. When it comes to scaling, you should be able to implement the type you want with ease in your cloud platform. So consider what you have available before migrating becomes a serious option.

###### NOTE

I worked on a project where we had to migrate from Heroku to AWS because of some changes Heroku made. That was honestly a painful process that took about a year and a half and involved multiple teams. In another project, we migrated from GCP to Azure because the parent organization switched over to Microsoft products for everything else. This took almost two years. Think about the big, long-term picture when deciding on the cloud platform because if you need to migrate for any reason, it’s a massive undertaking.

After you know that your cloud platform is what you want, then you can decide on the metrics you want to measure and monitor to determine when to scale your apps; we discussed many of those metrics in [Chapter 10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#backend_performance). You’ll need to decide which teams are involved and responsible for handling and testing the scaled app. There will be a rollout plan and a rollback plan to ensure that all teams are expecting the update, that they know what they can expect, and that there is a way to undo the changes if anything goes wrong. There will also be a test plan to double-check that the app is still functioning correctly and meeting the new performance requirements.

Finally, there will be a cost analysis to make sure the type of scaling you choose isn’t creating a large cloud bill. In almost every case, you’ll go with a horizontally scaled, automated approach, but you may find that a scheduled approach will keep your bill lower or more manageable if you only want to ramp up resources during a specific time.

## Document the Plan

Document the details and decisions for the scaling choice. The question of why this approach was chosen will come up a few times throughout the process, so have the architecture diagram and research readily available. You should update that diagram to show your expected changes as in [Figure 11-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch11.html#current_architecture_with_horizontal_sc), which is a detailed view of [Figure 11-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch11.html#what_horizontal_scaling_looks_like).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1103.png)

###### Figure 11-3. Current architecture with horizontal scaling

As you share the updated architecture diagram, document the questions that come up and the discussions around them. This will help everyone keep track of each decision point as well as help you keep the whole process organized. Don’t be afraid to include comments directly in the diagram because that can give you more context when you come back to discussion points later. Also encourage others on your team to contribute to the docs because that will help distribute the knowledge and give everyone a deeper understanding of what’s happening.

## Run Tests

You’ll usually need to have automated tests ready so that some initial checks can be done with the new resources. The first round of app testing with the new scaling in place should be done by you and the dev team. You have deep product knowledge, and you know how the code is supposed to work. You can find those little bugs that no one else will notice. After you’ve checked everything out programmatically and you see the app is running, double-check the deploy pipeline.

The app should be running on the expected version of Node, and there shouldn’t be any issues with dependencies. It’s also good to check the logs to see if any errors are being thrown from the new resources. Spend some time with the team going through all the endpoints to make sure they work as expected. This is where your unit and e2e tests will really show their value because you won’t have to do as much manual testing. It’s still good to run some manual tests, though, so choose some of those high-traffic endpoints to call first. This kind of testing can take anywhere from a few weeks to a few months depending on the complexity of the system.

When you have your scaled approach in place, it’s important to perform some stress testing to see how your backend app performs as the number of user requests fluctuates. This ensures that you have scaled your resources appropriately and the system doesn’t crash under extreme conditions. It gives you a chance to analyze performance under a great load so that you can determine if your system can handle sudden surges in traffic, see how the system works under unusual conditions, and see how prepared the system and teams are for any unusual conditions.

A few steps are involved with stress testing:

- Planning the test
    
- Creating automation scripts
    
- Executing the scripts
    
- Analyzing the results
    

Making any optimizations and tweaks

Planning the test means that you evaluate how the system currently performs and then define a stress-test goal. This could be something like handling 25,000 requests per minute without crashing, maintaining a three-second response time, and showing appropriate error messages. Your automation scripts will simulate user actions that trigger these requests and generate test data to use in the requests.

Then you’ll run the scripts and check your logs as you gradually increase the load on the system. At this point, you’ll see if you can find any bottlenecks where the system slows down. After all your scripts have been run and you’ve tested the system under peak load, you’ll analyze your results and look for metrics like response times and error rates. Finally, you’ll take those results and fine-tune the system to optimize your scaling settings to give you the best balance of cost and expected performance.

Do small tests as you start shifting toward the new resources. Check that the code still runs as expected. You’d be surprised how a change in resources can make the capitalization in filenames matter. So double-check _everything_.

## Communicate Progress

Communicate what’s happening as you do these small tests. Include other dev teams because they may be doing this one day. Include the Product team so that they understand why feature development is slower than usual during this testing period. Let the Support team know when any testing is being done that affects users so that they can prepare for an influx of messages.

Keep in mind that each team has its own expertise in this process. Your dev team is going to be expected to know the ins and outs of how your code works, the systems it interacts with, and why everything does what it does. The DevOps team is going to take charge of all the infrastructure changes, but they’ll have questions for you. The most useful thing you can do as a senior dev is make yourself available to coordinate with the DevOps team. The work that you’ll do tends to blur with their work, especially on smaller teams.

Depending on the timeline for the scaling effort, you should bring a more mid-level dev into these conversations so that they can learn how things work behind the code. The more people who understand how the project is being scaled, the better. This is another way to spread the responsibility for handling bugs and new features. For example, if the app hasn’t been containerized until now, the DevOps team will need your help figuring out if the app is running like expected.

Be prepared to have a lot of conversations with the Product team and any other business stakeholders, such as the Support team. They won’t need the technical details, but they will need to know timelines and what they should expect as far as performance changes and any downtime. Keep a consistent flow of communication with them so that they know when things are being done and why. This will keep their stress levels down and raise their confidence in what you and your dev team can do.

There will probably be interaction with other dev teams as well because their code might be dependent on your app working. Let them know when changes are expected so that they can plan their feature development around that. If you’re really lucky, you’ll be the first dev team to go through this process. Then you’ll become an expert that other dev teams will call on for help during their scaling processes.

I’ve worked on projects that have gone through this scaling process a few times, and the biggest thing is always communication. Unexpected technical issues will always pop up, but communication is what helps get everyone through them. Let people know when you’ve hit a blocker so that the effort doesn’t get delayed. Also push back when you know you need more time to test things so that you catch as much as possible before switching to production.

## Start Shifting Traffic

Many times, you can work with the DevOps team to trigger automatic scaling so that resources and costs are managed in the most optimal way for user reliability. This is really where the DevOps team should take over and handle everything on the cloud platform. It’s another area where you can decide to deepen your knowledge with the cloud platform or learn just enough to do what you need to do.

# CHOOSE YOUR SCALING APPROACH AS EARLY AS POSSIBLE

I worked on a project once where the decision to implement hybrid scaling was made at the mid-stage in the company. The product was mature enough that the user base was growing quickly and consistently. It took about a year for that effort to finish, and there were so many bumps in that road. Scaling is a huge undertaking that will involve multiple teams across the company, both technical and nontechnical. That’s why it’s better to think through your scaling approach during the initial phases of the project so that you don’t end up in this situation.

You never want to immediately switch all your traffic to freshly scaled resources because there’s a chance something can go wrong. A gradual transition to the new resources is a safe approach.

Have rollback plans in case something doesn’t work out with the current phase of scaling. During this time, it’s not uncommon to have your production server along with your new server resources. This way, you can ensure that users don’t experience much downtime as the changes are made.

Slowly shift traffic over to the new resources and see what happens. Stay vigilant as you watch the logs and look for any new errors or messages. It may help to have a separate alert for this new traffic so that you quickly know which part of production to check. It also helps to do this during nonpeak hours, so if something happens, it affects the least number of users.

Once you’ve completed these steps, you’re ready to go over everything with all the teams affected by the changes just to make sure it’s all working as expected.

# Conclusion

In this chapter, you learned some ways to extend your performance enhancements into a deeper part of your architecture. This is a complex process no matter which scaling approach you choose, and it’ll practically cement you into your current cloud platform. That’s not a bad thing, but it is something to be aware of, especially if you’re using platforms other than AWS, GCP, or Azure. I’ve worked on migrating across cloud platforms, and it’s not the most fun process.

Keeping communication flowing is essential. That doesn’t mean you’re responsible for overseeing this whole effort, though. It just means you do your part to make sure your dev team isn’t blocking anyone or being blocked. Do what you can to document changes and decisions, and be available to help out when you need to. Now you can set your attention on everything with the frontend.