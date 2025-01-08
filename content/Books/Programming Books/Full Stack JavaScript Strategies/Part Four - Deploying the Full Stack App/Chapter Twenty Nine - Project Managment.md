---
id: 01JH1P1CZZW4FTA63Y8AFF8642
modified: 2025-01-07T19:46:48-05:00
---
# Chapter 29. Project Management

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 29th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

There’s more to your dev career than doing all the technical work. You’ll have to do some form of project management. This is how you start taking more ownership for what gets shipped to production for users. Now is when you start really asking questions about features and the roadmap and work closer with the Product team. Get comfortable asking about the validity of a feature and push back when new functionality will change or break something already in place.

As the project matures, Product is going to want to collaborate with you and get your opinions on how features should work and what that looks like from a technical perspective as they expand the roadmap. You’ll be expected to chime in with things they haven’t considered, such as how designs might take longer to implement than it seems or the difference between frontend and backend work. You’re going to be an expert on how the app works and how changes might affect it, so you have to become comfortable communicating that openly and honestly.

In this chapter, I’ll go over:

- Approaching sprint discussions
    
- Defining and managing tasks to keep the team at a steady pace
    
- Handling communication with Product
    

This process is going to vary at every organization you work with, but you can bring up points from your experience that will be valid almost everywhere. The key is understanding where the lines are between you and Product so that you can better perform your role. I’ve been in some organizations where I’ve crossed the Product line too often, and I’ve had instances where I wasn’t speaking up enough in Product discussions. Work with your manager to get a grasp on what the expectations are and make sure everyone knows them. It avoids a lot of confusion and miscommunication later.

###### NOTE

In the cases where I was crossing the Product line, that usually involved defining acceptance criteria for a feature. The Product team does a ton of user research that the devs aren’t involved in and may not be aware of. So things like this can be left up to them, and you can ask all the questions you need to get clarification. Try not to make too many assumptions about how the feature should work from a user perspective and collaborate with Product to make that definition clear.

# Sprint Discussions

There are a few common themes that come up during sprint discussions, such as estimates, requirements, and division of tasks. This is a group effort and takes collaboration. It may feel tedious at times, but it’s helpful to ask as much as you can initially so that you and the team have a smooth sprint once it’s started.

## Estimates

Something that will come up early and often is Product asking for estimates. They are trying to get an idea of how long it will take to develop, test, and deploy a feature to production so that they can communicate a timeline with sales, customer support, and any other stakeholders. This may come up during sprint planning, when you and the team are picking up work, and you might feel pressured to agree on a date. Until you have all the requirements and designs as well as time to do some research, resist the urge to throw out a date or agree to one.

It’s easy to look at a ticket and think it’s simple to add something like a new dropdown or a new endpoint, but then you get into the weeds of it and find out it’s way more complex than you thought. Do not give your most optimistic estimates because development rarely goes perfectly. Give yourself and the team some breathing room with larger estimates. You’ll be surprised at what you run into when you try to make small changes, even if it’s something like updating copy or colors. The last thing you want to happen is you and the team coming in late on deadlines because you underestimated how much work something will take.

There’s an old saying: “underpromise and overdeliver.” Even if you know for a fact that you can finish a certain amount of work, it’s usually better to agree to a little less just in case something comes up. If you end up having time to finish everything and then some, then everybody’s happy and you’re not stressed out. Either way, you still meet the commitment you originally made.

###### NOTE

I want to stress how important it is to not give the most optimistic estimate. When you know the app and the codebase really well, you can come up with a solution pretty quickly on the fly, and that’s fine. You can include the details in the ticket unless you’re trying to help another dev have a chance to think through a solution themselves. Also, don’t give an estimate on work the team has to do until you talk it over with them.

## Dev Capacity

It will take time to establish a baseline for the number of points the dev team can handle. When you have a new team or a new team member, it may take two to three sprints to get an accurate measure of the number of points the team can complete. This should include your buffer for overdelivery. You can see how much other work gets finished outside the feature and bug tickets. Eventually, this will account for days off and holidays as well. Until you have a few sprints to set the baseline, it’s hard to say how much the team can get done.

Remember that point values will vary across organizations. You may work at an organization where an entire feature is 5 points and other places where the same work would be 13 points. Points can be associated with a certain amount of time; for instance, 5 points might represent a half week of work. But points are supposed to be a [representation of how complex a task is](https://www.atlassian.com/agile/project-management/estimation). They should take into consideration all the supporting activities you need to do.

Remember this when you are choosing tickets to work on for the sprint and leave yourself some capacity for other things. You will still have to do PR reviews, address bugs that come up from QA, and handle the other random things that inevitably pop up. When you max out the number of points or tasks you can handle, you don’t leave yourself room for these other little things. You’ll want to look out for the other devs, too. During sprint planning, when tickets are assigned and pointed, double-check with everyone that they’ve taken on a reasonable load.

It’s easy to fall into a cycle of taking on too much and working more hours as the team slowly gets stressed out. This is when your experiences will come out because you know how much you can handle without getting burned out. It’s tempting to try to show how much you know or how fast you can get through tickets, but that will lead you to a rough spot that can take months to overcome.

###### NOTE

Burnout is a real problem. I’ve experienced it several times over the course of my career, and each time it took months to come out of it. For me, it always starts by taking on a little more and more until my whole existence is work. Then, everything piles up, and eventually, I crack under the pressure. Burnout is not easy to recover from, so take [steps to prevent it](https://www.talkspace.com/blog/how-to-prevent-burnout/) from happening.

## Feature Requirements

Another thing that can’t be emphasized enough is making sure you have everything you need before you agree on an estimate for a ticket. All the designs and requirements should be ready for you so that you can accurately determine how long a ticket will take. Agreeing to incomplete tickets usually leads to scope creep, which balloons the work well past what you estimated. Of course, questions will come up as you do development, but you shouldn’t have questions about the core functionality of a ticket.

You should also consider having a research category of tickets when the details of a feature are still being created. [Figure 29-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch29.html#example_of_a_research_ticket) is an example of what that ticket might look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2901.png)

###### Figure 29-1. Example of a research ticket

If Product hasn’t written up specs, work with them to get that documented but try not to write it for them. This is where that fine line between responsibilities comes in. The specs should be able to tell you how the product works, what it should do, when it should do it, and how it affects the user. You should be able to break that information down into the technical implementation and ask Product more detailed questions. That’s the ideal partnership between Product and the dev team. All your sprint discussions should start based on some documentation they give you. [Figure 29-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch29.html#example_of_a_well_defined_ticket) is an example of how a good ticket could be written using the [Gherkin format](https://cucumber.io/docs/gherkin/reference/) we discussed in [Chapter 21](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch21.html#frontend_testing).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2902.png)

###### Figure 29-2. Example of a well-defined ticket

Push back on Product hard if the documentation is not ready for you and don’t agree to any work that hasn’t been properly defined. You might even push for things like epics to group a set of tickets under if the feature is large. An [_epic_](https://www.atlassian.com/agile/project-management/epics#:~:text=Summary%3A%20An%20agile%20epic%20is,epic%20serves%20to%20manage%20tasks.) is a large feature that can be broken into several stories. It likely has multiple components in a design and possibly involves backend updates. This requires multiple tickets to chunk that work into tasks a single dev can manage or for a couple of devs to work on independently. This is also how you can give better estimates for the overall feature because you know the smaller pieces that build it.

For medium features, you may have to work on a few components or make smaller backend updates. These tickets can also be grouped under an epic for better clarity, but that will be a decision for you and Product to make. Small features usually involve a couple of tickets and can generally be addressed by a single dev, so they don’t need an epic. Work with your dev team to figure out how you need to break apart the feature to determine its size.

## Dev Team Ticket Review

It’s a good idea to review all the tickets or feature specs as a dev team so that everyone understands what’s going on. This is how you can have more productive discussions with the Product team. While it does add another meeting to everyone’s calendar, it will save you all time over the course of the sprint. By facilitating these dev meetings, you’re taking more of a leadership role and showing your ability to juggle priorities outside the code.

When you have group dev discussions about tickets that haven’t been thoroughly defined, more questions will come up. People on your team will have different experiences and perspectives that will help make the ticket and the work more complete. As an example of how this meeting might go, [Figure 29-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch29.html#example_of_a_poorly_defined_ticket_befo) shows a poorly defined ticket.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2903.png)

###### Figure 29-3. Example of a poorly defined ticket before the dev meeting

Before the meeting, you should spend time going through tickets for the upcoming sprint to highlight the ones that need the most review. Set aside at least an hour to look over everything and start compiling a list of questions. Plan for the first 15–20 minutes of the meeting to be time for the team to review the tickets because it’s likely they haven’t had a chance. Send out reminders to the team a day before the meeting so that they can set aside time to review tickets. Sometimes it can be hard to get people to speak up during these meetings, so you can start with a question you have and then have everyone ask a question. It also helps to make a checklist of the things that should be defined in any ticket. Here’s an example of some of the things I ask for in tickets:

- Where are the designs?
    
- What data do we need to display?
    
- What happens with different statuses?
    
- How should we handle missing values?
    
- Are there restrictions on the parameters a user inputs?
    

By the end of this process, you’ll have a number of questions for the Product team. Send Product the team’s questions ahead of sprint-planning meetings and make sure to follow up. By the time you get to sprint planning, the ticket will be refined and look like [Figure 29-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch29.html#example_of_the_refined_ticket_after_the).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2904.png)

###### Figure 29-4. Example of the refined ticket after the dev meeting

As you get more comfortable with this process, you can start passing the responsibility to lead the dev meetings around so that others on the team can get that experience. Your sprint discussions will lead to discovering more about the feature than Product initially specified, and that’s a great thing. It will help give everyone a more realistic view of how much work a feature will take, the approach that should be taken, and the context for why one approach is better than another.

## Roadmaps

Whenever you have discussions about an upcoming sprint, you should review the Product roadmap. This will keep everyone up to date on what’s coming and when the anticipated release dates are. That way, you aren’t surprised by things the Product team has known about for a while. Reviewing the roadmap will keep everyone on track with priorities, and you can anticipate what you will focus on next. So if there’s something you can do to help prepare the codebase for the next round of features in your current sprint, you can discuss it with the team to get early feedback. [Figure 29-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch29.html#example_roadmap) is an example of what a roadmap can look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2905.png)

###### Figure 29-5. Example roadmap

Remember that roadmaps help your nontechnical colleagues make sense of the work the dev team is doing. This extends all the way up to leadership, such as VPs and C-suite executives. It keeps them aware of what’s happening and when, so they can handle any public relations around all the work your team has been doing. It’s easy to get lost in the code and forget about why the roadmap matters, but this helps the business present an informed message to users and any external stakeholders.

The dev team should also have some form of an engineering roadmap, even if it’s just a collection of tickets related to updates and refactors in the code. It’s important that technical concerns are brought up during these meetings as well. When you know that a third-party service is going to update an SDK, a package is being upgraded by a major version, some refactors need to be done to unblock future dev work, or you want some time to implement documentation or other DX tooling, Product needs to know about those tasks. That way, they can give you space to work on them without being pressed by stakeholders on feature deadlines.

A cool thing that happens when you start reviewing the product and engineering roadmaps together is you might find where new useful features can be added. Seeing the features expected for the year laid out in one place can generate some creative ideas. For example, Product might want to add email alerts to user actions, and you already have better event handling on the dev roadmap. That can lead to a new combination of the two tasks that gives you both what you need at the same time. Without that discussion, they would have been implemented separately and maybe at different times.

# Defining and Managing Tasks

There are a lot of approaches to how you define and manage your tasks as a dev. Many of these will be based on personal preference, experience, and the way your organization works. A few things you may want to keep in mind are how your tasks fit into the roadmaps, how your work may overlap with other developers’ work, feature priorities, and your own interests. These things will go into creating your own dev workflow, which will help you consistently finish all your tasks for a sprint and help out the rest of the team without too much stress.

## Maintain Team Awareness

As you and the team work on features, keep yourself up to date on the roadmap and have a general idea of all the features that are currently in development. You’ll be asked to help other team members with their tasks from time to time. Being aware of what everyone is working on is a great way to add value to the team. It doesn’t mean you’re responsible for tracking everything and making sure things get done on time. That’s a job for the tech lead or engineering manager. This is for your own sanity, and it helps when discussions come up about how to shape some of your technical decisions.

# DIFFERENCES BETWEEN TECH LEADS AND ENGINEERING MANAGERS

A [_tech lead_](https://leaddev.com/personal-development/what-tech-lead-first-among-equals-developer-team) is a dev who has other responsibilities within a team with respect to their skills. For example, you might have a backend tech lead who handles some of the more difficult tasks with that part of the stack and becomes the go-to person for questions in that area. They’re still individual contributors and don’t have direct reports.

An [_engineering manager_](https://leaddev.com/career-paths-progression-promotion/what-engineering-manager-taking-step) is usually responsible for strategic planning in the department, budgeting, and hiring. They have direct reports and rarely get into the code. They may occasionally jump into the code, but they typically provide the resources for their reports to get their tasks finished.

For example, if someone on the team is implementing new shared components and you’re also working on the frontend, you might find that there’s some overlap in your work even if it’s for a different feature. Or if someone is creating a new database table for the backend, you might be able to add your own fields to it instead of having conflicts when you try to create something they already made. Keeping mental notes of what others are doing will help you all avoid duplicate work. Even though you will have dev meetings throughout the week, it’s useful to keep yourself aware of what’s coming up.

Having a shared goal for the end of the sprint will keep the team focused. It brings a sense of unity to the tickets and clarifies with the team what they are building toward. As everyone works on their tickets, they will have a better idea of how what they are working on connects to the overall goal of the sprint and the work the rest of the team members are doing.

In some organizations, you’ll find that Product puts tickets into the sprint based on the priority they have and the team’s capacity. At other organizations, engineering will have a more hands-on approach to deciding what tickets are in the sprint. Either way, tickets will be assigned to the team based on how much capacity an individual dev has and sometimes whether they are more focused on the frontend or backend. You might end up working on a large feature that spans a few months, or you might have a lot of smaller tickets that jump everywhere.

## Write and Clarify Tickets

After you’ve done your research, created the smaller tickets, and discussed your findings with the relevant people, you can finally add details to the tickets you’ve created. Try to include as much as you can so that anyone on the team can pick up one of your tickets with few questions. Include links to feature specs, UI designs, and any other information that can help a dev get started. Also try to list some initial acceptance criteria so that when you do have those sprint discussions, there’s a starting point.

###### NOTE

If there’s a specific feature or task that interests you, speak up! You don’t want to be passive with the work you get to do. Keep in mind that you may not always get to do the stuff that interests you, but that will make you a better dev in the long run.

This may include researching tickets about something you’ve never worked with. When you’re working on a research task, try to create smaller tickets as you discover functionality that needs to be added. You don’t have to fill in all the details immediately, but having some initial placeholder tickets will help Product with planning deadlines and setting expectations early.

[Figure 29-6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch29.html#feature_epic_with_tickets_in_trello) is an example of a large feature epic with smaller tickets.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_2906.png)

###### Figure 29-6. Feature epic with tickets in Trello

It’s usually a good idea to have focused tickets for the frontend and backend work. Then you can have more details about the backend work by endpoints, resources that need to be created, and events that need to trigger actions in different systems. The frontend can be broken down into components that need to be added or updated, API requests that need to be made, and any hooks or other helper functions that need to be created. As you define these tasks, you want them to be as independent as you can make them.

As part of the focused tasks between the frontend and backend, it’s helpful to have tickets to add mock data and tests. With this, the frontend devs know what to expect from the backend, and the backend devs understand what they need to do for the frontend. That will enable multiple devs to work on the same feature without having a lot of overlap. This really helps when someone has finished all their tickets for a sprint and is looking for small pieces of work to do to help keep the team moving forward. Again, it will be on your tech lead or engineering manager to coordinate who’s working on what, but you can be a huge help by writing good tickets.

## Consider Overhead Tasks

When it comes to managing your own tasks, you need to take a realistic look at your capacity. Are you going to be out a few days during the sprint? Do you anticipate working with QA to nail down bugs before a release? Is there a lot of dev work going on that will lead to a flood of PR reviews? Are any new team members joining? Are you integrating a new third-party service? Will there be any package version upgrades? All of these will take away from your capacity for new work, and you have to account for them so that you don’t end up taking on too much.

These are the subtle tasks that tend to be overlooked during sprint planning, so it’s up to you to consider them for yourself. Bring up these tasks in your sprint discussions with Product so that they’re aware of the other things on your plate. This helps set an example for the dev team because it will make others consider what they have to do as well. Then you all start making more accurate estimates for how much work you can get done. As your estimates become more accurate, Product can come up with more realistic deadlines for stakeholders.

Something else to keep in mind as you go through the tickets in your sprint is their priority. There might be some work you’re looking forward to more than other parts, but stay focused on priorities. Work on the highest-priority tickets first, and then you can enjoy the end of your sprint as compared to rushing at the last minute. Things always come up when you’re working on high-priority tickets that you don’t expect or didn’t account for. So it’s better to start them as soon as you can.

## Pace Yourself

You have to find a good balance between challenging yourself and taking on too much. The last thing you want to do is end up with more tasks in a sprint than you can handle. If Product wants you to switch your focus midway through a sprint, ask them what should be deprioritized. While it may be necessary to push through a heavy workload from time to time, that should not be the expectation every time. Be conscious of what you agree to because it will set a standard that will be hard to change as time goes on, and it may become part of the company culture if you agree each time the situation comes up.

You have probably experienced burnout a few times throughout your career. It’s not the easiest thing to recover from, so it’s best to try to avoid it. By keeping a reasonable workload, you set an example for the team to take care of themselves as well. As we discussed in [Chapter 22](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch22.html#frontend_debugging) on debugging, make sure you take breaks from your work throughout the day. The [Pomodoro technique](https://todoist.com/productivity-methods/pomodoro-technique#three-pomodoro-technique-rules-for-maximum-productivity) is a popular approach to balancing focused work time with regular breaks. It’s easy to get hyperfocused on a task that you want to get finished, but take the break anyway.

## Manage Context Shifting

Over the course of a day, you’ll likely have multiple meetings, get messages from other devs, answer QA and Product questions, work on a couple of different tasks, and deal with any alerts or other miscellaneous happenings. This is a lot of context shifting. When you do that, it slows your workflow because you have to orient your mind to the new task on hand. Doing this multiple times throughout the day can cause mental fatigue and can even create some emotional overload.

When you’re working on a feature and you’re really focused, it can take a while to shift your thoughts to an unrelated meeting. So you have to do things to protect your [flow state](https://www.medicalnewstoday.com/articles/flow-state#:~:text=The%20term%20%E2%80%9Cflow%20state%E2%80%9D%20describes,about%20themselves%20or%20their%20performance.) when you hit that groove. _Flow state_ happens when you are completely focused on a single task and nothing else distracts you. One thing you can do is try a version of the Pomodoro technique where you won’t respond to any messages or calls until your time is up. Unless production is on fire, most questions or tasks can wait 30 minutes or longer.

Another technique is to separate your day into discrete blocks. If you know you’re more productive in the afternoons, block off a few hours then for the more difficult tasks. You can have a block of time in the morning just for meetings, impromptu pair-programming calls, and office hours when you’re available for general questions. Later in the afternoon, you can make time for PR reviews or other meetings. The main thing is to have hours of time set aside for one specific type of work so that you aren’t context switching every few minutes.

###### NOTE

I’ll admit that it took me a while to learn how important protecting my time is. I used to think that when I received a message, I needed to respond immediately. Or if a team member reached out with a question, I thought I should drop everything and help them as soon as I could. This is not a sustainable way of working, and it led me to severe burnout several times. Blocking off time on my calendar to focus on coding, meetings, and other tasks really helped me become more efficient, and it keeps me from burning out. Remember, every message or alert you receive is not a fire. So it’s OK to acknowledge someone’s request and let them know you’ll get back to them.

You can make a little game out of trying to combine contexts where you can. Maybe you’re working on a feature with another dev, and you can include pair programming in your focus blocks of time. Or you can switch up your work environment to help make context switches faster. That could mean changing rooms, moving your monitor to a different angle on your desk, or changing the type of music you are playing. There are ways you can condition yourself to get in the flow state for all the tasks you’re juggling.

Something that helps tremendously when you’re trying to figure out how to organize your day is to record what you currently do for a few weeks. There are [products that can help](https://gizmodo.com/how-to-find-out-which-apps-and-websites-youre-most-addi-1822667517) with that, such as [WakaTime](https://wakatime.com/) or even just a planner. You’ll find patterns in your work routine that you can use to your advantage and optimize your schedule. It can be surprising when you really analyze what tasks you focus on and when you do them. As you go through your day, take notes on what you worked on and for how long. Record each time you shift contexts and what you switched between. You’re creating your own personal time data that will help you determine what changes you should make.

If you don’t use a time management tool, this is the perfect time to start. When you have your tasks for the week laid out for you, it takes a bit of mental load off you because you aren’t worried about what you need to do every day. My favorite time management tool is a paper planner. Writing tasks down for the next two to four weeks helps me stay on top of everything from work to personal tasks. There’s also a little satisfaction in physically crossing things off the list. But there are digital tools like [Google Calendar](https://workspace.google.com/products/calendar/), [Todoist](https://todoist.com/), [OneNote](https://www.onenote.com/), [Evernote](https://evernote.com/), and [Obsidian](https://obsidian.md/) if you want something with more features.

A combination of these approaches will help you deal with context shifting more effectively and get more done with your time. Try out a few of them and feel free to use your creativity to come up with other techniques that work for you. Give each approach a few weeks to evaluate its effectiveness. Once you find what works for you, stick with it! You can experiment with new tools or different techniques from time to time, but you’ll have a point of reference to come back to if things get really busy.

## Keep Communication Open

One of the best things you can do is to foster a culture of communication for your team and others. If you see that a task is going to take longer than you thought, let the relevant people know as soon as you do so that there aren’t any surprises. If you run into something in the code where you need to have a discussion with the team, bring it up. When you speak up about anything you encounter, that normalizes it for the team to do the same. You might find that others have the same questions.

###### TIP

There are many environments, both professionally and culturally, where asking questions can be met with contempt even though no one knows the answer. Try to be the person who speaks up, no matter if you think it’s a “stupid” question or not. I can’t tell you how many times I’ve spent hours trying to find an answer for something I thought I should know just to find out that I’ve run into a larger-scale question that no one on my team can answer either.

It’s better to overcommunicate and have people ask you to stop than to sit quietly and struggle for long periods of time. It helps if you mention things you’ve tried or ideas you have for solutions to get the conversation started. Your thoughts might reveal areas of the product that need more consideration, or you might find that some of the devs have expertise in areas you didn’t know about. Always remember that no one knows everything, so don’t pressure yourself to become an expert in everything. When you run into inconsistencies in requirements or things that shouldn’t be allowed from a technical perspective, let Product and Design know.

# Conclusion

This chapter went over some of the things you can do from a project management angle. There’s always some overlap between your work and Product’s work, so this will help you keep the division more equal. Don’t be a passive listener in meetings because the things being discussed will directly affect your daily work. If it seems like Product has an overly ambitious roadmap and deadlines, point that out early. When you get tickets that don’t have well-defined acceptance criteria, ask the detailed questions to get it.

It’s your responsibility to manage your time efficiently, so do what you can to help yourself. This includes using time management tools, taking breaks, and understanding the tasks that you have to do to keep feature development moving forward. If you have a hard time focusing or planning, consider reading [Getting Things Done by David Allen](https://gettingthingsdone.com/what-is-gtd/). Reach out to the dev team to get help, suggestions, and feedback because everyone has different experience and they can all teach you something. This is how you grow into more leadership areas because you can be a multiplier for the team when you foster a better culture with your actions.