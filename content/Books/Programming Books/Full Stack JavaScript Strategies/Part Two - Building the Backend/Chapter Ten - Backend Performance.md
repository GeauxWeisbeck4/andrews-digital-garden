---
id: 01JGFP8PM305A624BZYE9C5Z98
modified: 2024-12-31T20:04:28-05:00
---
# Chapter 10. Backend Performance

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 10th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Now that you’ve got an app that’s ready for production, it’s time to look at some performance improvements you can make. This is where you start looking at metrics and tweaking your infrastructure and configs. Making changes to the code can also improve your performance, so never underestimate a good refactor.

How you handle performance updates will vary greatly depending on your app. Deciding where and how to make performance improvements will take insight into how the app works, what’s currently happening, and how it’s expected to grow. Once you get these data-driven answers, you can start looking at trade-offs among the options you have. I’ll go over a few things you might research as your app is used by more people.

In this chapter, you’ll learn about:

- Metrics
    
- Alerts and monitoring
    
- Caching
    
- Product considerations
    
- Other ways to speed up performance
    

Taking care of performance enhancements can be an exciting time because it means the product is being used. This is one time when having meetings can be helpful because this is going to involve costs to your organization. Stakeholders see the product from a different perspective, and that can help you make decisions about the best approach.

# Metrics

To determine if you need to look at performance enhancements, figure out what brought up the topic in the first place. Were users reporting slower load times, or was the server crashing under a certain load? The enhancements can also be a preventive measure. If you know you’re releasing an app that users have been waiting for, your servers can hit a utilization level you didn’t think was possible.

That’s why you turn to the data if you’re unsure or you need to explain the reason for new engineering costs to someone. Look through your logs to see if there are any general trends. You might notice one endpoint getting a significantly higher number of requests than the others. You could see that a particular set of endpoints are used the most during a certain time of day. This type of business analysis can guide what enhancements will be most beneficial.

Some backend metrics you’ll see include:

- The application’s average latency (amount of time the app needs to respond to a request on average, excluding processing time)
    
- Standard deviation of the latency for responses
    
- Minimum and maximum latency
    
- P90 latency (the slowest response in the 90th percentile of the fastest requests)
    
- Number of requests per second
    
- Data I/O ratio (the data input metric is the size of the request payloads coming in, and the data output metric is the size of the response payloads going out)
    
- Peak response time (PRT; finding the longest response times out of all requests coming to the server)
    
- Hardware utilization, such as RAM and disk space usage
    
- Number of threads being used across all requests (determines how many concurrent requests are happening at the same time)
    
- Server load (measures the average number of threads currently running or waiting for CPU time during a specified time range)
    
- Server uptime rate (percentage of time that a server is available to use)
    
- HTTP server error rate (how often certain errors occur)
    
- Apdex (an open standard for aggregating user satisfaction into a weighted average score)
    

# MORE INSIGHTS INTO PERFORMANCE METRICS

The metrics we discussed in this section have a lot under the hood. So I got some input from Ethan Brown.

Regarding the P90 latency:

> The P90 latency is the value that partitions your requests into the 90% that are at or above a specific latency threshold and the 10% that are slower than that. For example, if the P90 latency is 100 ms, 90% of your requests are completing in under 100 ms, and 10% are completing more slowly.

Regarding PRT:

> “Latency” is explicitly referring to transport latency (excluding processing time), and peak response time includes processing time. Max latency and PRT are not the same.

Regarding number of threads:

> Node is, by default, a single-threaded server. Unless you explicitly build it to use worker threads, it’ll be single-threaded. A server configuration may have multiple Node processes running and do some kind of load balancing, but the usual configuration is scaling instances out, not up. Of course, a single-threaded Node process can handle concurrent requests by using an event loop, but that won’t show up in OS threading statistics.

Depending on the focus of the business, there will likely be a subset of these metrics that you’ll consider. You’ll need to be comfortable using multiple services and finding this type of information in your cloud platform dashboards. These [Splunk dashboard examples](https://logit.io/blog/post/splunk-dashboard-examples/) are good examples of what you might look for as you go through your product. They highlight specific metrics that matter most to the users, such as severity errors and network performance. You are doing something similar for your product as you decide which metrics are important. [Figure 10-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#gcp_metrics) and [Figure 10-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#api_requests_per_second_in_gcp) are examples of some metrics you may find on a GCP dashboard.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1001.png)

###### Figure 10-1. GCP metrics

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1002.png)

###### Figure 10-2. API requests per second in GCP

###### NOTE

Since these metrics have the most meaning from a production app, it will help you to actually see the numbers in tools being used at your organization or on any project you’re working on. If you can make time this week, look through all the server tools you use. If you don’t know where to find things or what different metrics are being measured, make sure to ask someone. Get comfortable digging into the areas you might be less familiar with. You don’t have to become an expert, but you do need to know enough to be effective.

A couple of other optimization improvements you might consider include algorithmic performance and database query optimization. [_Algorithmic performance_](https://klu.ai/glossary/algorithmic-efficiency) is usually best suited to apps that are computation intensive, such as data processing apps or apps that handle real-time calculations like in video games. Then you’ll consider metrics like time complexity and space complexity. [_Database query optimization_](https://www.geeksforgeeks.org/best-practices-for-sql-query-optimizations/) is when you refine SQL queries for performance and efficiency in your database. That can include decreasing read or write times or finding ways to target specific data and exclude what you don’t need.

I’m not going to go into detail on these last two because they can lead to prematurely optimizing your app. As you start to see patterns and trends over time, keep everyone up to date. That way, you can prevent performance issues before they become a big thing. Tracking metrics will give you other insights you can use to make operations run more smoothly.

# Alerts and Monitoring

Since you’ll already be measuring your metrics, you might as well add some automation in the form of monitoring and alerts. Monitoring will help you find when your metrics reach a certain threshold, and alerts will notify you when those thresholds are exceeded. By setting thresholds for your metrics, you can see how often they get close to the max, which will tell you which parts of the app need adjustments.

By monitoring your metrics over the long term, you’ll see trends that happen throughout the months and years. This can help you make decisions that can save the organization a lot of money on cloud expenses. You’ll be able to see what times of the year require more resources than others, and you can scale up and down accordingly.

###### NOTE

Something cool you can do for your team is to have a quarterly or semiannual meeting just to talk about these metrics. It’s a good way to expose more junior developers to things that happen behind the code and give everyone an idea of where technical improvements can be made. You might even include the Product team in this meeting so that they can see where users are interacting with the app the most from a technical side.

Something else you can do with monitoring is trigger events when a metric is within a certain range. You can always use the monitoring and alerts services that come with your cloud platform, such as [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview), and [GCP Cloud Monitoring](https://cloud.google.com/monitoring). [Zapier](https://zapier.com/), [n8n](https://n8n.io/), and [Make](https://www.make.com/en) are other widely used services that integrate with a number of tools. They can help you trigger complex events automatically when your metrics reach a limit. For example, you could have an automation that scales up the resources, sends an email and Slack notification to the dev team, updates a message displayed on the website, and updates a value in a database. No matter what your automation flow looks like, you need to know what’s happening.

Alerts are going to help you in a few ways. The biggest is that in production, you can find out about issues as soon as they happen with messages in specific chats or channels with relevant team members. This gets into the practice of [ChatOps](https://www.atlassian.com/blog/software-teams/what-is-chatops-adoption-guide), where you use tools to help automate communication between resources and people ([Figure 10-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#bot_integration_into_slack)). Alerts help spread the responsibility and awareness of issues across the team. One thing you should stay on the lookout for is how you can enable the team to do more. Make sure that everyone knows how the automations work and what metrics are being monitored.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1003.png)

###### Figure 10-3. Bot integration into Slack

Once you understand where the metrics are and you have some monitoring and alerts set up, write some docs and do a quick call with the team to explain everything. This is a quick way you can level up your team as you learn new things. Your team will also help you level up with the questions they ask. Questions are always good for you because they help you figure out where you can deepen your knowledge.

There’s another art involved with alerts. Make sure you send only the most important ones, or else they all tend to get ignored. You’ll have to make updates to keep alerts relevant as metrics change. Set aside some time to go through the current alerts, see which ones are happening most often, and make updates accordingly. Also consider having alerts delivered to different places depending on the severity. You might have production alerts sent to a Slack channel and emailed to the team. Other alerts might go to a different Slack channel or skip the email part. You’ll learn about all this in detail in [Chapter 12](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#monitoringcomma_loggingcomma_and_incide).

Once you have all of this in place and it’s been through a few weeks of testing, you can start trying out more performance enhancements.

# Caching

A few metrics are likely to get monitored at every organization, such as latency. You usually want this number to remain low so that users have the fastest experience possible. Server-side or database caching is one of the enhancements you can add to help with this. Database caching usually involves having a storage location to retain responses from the servers it sits in front of, which reduces the latency by reducing the number of database requests.

Load is another metric you might measure, but it’s more complex than latency. If you’re using a serverless technology, the load metric probably doesn’t matter, or there may be different metrics you would care about. If you’re provisioning your own servers to handle your aggregate load, the objective is to find a balance. You don’t want load to be routinely low because that would mean you’re paying for resources you’re not using, and you don’t want your servers at high load all the time. So you have to reserve capacity for spikes in usage, which can be difficult to anticipate.

When you use a server-side cache, the server will make a request to the cache first, if possible. The cache contains the responses for the most requested endpoints. This set of endpoints will usually be determined by your monitoring efforts. The cache gets the response values either at a set time interval or when a request is made for the first time within a certain time period. That way, users are getting up-to-date information and not stale data.

If the request data is found in the cache, then the server will never hit the database, which saves time on sending the response. That’s how caches are used to improve the server performance. They make sure the database server only gets called when it’s necessary and not every time a request is made, as shown in [Figure 10-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#cache_hit). This type of caching is also called _database caching_ because it reduces the number of queries made.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1004.png)

###### Figure 10-4. Cache hit

###### NOTE

When you are using a cache, it’s very important that you don’t store PII in it. This is a security vulnerability because there’s always a chance someone can attack the cache instead of the server if they figure out what is being used. [Cache poisoning](https://owasp.org/www-community/attacks/Cache_Poisoning) is a vulnerability attackers can take advantage of by forcing malicious content into the cache that is referenced by users. Users will continue receiving the malicious content until the cache is purged.

Sometimes you will have requested data that isn’t in the cache, or the responses in the cache become stale and get removed. When that happens, it’s usually a cache miss, like in [Figure 10-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#cache_miss). That means the request goes through the cache and hits the database. The server response gets stored on the cache and then sent to the browser.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1005.png)

###### Figure 10-5. Cache miss

## Cache Strategies

You can use several cache strategies to maintain the balance of fresh data with fast responses. Some of these strategies include read-through, write-through, write-back, and cache-aside.

With _read-through_ (shown in [Figure 10-6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#read_through_cache_strategy)), the cache is always checked first for data. If the data isn’t available on the cache, the request is made to the backend to fetch that data from the database. It’s then stored on the cache. There isn’t a mechanism to update the data in the cache, so this strategy is best used in situations where the database isn’t updated often or the requests are really slow. This doesn’t affect how update or create requests are handled. They still go straight to the server and database.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1006.png)

###### Figure 10-6. Read-through cache strategy

The _write-through_ strategy (shown in [Figure 10-7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#write_through_cache_strategy)) lets you write data to the cache and the database from the requests. This ensures that you always have the most up-to-date data, but it can slow the initial write operation because it’s waiting for the database write operation to finish. You’ll usually see this strategy implemented along with the read-through strategy to get the best performance.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1007.png)

###### Figure 10-7. Write-through cache strategy

You can also use the _write-back_ or _write-behind_ strategy, like in [Figure 10-8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#write_back_or_write_behind_strategy). This is similar to the write-through strategy except you make multiple writes to the cache before the database gets updated. This lets you process user input a lot faster, and you can handle database updates in batches, although you have to be careful to avoid massive data inconsistencies or data loss in case something happens to the cache before the database is updated. Write-back is useful for high write performance when you don’t need data immediately in your database, like with image or audio processing.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1008.png)

###### Figure 10-8. Write-back or write-behind strategy

A final strategy is the _cache-aside_ approach, shown in [Figure 10-9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#nine_cache_aside_strategy). In this case, the application has to manage the cache. When data is requested, the application will check the cache first. If the data isn’t in the cache, then you’ll get it from the database and store it in the cache. This approach is a pretty well-rounded one. The main concern here is keeping the cache data up to date. This is a strategy to consider when your resource demand is unpredictable so that you can load data on demand. You can also use this strategy when the cache doesn’t have read-through and write-through operations.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1009.png)

###### Figure 10-9. 9 Cache-aside strategy

## Types of Caching

There are different types of caching, such as caching you build yourself and caching tools; both are typically in-memory. Caching in-memory stores data on the server’s RAM. This approach can be useful when you have a large amount of data that rarely changes, such as a product list. This product list might get loaded the first time the user visits the site and then is cached in-memory. Implementing in-memory caching is usually fast, and it provides quick response times. One of the biggest downsides to this type of cache is that the data is gone whenever the system is restarted or the RAM is cleared.

Some common caching tools include [Redis](https://redis.io/) and [Memcached](https://memcached.org/). With a caching tool, you don’t have to worry about implementation details. If you’re working on an app that has users around the world, this type of cache makes sense. You can have cache servers in different regions so that they can get fast response times with a distributed cache system.

Having distributed cache servers with these tools is a cost-effective way to make data available all over the world. Some drawbacks are that data consistency can become a problem and the system can become complex to manage. When there are data-consistency issues, some users might see a different set of data than others. Once the distributed system is large, it can be hard to trace issues and manage configs.

There’s also client-side caching, but I’ll come back to that in [Chapter 20](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#frontend_performance). Once you’ve figured out how to add caching to your architecture, take a break and step back to look at it from a different angle. [Figure 10-10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#architecture_diagram_with_cache) shows an example of how you can add Redis caching to your architecture.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1010.png)

###### Figure 10-10. Architecture diagram with cache

## Cache Implementation with Redis

Just so you have an idea of how the code would look to work with Redis, here’s an example with the products controller in the project. First, you’ll need to configure the app to use caching with your Redis server. You can experiment with a [Redis instance in Docker](https://www.docker.com/blog/how-to-use-the-redis-docker-official-image/). Once you have your Redis credentials, you’ll need to install a few packages and then update the `imports` in your _app.module.ts_ file:

```
// app.module.ts
```

The `CacheModule` has all the values you need to connect to your Redis instance. You can check out [the full code in this file](https://github.com/flippedcoder/dashboard-server/blob/main/src/app.module.ts#L20) to get the details; for example, `redisStore` is an imported package. All the environment variables are values that you will get from your Redis dashboard.

Now you have to update your endpoints to actually use this cache. Let’s look at the products controller and see how you can cache the response for fetching all the products:

```
// products.controller.ts
```

This involves some imports, so I encourage you to look at [the full code in this file](https://github.com/flippedcoder/dashboard-server/blob/main/src/products/products.controller.ts#L21) so that you have all the context. There are three big things to note here regardless of the framework you use:

- The response from this endpoint is automatically being cached. That’s something you want to happen on the busy endpoints so that the user experiences those speed improvements.
    
- Define the cache key. Your cache stores data in key-value pairs, so this cache key tells you where to look for the data.
    
- The cache time-to-live (TTL) value dictates how long the value will remain on the cache before it’s declared stale.
    

You can do more advanced things with your cache, like storing responses from third-party services you use to speed up your responses. No matter what your framework is or even if you write your backend in straight Node, as long as you have a way to implement the three things I just mentioned, you can implement an efficient caching strategy.

Now that you’ve implemented your first attempt at caching, it’s time to take a look at your performance enhancements from a different perspective.

# Product Considerations

It’s easy to get caught up in the technical details of your enhancements to the point where you lose sight of what the original problem was. At every step of your enhancement implementation, stop to ask how it affects the product you’re building. Does it make things faster and easier for a user, or is there a level of trade-off that needs to be considered? While your caching strategy will decrease latency, how confident are you with identifying and debugging cache issues? Is this something that needs to be considered for team velocity because it needs to be maintained? These are a few questions that may come up as you think about performance improvements from the product side.

###### WARNING

You also want to consider the image of your dev team and the engineering department as a whole within the organization. This is especially true when it comes to monitoring and alerts. Be careful with what you agree is important to report on because it will influence how other teams view you. It shouldn’t be this way because of the dynamic nature of software, but how the team is perceived does make a difference in the level of autonomy and trust you have.

Think about performance from a user’s perspective and actually test it out. You might try throttling the response rate to compare the difference in latency before and after caching to make sure it has the impact you wanted. Also think about how this can grow in the future. The service you decide to use for caching will help you determine future costs and maintenance.

You should also consider other ways to speed up your app. Maybe you can increase the RAM on the server to handle more requests or bigger loads, or you can increase the number of concurrent requests allowed programmatically. Remember, there’s always a trade-off with the time it takes to implement performance enhancements compared to implementing new features. You can start with a quick approach to test a hypothesis and then add some tickets to research and plan a long-term solution. Some things you might consider to determine if an optimization is worth the time include preventing customer churn due to slow responses or comparisons to competitors.

# Conclusion

In this chapter, you learned how to find areas for performance improvements. Turn to the metrics you’re going to use to measure performance so that you can figure out what improvement means. Look at your monitoring to see where your app consistently crosses the threshold for your metrics.

Once you know which endpoints need to be addressed from your metrics, you can decide how to make your improvements. This might be implementing a caching strategy, increasing resources, or doing something differently in the code. Many of the other strategies you can use start to get into scaling, which we’ll cover in the next chapter.