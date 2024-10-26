---
id: 01JB58FGP72PH2RK6TD4R84QTM
title: Chapter 14 - In Production
modified: 2024-10-26T16:40:38-04:00
tags:
  - full-stack
  - books
---
# 14. In Production

Chris Northwood[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_14_Chapter.xhtml#Aff2) 

(1)

Manchester, UK

In a digital organization, stakeholders put a lot of trust into their full stack team’s ability to make the right decisions, and into the approach they take to developing software. However, as Spider-Man’s Uncle Ben said, with great power, comes great responsibility. When a team has control over how it builds software, what gets built, and when it deploys, the team should also take responsibility for how that software runs in production. The term “DevOps,” a contraction of “developers-in-operations,” embodies this new skill set and mindset.

A full stack team will have an embedded QA (the result of a movement referred to as “developers-in-test”), but when this happens, it’s not just about increasing the level of test automation in a product, but also about ensuring the whole team is responsible for the quality of the output. DevOps is sometimes contextualized as “DevOps team” or “DevOp,” but this often refers to a traditional operations team that is using modern automation techniques, rather than one where developers and operations are embedded together in a shared team. At best, such teams are simply mimicking the process of DevOps by using new automation tools, confusing cause and effect. Teams that embed the DevOps culture may use automation, but that’s the result of the culture that has emerged. The most important part of the DevOps culture is that the team as a whole is responsible for how the product behaves in production. One common way this manifests itself is in a “build-it-and-run-it” team, where that team is the first line of support for any issues that occur in production. Some teams, especially those with a large user base, may have an external support team that handles the front line, with checklists and run books to help diagnose common issues, but those documents are developed by the team, which has a close relationship with front-line support.

IT in a traditional organization, especially those where teams are built around technical functions, can develop a silo mentality. All too often, teams operate with a “works on my machine” mentality, rather than accepting joint responsibility for the good of the organization and its users. At its very core, DevOps aims to break down those barriers, with more reliable software and a short time to fix issues when they do occur, being the aim. When a team I was working on adopted DevOps, having to carry around the “batphone” (an on-call mobile phone that could be used as a first point of contact for issues) drove home how real these new responsibilities were. No one wants a 3 a.m. phone call because their product has fallen over.

## Fire Drills

The people who best know how a product can break are the people who built it. Fire drills are an effective way to explore these scenarios and identify work needed to harden your software.

For a team new to DevOps, a fire drill might just be a theoretical exercise. It starts by asking questions such as “what would happen if our database became corrupted?” The team identifies any potential problems, and suggests solutions to these. For all of the identified scenarios, the team can check the following:

- How would we know this situation is happening? Do we have sufficient monitoring around the application to identify this as a situation, and how would it manifest itself?
    
- Do we know what the impact will be? Is there any temporary mitigation we could put in place?
    
- Do we know how to recover from this situation?
    
- How likely is this situation to occur?
    

By answering these questions, the team can identify work that might be needed; perhaps additional logging or alarms need to be implemented, or a checklist added to the application’s run book. Mitigating against these scenarios can be captured as work and added to the backlog like any other work. User stories such as “As an on-call developer, I want the system to fail over to a database replica when the primary becomes unavailable, so that I can minimize downtime during an incident,” or “As a member of the support team, I want to see hostnames in error logs when a connection fails, so that I can quickly identify whether a single back-end server has failed” are perfectly acceptable, and phrasing them in that way helps product owners prioritize the work. No one wants an outage! However, identifying likeliness can also help with this. If your application is deployed to a cloud provider across multiple regions, it is possible to have a multi-region failure (so you may want to also use a second provider to host an instance of your application to mitigate this risk), but the likelihood of such an occurrence might be so low that it is not worth the investment to make the improvement.

Once a team feels confident they have a product that is resilient, they can start turning these fire drills into more practical exercises. Identifying flaws in a recovery plan in a controlled environment is much better than discovering them in a real incident.

For example, in a test environment, a team may decide to corrupt a database, or take a key server offline, and check that the system handles the failure as expected, or any manual recovery steps (such as restoring a backup) work as intended. This only works, of course, when there is parity between these environments and the production environment. Particularly brave teams often choose to run these fire drills in their production environment, especially when there should be no impact on the user. Even the best-intentioned team will have differences between environments—such as the volume of data, or the load on a system—which can change the impact of an incident.

Every team member should take part in fire drills, and they should happen regularly. This can include non-technical team members, such as the product owner, to help build a whole team mindset about operating the service. Sometimes it’s obvious that a change or new feature to a system can impact how a system responds to an incident, or introduce a new way a system could fail, and those can be identified and dealt with early—but sometimes it’s less obvious. Identifying and rehearsing ways of handling the most common or likely failures will significantly improve your time to respond.

There is a growing set of tools that can automate the process of running a fire drill by deliberately injecting failures into a system on a continuous basis. This can result in a high level of resilience, forcing you to build in automated recovery for certain types of failure (and reducing your support workload). Netflix’s Simian Army is the most famous example of this, and includes the Chaos Monkey, which randomly kills single servers, but there are other tools (such as introducing latency to network connections) too. If your team feels comfortable running these tools in production, then you should be proud that you have a high degree of confidence in the resilience of your product.

## Run Books

Run books for an application become a kind of bible while an incident is occurring. In a high-pressure situation like an outage, the last thing you want to be doing is trying to remember where on disk a log file is stored, the URL for a status page, or the details of any important upstream systems the application depends on.

The other thing a run book should include is checklists. When an alarm goes off in flight, an airline pilot will have a series of checklists to run through to help diagnose what could be wrong with the plane and how to correct it. The last thing they want to do is crash because they forgot to check the flaps were in the right position, which is a simple task that is easy to overlook in an emergency.

Fortunately, when an incident occurs in your application, it’s unlikely to be a life-or-death scenario, but there will almost certainly be real pressure to resolve it, which may induce panic in the severest cases. Having a checklist that was written by you and your team when you had a cooler mind will allow you to negate some of that panic, and get the incident dealt with quicker.

What should a good checklist do? It should give you a clear set of actions to take in response to an alert. An alert could be an error report from a user, or one generated by your monitoring system. Take, for example, the following two checklists:

### Zenoss has Generated a Lowdiskspace Alarm for the MySQL Server

_*Impact: None yet, may escalate into an outage for the e-commerce catalogue*_

1. 1.
    
    Log in to MySQL server
    
2. 2.
    
    Delete all records from the session table that were last updated over seven days ago
    

### User has Reported they have not Received the Daily Marketing Report e-mail

_Impact: Internal users may not be able to track performance of time-sensitive A/B tests_

1. 1.
    
    Ask user to give their e-mail address, and to check their spam folder.
    
2. 2.
    
    If the message is not in to the spam folder, log in to the report system at [https://reports.marketing.example.com/admin/](https://reports.marketing.example.com/admin/) and select “Edit Report Recipients,” and ensure that their e-mail address is in the list of recipients. If it is not there, re-add it.
    
3. 3.
    
    Once the user is in the recipients list, select the user and the “Manual Resend” action on the screen to trigger the re-send. Ensure the user has received the report.
    
4. 4.
    
    If the user believes they should previously have been a member, then select “Audit Log” on the admin screen and check for any actions that would have removed them, and follow up with the responsible user.
    
5. 5.
    
    If the user was on the recipients list, then select “Report Generation Status” and ensure that the report run time field is showing a time at approximately 6 a.m. that morning. If the report has not run, escalate the incident, as it will affect all users, and follow the “Daily Marketing Report Not Generated” checklist.
    
6. 6.
    
    If the report was generated, then log in to the SMTP gateway with SSH at smtp-gw.platform.example.com and run: grep <email address> /var/log/mail/outgoing.log. Check for any errors or deferments in the log. If there are deferments, then force the message to be processed by running “process-mail -mid <messageid>” with the message ID from the log file. If there are errors, escalate to the corporate e-mail team.
    
7. 7.
    
    If the message does not appear in the outgoing log, then check the mail reporting application log by accessing the logging portal using the credentials in the team password store. Apply the “marketing report error” filter, then check for any appropriate error messages.
    

The second checklist may seem overly verbose, but if you’re unfortunate enough to be a new developer on a team doing an after-hours on-call rotation when an incident occurs, you’ll be grateful for the detail.

Being explicit can also help avoid mistakes. For example, on the first checklist, it might be easy to accidentally access the test database to make the change, and then wonder why the issue has not been resolved. If this is a system that has been stable for a number of years with little need for maintenance, then it’s easy to forget exactly which MySQL server it runs on, or what the credentials are—is it my personal developer login that will give me access, or some global one? How exactly do I access the database—is there phpMyAdmin, or can I connect using a desktop tool, or do I need to SSH in and run the command line? The second line is also potentially dangerous; all it takes is for someone to write a SQL statement with a less-than swapped for a greater-than, and all recent sessions have been lost. Better to have a simple script that can be used, or better yet, a scheduled maintenance task to avoid the situation ever occurring. The first checklist is also incomplete: what happens if it’s not due to a temporary table getting full that the server is out of disk space?

Write checklists assuming that its reader knows nothing about your product, or your organization. This is especially true when you're not first-line support for your application. It can also be helpful to link to previous incidents (which may be recorded in a ticketing system), as comments on those tickets can help diagnose complex issues.

The final elements a run book should include is a clear path of communication. For high-severity incidents, communicating the impact is key. For example, in an outage where the checklist has not resolved the incident, contacting the technical lead for that team can provide additional insight into how to proceed. In another case, if an outage has impacted a critical business function, such as the ability to sign up for the web site, then the marketing or customer support team might need to know, so they can handle any complaints coming via e-mail, or suspend a major marketing campaign that is expected to drive sign-ups until the incident is resolved.

Many an incident has been delayed simply because no one knows who is responsible for a service that is causing an outage, or they can’t get in touch with the right people to help resolve it, so it’s important to keep this list up to date. People join, move around in, and leave organizations constantly. You might want to consider simply including roles, and linking to a global address book, or some other solution that works best in your organization.

If your product only needs support during business hours, then having a team e-mail address or similar might be enough, but for after-hours support, you may need to get in touch with a particular person who is the designated support contact, or the person an automated monitoring system escalates to, for a particular period of time. To accomplish this, an on-call system can be used, where team members take turns providing out-of-hours support. For this system, a rota is drawn up and published, and the person who is on call ensures they are available outside of those published hours. Often, this rota with contact information is linked to or inserted in the run book, and when someone else needs to escalate to you, they can consult that book, or a monitoring system can programmatically query it to know who to page. Other approaches include having a physical phone that is passed like a baton with a set phone number, or simply updating the contact details in the runbook and monitoring system at every hand-over.

For complex issues, there may need to be an escalation plan in place to bring in other team members, or members of the wider organization, if a situation is particularly complex and critical and cannot be resolved solo. This should also be put in place, but the decision to escalate left to the person on call, rather than to third parties. Some automated on-call systems do allow automatic escalation if an incident is not acknowledged, too. This can mean that the team is always on call, but these escalations should be infrequent.

### The Human Factor of On-Call Rotas

Many organizations introduce an on-call rota as part of moving to a DevOps culture, but this should be done with caution, as it can change the nature of the work and severely impact work-life balance for a team. At the very least, people who are on call should be able to influence any shift patterns they are given. Being on call for too long can lead to burn-out, and spending all night fire-fighting a live incident and still being expected to turn up for a nine-to-five working day is usually not feasible. Many jurisdictions have rules around working hours and minimum break requirements that must be taken into account when planning a rota. When introducing an on-call system, you might also need to consider how the compensation for a team member might change, as they are expected to take on additional work, and what flexibility may be needed to support their work-life balance.

## Monitoring

Every product with users will be monitored by default, but if the best way to know if your system is down is because your customers are tweeting about it, you have a problem. Implementing effective monitoring will allow you to know about problems before your customers do, and get them resolved quicker. Even better is monitoring that can alert you to a problem before it turns into a full-blown outage (for example, increasing response times or low disk space).

Monitoring can give us two types of data: qualitative and quantitative. Qualitative data is easy for humans to read an interpret, but harder for computers. Quantitative refers to numbers: counts of requests, CPU load, etc. This is the data computers are great at processing. In terms of monitoring, we generally talk about logs (qualitative) and metrics (quantitative). Sometimes logs can be very structured, such as HTTP access logs, and these are often transformed into metrics, but often they’re much less structured, such as tracebacks or other log messages. During an incident, both types of data are important. Metrics can tell you something is wrong and give you a start as to where to look, and logs can give you further context and a high level of detail.

Monitoring is incomplete without alarms or alerts. Capturing a lot of data can be useful for post-mortems or other analysis, but for the purpose of monitoring your system in production, you need the data to tell you something on the spot. These alarms are typically set against the metrics that your monitoring system captures, leaving the log files available to give you greater insight into your system.

Alerts are often triggered by rules that set thresholds against the metrics, and often contain a severity. For example, you might specify a rule such as “Raise an alarm for ‘LowDiskSpace’ with level warning when the free disk space is less than 5GB,” in addition to different severities against different thresholds. Often, these alerts can be configured to clear automatically when the rule no longer applies, and many alert systems employ the concept of “flapping,” which occurs when an alert is raised and then cleared several times in rapid succession. Systems often have different notification rules for these alarms—for example, a warning message might simply send an e-mail to be dealt with during office hours, but a higher severity might cause a text message or automated phone call to be sent. Figure [14-1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_14_Chapter.xhtml#Fig1) depicts an example of how an application can be monitored.

![../images/471976_1_En_14_Chapter/471976_1_En_14_Fig1_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_14_Chapter/471976_1_En_14_Fig1_HTML.png)

Figure 14-1

Showing how an application can be monitored

When setting alerts, it’s important to consider any cases where absence of data can indicate a problem. Setting a threshold to say “more than five errors per minute” makes sense, but a threshold such as “serving less than 10 good requests per second” can be helpful in diagnosing network or load balancer issues that are stopping requests from hitting your server.

Sometimes alerts can be triggered directly as the result of an event. For example, if a circuit breaker is triggered (see the Designing Systems chapter for more on circuit breakers), you may want to flag an alert directly, rather than going through a metric.

Metrics come in various shapes and sizes too. Something like CPU usage or disk space is an absolute value that can be sampled, and thresholds set against directly. Others are simply counters. Counters are incremented when certain events occur (such as number of requests or errors served). These counters require processing to be made useful for thresholds to be set against them, usually into a per-second or per-minute rate, or even measuring the rate of change. Sets of data can be even more advanced in the form of response times that can be processed, such as averaging, to give a more meaningful metric.

There are lots of tools available to help you monitor your systems, from hosted and managed systems like Datadog or New Relic to self-hosted systems like Nagios, Sensu, or Grafana. Some try to do everything, but others separate monitoring and alerting. Some try to give you many metrics out of the box, but there will always be domain-specific metrics you will want to expose directly, so when building your code, it’s important to consider how you might want to hook into monitoring and metrics around key functions. This might include communications with dependencies (number of responses made to a back-end system, errors received, response times, cache hits and misses), or other business rules, such as average size of a response.

It can be tempting to set the metrics of your monitoring system, business KPIs and analytics to track the same thing, but actually these concerns are all different. The first relates to monitoring the health of your system from a technical perspective, and the others are about tracking how well your application is meeting its business goals and user behavior. The same solution might not satisfy all three different needs.

A final note about logging. Separating out your logs based on concerns will make life a lot easier for you when an incident occurs. Trying to find an error in the middle of a busy access log is hard, so you should try to keep a log of activities that happen on your system (such as an access log) separate from any audit logging or error logging. In an incident, the error log will be your first port of call, so ensuring only relevant information remains in that will make your life a lot easier. The error message that you log should contain enough information to help you debug an issue. If a request has failed, then make sure you log enough information about the request (such as the URL it was trying to hit), as well as the reason for the error so you can attempt to reproduce the error condition, and to understand how that error condition can propagate through your code. If the same error could occur at multiple points in your codebase, make sure you include a way of tracking it back to the exact line of code where the log message actually got written, to help you verify that the branch of code you expect to be executing is doing so. Logging tracebacks from exceptions (taking care not to expose these to end users) can be invaluable. The last thing you want to do is have to make a deploy of your code during an incident in order to add enough logging to diagnose a problem. However, be careful with what you log. Adding in log messages around login or password validation can lead to accidentally logging people’s passwords and creating a security breach. This is especially true when enabling raw logs—for example, for database connections or web connections, where the body of the request is kept.

There are many logging frameworks available for most languages. At the very least, you want one that will allow you to add timestamps to messages, as well as make it clear in a plain log file where a multi-line log message ends and the next one starts. There are also good tools for reading logs, including some that can aggregate the same (or similar) messages together and provide a count, which can help you sift through busy ones. However, having a plain text log file on disk in the middle of a crisis can provide quick access to logs that these more complex tools can cause issues with.

If your application is deployed on multiple servers, or you’re using the immutable infrastructure pattern, you will also want to consider a particular way of aggregating logs: sending them off your box to a central service that allows you to see the state of your whole application (including figuring out if an issue is limited to one server). In the case of immutable infrastructure, aggregating is necessary to maintain your application’s logs so you can diagnose them at a later date. Otherwise, they are lost when an instance is deployed or reloaded.

## Responding to Incidents

An incident always happens as the result of a change—a change in code, configuration, or underlying database, or a request by a malicious user. This change can be very subtle. It could be a particular combination of long-standing requests that exposes a bug that was never tested, or even just the onward flow of time. The change may not be made by the development team, but there is always a root cause for an incident, although it may not be possible to diagnose the very root cause (for example, a failing hard disc). When it comes to an incident, your job is not only to resolve it and recover any loss in service, but also to understand what change caused the incident to keep it from happening again (a root-cause analysis).

Incidents ideally start with a team being notified when a system has failed. If you are being notified by a third-party, such as a dependency, or by your users, this highlights a gap in your monitoring to be resolved in future. The first point of call should be the run book to find if there are steps to resolve it, or if there are any known issues or previous occurrences that way indicate which actions should can be applied.

When following steps, you should make a note of what you have done, perhaps in a logged chat system or on a ticketing system. This will allow you to look back at what you've tried, and if you need to escalate, to quickly hand over that information. It also allows some stakeholders to monitor progress, and at the end of the incident, for you to look back at what you did and use that to drive any future improvements.

Of course, the actual actions that are taken are context dependent, but using pre-prepared tools—such as your run book and any debugging tools you may have identified as needed in your fire drills—should hopefully allow you to address the incident in the short term, and then identify a root cause and any deficiencies in your system to be addressed.

## Summary

In many organizations, full stack teams are empowered to build and run their own systems. Running a system requires a different skill set than building it, and this was often reflected in organizations’ structures, with separate operations teams and development teams. When development teams run their own systems, then they must apply those operations skills themselves, leading to a way of working known as DevOps (developers-in-operations).

When building a system, you should consider how you will detect and diagnose any operational issues. The most common ways to do this are to emit useful logs and metrics that can be collected and searched, and to use monitoring tools that run checks against the system to catch common failures. When a check fails, then an alert is sent to the team, or a nominated on-call developer, which causes them to start investigating the fault. Alerts can also be triggered when metrics breach particular thresholds—for example, if an error count spikes.

To ensure that an incident can be resolved quickly, rehearsals known as fire drills can be run that simulate failure in a controlled way to instill confidence in resolving issues. These procedures to resolve common issues, or pointers about how to debug or what a particular alert means, should be recorded in a run book for that particular application.

With these in place, a team can confidently run their services and take incidents in stride.