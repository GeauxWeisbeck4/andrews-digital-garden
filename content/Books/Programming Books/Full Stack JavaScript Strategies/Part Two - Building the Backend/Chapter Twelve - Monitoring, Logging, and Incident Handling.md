---
id: 01JGFPAMBF1AGHGGVRHMWKQ7NP
modified: 2024-12-31T20:05:31-05:00
---
# Chapter 12. Monitoring, Logging, and Incident Handling

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 12th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Now that you have built your backend, it’s time to take a deeper look into your monitoring and logging tools. This is where you’ll get the info you need to figure out what’s happening in your app across different environments and decide how to handle it.

You’ve set up logging throughout your code, so you already understand what to log and have plenty to test and check for. The goal now is to get meaningful insights from what you’re already logging, such as figuring out when incidents are occurring and resolving them.

In this chapter, I’ll go over:

- Multiple uses for logs and monitoring data
    
- How different tools work
    
- Incident plans for production based on logs and monitoring
    

Monitoring and logging aren’t there just for when something goes wrong. They tell you the story about what is going on across your full stack app and even on the infrastructure level. If you know exactly what happens with the app and when it happens, then you can find out why it happens. This gives you a chance to exercise some creativity and take part in the direction the product goes.

# Uses for Logs and Monitoring

Let’s start by defining what logging and monitoring are. If you remember in [Chapter 9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#backend_debugging), we talked about logging for debugging purposes. _Logs_ are messages that are sent as events happen in your app. These are typically created when an action is triggered so that you have a record of the events that happen. Your logs can initially be sent to a text file somewhere in your system, but you’ll move on to more robust logging tools so that you can easily search for messages associated with particular events.

This is where monitoring comes in. _Monitoring_ gives you real-time insight into key events and metrics so that you notice errors, unusual activity, and downtime immediately. Monitoring can make your logs more actionable because it will highlight the activities and messages that are the most important. This is crucial for your apps that are in production and actively being used by customers. It’s one of the first ways you can determine an incident is occurring. Before we discuss incident handling, let’s go over some of the practical uses of logging and monitoring.

## Use Logs for Debugging

As you know, debugging is an important use of logs. Whenever you get a bug or you’re looking to see if a job has run, see if you can find it in your log messages. A quick way to figure out where in the code an error is coming from is to search for the log message in the repo. That will take you to the exact line of code where the message is being sent from, and you can start your debugging efforts from there. Take a look at the log messages to see if there are any other errors that occur higher in the [call stack](https://developer.mozilla.org/en-US/docs/Glossary/Call_stack). That’s another way to quickly get to a root cause if you aren’t finding anything in the first file you check.

You may find out that you have bugs in production that you can’t replicate in staging or locally. Or you may find bugs in staging that don’t occur in production. That’s when you can debug the differences in your environments to see if there are resource issues or if it’s something like a limitation of the test accounts you have for third-party services or in your database.

Sometimes, your logs will show that errors consistently happen in staging, and you need to address them just to keep the messages clear for real errors. Other times, your logs can show you warnings for your code that don’t seem to cause issues. Always take a look at the warnings regardless of whether they cause any side effects. Just because the app still runs with a variable being undefined doesn’t mean that this is an issue you can ignore. A warning can still lead you to a root cause for an error, but be selective about the warnings and errors you report. Sometimes they create noise you have to sort through, which can lead to everything being ignored, even the important errors.

Another useful thing is tracking bugs through the full stack of log messages. The frontend will often give you a good starting point for where issues are happening because the app will stop functioning there. With your logs, you can look for any backend errors that happened at the same time as the frontend error. That could lead you to the source faster than if you dig through the frontend code. This is one approach that can save you time when the true issue is something with the backend anyway.

Your logs should also be the first place you check when you notice any discrepancies with jobs running. Data sync issues can be tricky to track down because they are dependent on so many layers. By going to the logs, you can find out if your next step should be checking the database for inconsistencies or if there’s something in your queue that isn’t being executed because there are errors with the infrastructure. You can also use your logs to cross-check if any issues from third-party data coincide with downtime from those services.

Another cool thing you can do with logs is add more as you debug. You might find that you track down a bug to a certain point but you can’t see it unless you deploy to staging. You can add more logs and open small PRs for them that give you more insight into what happened before or after the original log message in your code. It’s not uncommon to wrap data-fetching functions with logs that capture parameters that were sent and the data that gets returned. Throw in as many log messages as you need to figure out what’s happening. You could also use different log levels like we discussed in [Chapter 9](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch09.html#backend_debugging), coupled with environment variables to turn log messages on and off.

###### TIP

Something I mentioned in an earlier chapter is being mindful of how you pass data in log messages. The main reason why is because seeing `Object [object]` in your messages isn’t helpful. If you do run into this, modify your log message to use `JSON.stringify` or something that extracts the data out of the object because these messages are going to be converted to strings every time.

Logs can be used for more than debugging because they give you actual, real-time insights into how users interact with your app. They also show where you might need to invest in your staging-environment resources or limits you’re running into with your third-party services. It can take time to get the most use out of your logs because it’s a lot of data to sort through if you don’t have a specific goal. Logs can answer a lot of questions, so this is your chance to exercise real technical creativity and come up with interesting questions to find answers to.

## Use Monitoring to Inform Your Actions

While the logs will give you plenty of information, monitoring gives you aggregated info that you can act on immediately. Monitoring lets you set up metrics and thresholds for how your app behaves in production or any other environment. You’ll have metrics regarding how often endpoints are called, when certain errors are triggered, and other activities that you and the different teams decide are important. These are usually coupled with alerts so that you and the team are aware when unusual activity is happening in the app.

Monitoring gives you actionable information from all of your logs. You and the infrastructure team can take actions based on the metrics you get over time. Monitoring happens in your environment in real time, so you can be aware of when resources need to be scaled up to accommodate more traffic. You can also quickly determine if someone is trying to perform a [DDOS attack](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/) on your app based on the number of calls and frequency of those calls being made to the backend. Since you know the errors you may encounter, you should set up a metric for the number of errors the app runs into so that you and the dev team can better address them.

The metrics you have from [Chapter 10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#backend_performance) and upcoming in [Chapter 20](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch20.html#frontend_performance) can give you other important app data to monitor. These metrics track when your page loading time changes, how long API responses take, and your cumulative layout shift. With monitoring, you’ll get graphs and statistics based on these metrics, and you’ll have a threshold set for all of them. If that threshold is passed, then alerts will be triggered so that the teams can figure out what the issue is and resolve it. Alerts help make sure that a team will be responsible for addressing issues.

Monitoring and alerts also help make incident response more manageable because the team gets a notification when thresholds are passed. This might seem like something that should be in place by default, but I’ve worked in places where they didn’t set up any monitoring or alerts until the app had suffered DDOS attacks a few times. So whenever we were on call, we had to manually dig through the logs to figure out which endpoints were getting flooded with traffic. That made the process much more stressful compared to after we had monitoring in place. Then when issues came up, the root cause was clearly presented to us, and it was a matter of either working with the infrastructure team or making a quick code update.

## Monitoring and Logging Tools

Now let’s take a look at some popular logging and monitoring tools so that you have an idea of how they work. I’ll go through some logs and show monitoring examples in [Datadog](https://docs.datadoghq.com/logs/log_collection/javascript/) because it’s one of the more commonly used tools I’ve seen across organizations. [Sentry](https://docs.sentry.io/platforms/javascript/guides/react/) and [LogRocket](https://logrocket.com/) are also great products that are widely used, and I’ve worked with these as well. I’ll implement logging and monitoring on the backend so that you can see how Datadog works, what its dashboards look like, and some of the features you have access to. Remember that while some of these tools have free trials, they do have a price for production apps.

###### NOTE

Just as a mention, Sentry can be self-hosted for free if you want to test out features more before committing to a purchase. Or you might find that the self-hosted option is perfect for what you need. Do some research to see if the monitoring and logging tool you want to use has a self-hosted option. All the tools mentioned in this chapter offer both logging and monitoring since the functionalities are closely coupled.

Datadog has a 14-day trial so you can test it out. For these examples, you’ll work in the backend and trigger errors on endpoint requests and info logs. You’ll need to create an account and then [set up an agent](https://docs.datadoghq.com/agent/) to try Datadog. [Figure 12-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#datadog_agent_running) shows an example of the output I got running the agent on macOS after following the setup docs.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1201.png)

###### Figure 12-1. Datadog agent running

Validate that your agent is running and then install Datadog’s recommended logging tool, [Winston](https://github.com/winstonjs/winston):

npm i winston

Create a utility file called _datadog.ts_ in the _utils_ folder that will initialize an instance of the Datadog logger with all of the setup from the [Datadog docs](https://docs.datadoghq.com/logs/log_collection/nodejs/?tab=winston30):

```
// datadog.ts
```

Then you can add the `datadogLogger` in any controllers or services you want. For this example, you’ll add it to the orders controller like this:

```
// orders.controller.ts
```

Now if you go to your Datadog dashboard and check out the logs, you’ll be able to see whenever a request to `GET /orders` is made and any associated errors. The dashboard will give you a list of all of the logs that have happened in the past 15 minutes by default, which in production can be thousands of logs, as shown in [Figure 12-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#live_list_of_logs_being_monitored_in_da).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1202.png)

###### Figure 12-2. Live list of logs being monitored in Datadog

This is one way that monitoring works. You’re able to see the events that are getting triggered in real time with some statistics around how often errors are occurring, when they happen, and what they are. By monitoring your services this way, you can use your logs to figure out when incidents happen and start getting to the root of what’s causing them. You’ll also be able to set up alerts that trigger when certain threshold limits in your monitoring are passed.

You can look through all the logs over a certain time period, or you can filter by log type, service, or some other parameter. Once you find the log you’re looking for, you can click on it to get a more detailed view, as shown in [Figure 12-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch12.html#a_detailed_look_at_a_log_in_datadog). This will give you info like what was sent in a request or the error thrown for the response. A cool feature with Datadog is that you can send any type of logs to it. It also helps you track stats for how often users trigger certain functionality.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1203.png)

###### Figure 12-3. A detailed look at a log in Datadog

Datadog has a lot of information available, but you have to learn how to use the filters effectively. If you’re logging more than errors, it can take some time to get used to sorting through all the logs. Once you start adding all the backend logs to the dashboard, it will become even more important to have detailed log messages so that you know what data was used for different requests and how that connects to what was sent from the frontend.

Keep in mind that Datadog is just one tool, and the others I mentioned earlier are definitely worth researching more. You may discover that you like another tool’s interface better or that it has features you find more useful. Now that you know how logging and monitoring are connected, let’s move on to how you can use them to notice and handle incidents that happen.

# Incident Playbooks

Despite your best attempts to keep everything free of bugs and errors in production, there will be new and surprising ways the app will mess up. No matter what you do, users will always find ways to break the app unexpectedly, services will go down, and tools will have glitches. That’s why you have to have an incident playbook ready to quickly and procedurally address production issues. You need to work with the infrastructure team to create this playbook because they will be involved in the process as well.

## Playbook Stages

Several stages go into an incident playbook:

1. Detect an incident
    
2. Set up incident communication
    
3. Determine the impact and severity
    
4. Notify users
    
5. Notify the right teams
    
6. Delegate the incident responsibilities
    
7. Resolve the incident
    

Let’s take a look at each.

### Stage 1: Detect an incident

To get started, you have to know what an incident is for your app. This can tie into the metrics you’re monitoring, especially around security breaches and downtime. That’s why it’s important to get your logging and monitoring tool set up in a way that everyone understands and is familiar with. Remember, you can’t resolve an incident if you don’t know what it is. Once you have defined a list of incidents, when one actually occurs, you and the team won’t spend time arguing over what is happening.

### Stage 2: Set up incident communication

Once you know what your incidents are, you can move on to setting up the communication channels for them. Something I’ve seen work is having a designated Slack channel or Teams chat for each incident type. Then you can add the relevant people for each of those incidents to the specified channel. These initial channels are small, sometimes with fewer than 10 people. This keeps the communication more focused, and you all can give updates to others after you figure things out.

### Stage 3: Determine the impact and severity

After you have the appropriate group of people in a designated channel, you all will need to assess the impact and severity of the incident. As you look into the severity of the incident, note things like the number of users affected, which services are down and what functionality that affects, whether it’s something that can be fixed quickly, and how much the initial research reveals about the problem. This will give you an idea of how much the organization will be affected during the incident. It will also help you figure out how many people should be dedicated to working on the incident and where they should focus. After you and the incident team have determined the impact and severity, check in with the stakeholders and give them this information.

### Stage 4: Notify users

Start communicating with users about what is happening. If there’s a customer support team, they should be included in the stakeholder communication because they will be talking directly with customers; they need accurate information so that they aren’t blindsided. Once user communication has gone out, you can focus on the incident resolution.

### Stage 5: Notify the right teams

By now, the teams that need to be included should know they are going to have a role in fixing the incident. Based on the initial research, there should be a direction for all of the involved teams to move in. This is likely going to be some combination of the dev and infrastructure teams working together.

### Stage 6: Delegate incident responsibilities

Now you’re in the midst of searching for the root cause. Something I’ve seen work is having an incident call with everyone on it. That way, you can do quick screen sharing when someone finds something, and it makes teamwork more effective because you can throw out ideas as they come up so that no one is silently spinning their wheels. It also helps when you’re working with multiple teams on the same call because you can trigger something programmatically and have the infrastructure team see what happens or vice versa. This helps with the optics of your team and the engineering department as a whole, which is under scrutiny at this point no matter what.

# FREQUENTLY COMMUNICATE WITH STAKEHOLDERS DURING INCIDENTS

Always keep stakeholders up to date with everything you discover during an incident. Since they don’t have the same technical knowledge as you or the other teams involved, they won’t necessarily understand what is happening. This will help manage anxiety around the incident as well. Every organization wants to have a reliable product for its customers, so when something is wrong, emotional responses can be expected.

Keep the heat off you and the incident group by checking in regularly with updates. Even if the update is that you are still looking into the matter, that’s better than absolute silence. A rule of thumb I’ve found useful is to send updates every 10–15 minutes. This shows that you are treating the matter with urgency and it’s your highest priority to get things fixed, which gives stakeholders confidence in all the technical experts involved.

### Stage 7: Resolve the incident

Once you think you have a resolution, do some testing among the teams involved before you announce that everything is fixed to the stakeholders. Sometimes it seems like everything is fine until you click a different button. Have multiple people test the resolution in as many ways as you can as quickly as you can. You can update stakeholders and let them know you’re working on the resolution, but don’t tell them it’s ready until there has been some level of testing. Let them know to wait to communicate with users until the resolution has been deployed to production and the teams have done a little more testing there.

## Incident Response Template

Now that you know the main stages in an incident response, here’s a general template of an incident playbook you can use based on [incident response playbooks from Google](https://cloud.google.com/docs/security/incident-response); just remember to update it with the details for your organization:

- Identification (Stages 1 and 2)
    
    - Become aware that an incident is happening through monitoring and alerts.
        
    - Report the issue to the correct incident response team.
        
- Incident coordination (Stages 3, 4, and 5)
    
    - The incident response team triages the incident.
        
    - The impact and severity are determined.
        
    - Facts around the incident are gathered.
        
    - Relevant teams are notified to investigate the incident.
        
    - Stakeholders are notified with the current findings.
        
- Resolution (Stages 6 and 7)
    
    - The facts around the incident are researched.
        
    - Consistent communication is sent to key stakeholders.
        
    - Any steps to mitigate damage are taken.
        
    - The underlying issue is resolved.
        
    - The resolution gets tested.
        
    - Any affected systems and apps are restored to normal operations.
        
- Postmortem
    
    - Have a retrospective meeting on what happened.
        
    - Make a report that outlines timelines and actions taken.
        
    - Find areas that can be improved.
        
    - Plan for any processes or tools that need to be improved or maintained.
        

## Blameless Postmortems

After everyone is sure the incident has been resolved and users have been notified, it’s time to do a [blameless postmortem](https://sre.google/sre-book/postmortem-culture/). This is as crucial as every other part of incident management because it’s going to help find places where your playbook can be strengthened and where you can add resilience to your other processes. This is not a place to point fingers or assign blame to anyone.

Incidents don’t typically happen because of one person. Usually, multiple things happened leading up to the incident, and understanding what they are and how they all contributed is the purpose of the postmortem. One nice thing about having a dedicated channel to handle the incident is that you get timestamps for everything, so you can see how long the incident lasted and how well the team worked together toward the resolution.

Here’s a general template for some things you should cover when you’re doing a postmortem:

- Set a meeting time
    
    - Avoid placing blame on anyone.
        
    - Focus on the issues that happened in the process leading to the incident.
        
- Share everyone’s findings
    
    - The incident team should have notes on what they did during the resolution process.
        
    - Each team member can give the details from the part they reviewed.
        
    - No detail is unimportant.
        
- Write a formal report
    
    - Include everyone’s findings.
        
    - Highlight key actions that happened over the course of the incident.
        
    - Create takeaways for improvements.
        
    - Share the report across the organization.
        
- Praise people for doing things right
    
    - Consider having a meeting with teams across the organization based on the scale of the incident.
        
    - If someone made a mistake and owned up to it, publicly praise them for it to encourage a culture of this.
        
- Review the postmortem a few days later
    
    - This will give everyone time to take a step back and see if the report is good.
        
    - Do some role-playing through the steps that happened during the incident.
        
    - Accept feedback on what could have made the postmortem more effective.
        

Keeping postmortems blameless gives everyone more confidence to report issues rather than sweep them under the rug. If the first thought someone has when they become aware of an incident is that they might lose their job, that incident could last much longer than it needs to. It can’t be emphasized enough that no one person is the point of failure leading to an incident. There had to be, or should have been, multiple checks to go through in a release process. So focus on the process failure, not an individual failure.

You will eventually become an important point of contact when incidents arise. Since you’ve been involved with setting up much of the core functionality of the app and you’ve already worked with all the teams that could be involved in an incident, you have a broad view of how things work. This is a time when all your documentation and planning will shine because you won’t have to dig through code to find some of the connectivity answers quickly.

The main thing is to keep a calm attitude and approach the situation systematically. Others may be highly anxious because there’s an issue in production, so your levelheadedness will be a true asset. There’s a balance you have to maintain to keep those emotional responses tempered as you and the team work to resolve the issue as quickly as you can. When you look at large-scale incidents, such as the [CrowdStrike incident](https://www.crowdstrike.com/blog/statement-on-falcon-content-update-for-windows-hosts/), you can see that potentially millions of dollars are at stake and the impact could affect critical services like health care, transportation, and finance. You absolutely have to make sure everyone involved is in constant communication with stakeholders, even as high as the CEO.

# Conclusion

In this chapter, I went over the reasons why logging and monitoring are useful and some of the tools you can use. Your team will look at the logs and monitoring dashboards to help debug issues and get to root causes for incidents along with other engineering teams. You can debug any errors and get insight into what’s happening with your app more efficiently than if you were just looking through code and PRs. It can take some time to set up dashboards and figure out what the relevant data points are. It also takes some digging into the features of your tools because some of them can help you decipher all the data you get from your logs.

The logs and monitoring tools give you an understanding of the life of your app as it continues to grow in production. Make sure you have monitoring in place so that you can set up alerts when anything interesting starts happening in any of your environments, such as resources reaching their limits or endpoints being called an unusual number of times. Any of these could indicate that an incident is occurring. This will save you, the team, and the organization from situations that could have major impacts on revenue and user experience.