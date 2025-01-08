---
id: 01JGFNWPMFFKJ5K085PEWKQ2NK
modified: 2024-12-31T19:57:54-05:00
---
# Chapter 1. Kicking Off the Project

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 1st chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Welcome to the team! If you’re like most readers of this book, you’re a frontend or backend developer, but you would like to be a senior-level full stack developer. Now imagine fast-forwarding to that point in your career: you’re a new, senior, full stack JavaScript developer. That means you’re responsible for helping drive the technical direction of the project and coordinating efforts across different teams to get software to users. Your task is to start a _greenfield app_, a project you get to build from scratch with no existing code, based on designs and conversations with a number of teams.

Let’s break down what you’ll learn in this chapter:

- How to work with several other teams to fully define features
    
- How to take a design and break it into small, actionable tasks
    
- How to determine the data you’ll need to work with
    

# The Project You’ll Build

The project you’ll be building as you work through the book will be an ecommerce platform that has access to a lot of user data, including some personally identifiable information (PII), and that connects with multiple third-party services. You and your team have been tasked with creating a user dashboard so that customers can make purchases, see information about their order history and other actions they have taken, and interact with their digital purchases, like ebooks. They’ll also be able to take different actions based on their permissions within the app.

You’re starting with a fresh Product team and a Design team that you’ll work with to make this dashboard a reality. The Product team will be responsible for talking with different stakeholders to decide what features should be built. The Design team will be responsible for making the UI for those features and doing the first pass at the user experience.

The first part of the project will likely involve the Product and Design teams working closely together to make some mock screens for you and some behavioral docs. These mocks will be shown to you and discussed in a project kickoff meeting.

# Project Kickoff Meeting

The kickoff meeting typically involves the Product team going through the mocks the Design team has made. It’s very likely that these mocks will change as you go through development, but they should be about 80% of the way there. They’ll take you through the user flow, show you how users interact with the app, and give you an idea of the data you need. At this point in the project, get comfortable asking a lot of questions. Usually, you’ll think of things when you start looking at the designs from a technical perspective that the Product or Design teams haven’t considered.

###### TIP

Make sure that in this initial meeting you do some introductions so that everyone knows who they’re working with. It’s easy to dive into the work immediately, but remember that it’s just as important to get to know your coworkers.

Remember, this whole process is very collaborative. Asking questions early and often will make things go more smoothly as the project moves forward. Right now, only two screens need to be built for a proof of concept, although Product has told you that the functionality will expand over time. Let’s take a look at these screens and go through the user flow.

The screenshots are from a commonly used design tool called [Figma](https://www.figma.com/). There’s usually a design somewhere that visually describes how the app flows. As you move through the engineering process, you’ll focus on one of these views at a time, like the user info screen in [Figure 1-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#user_info_screen_design_in_figma).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0101.png)

###### Figure 1-1. User info screen design in Figma

[Figure 1-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#user_actions_screen_design_in_figma) shows a screen where users can see their information, such as orders, suggested products, and other details. This is also where users edit their information, such as their address, and where they can cancel orders and see their digital purchases.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0102.png)

###### Figure 1-2. User actions screen design in Figma

The design in [Figure 1-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#app_designs_with_mobile_view_in_figma) shows how these views should look on mobile devices. Most projects that interact with external users will have mobile designs. If you don’t see these designs, make sure you ask about them!

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0103.png)

###### Figure 1-3. App designs with mobile view in Figma

A lot of designers use Figma when they create the mocks for an app or a particular feature. Some other tools you might run into include [InVision](https://www.invisionapp.com/product/prototype) or the newer alternative, [Penpot](https://penpot.app/). The Figma designs also have the HTML and CSS in them, so they can be useful for developers when they want to check dimensions while arranging elements on the page. Although the CSS is there, it’s not always completely accurate. You can use them as a starting point, but make sure to modify the styles to meet the requirements.

By the time you see these mock designs, Product and Design will have had several meetings to iron out as much detail as they can. That doesn’t mean they’ll have everything figured out, and that’s where you come in. Based on these designs and the explanations of how a user will interact with the app, you have to work with your team to decide the technical approach you want to take.

Let’s walk through these designs, like how you will with your Product team. On the first screen ([Figure 1-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#user_info_screen_design_in_figma)), there are:

- A navigation bar on the side of the page with links to other areas in the app
    
- A search bar at the top of the page to help find products
    
- A featured products section, along with the user’s most recent order
    
- A sortable, paginated table with several columns that displays all the user’s orders
    

Several sections on the second screen ([Figure 1-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#user_actions_screen_design_in_figma)) have fields that the user can interact with. Users can toggle a few settings, update fields, and take actions based on their purchases. This screen has more complexity than the first screen, so you’ll want to take the time to really go through this with the team. Product and Design will often turn to you as the senior developer on the team for the final say on technical decisions.

###### NOTE

Never make promises without talking to your team first, no matter how hard you get pushed. The Product team may pull you into a meeting to try to get an idea of how complicated it will be to implement a feature they have in mind. This can lead to them expecting a quick commitment on a timeline, which is something you’ll have to get comfortable handling as you become a more senior-level dev. You have to firmly state that you can’t give an accurate estimate until you talk to the rest of the team. The other devs will be able to bring up concerns that you might not realize, and they’ll be working on the feature eventually anyway. Stand your ground and simply don’t commit if the Product team keeps pressing for a date. That way, _you’re_ not the one who put the team on a tight deadline.

With these designs in hand and a few documents on how the screens are supposed to work, it’s time for you to develop an understanding of the business logic before you go to the team. Having a solid understanding of the business decisions behind the designs is a huge help when you’re trying to explain them to other developers.

As you go through this information, you’ll need to consider several things based on the designs you have.

# Design Considerations

At this stage, you’ll find yourself asking a lot of questions, and that’s totally normal. Now is the time to really dive into the details and get nitpicky. One of the ways you can gain more experience with evaluating designs for technical purposes is by getting exposure to different design sets. For example, the Design team you’re working with is likely doing design work for other apps in the organization.

If you have questions about how a dropdown is supposed to behave, ask if there are other designs you can use as a reference. Often there are, and you should take the time to thoroughly understand how they work. Some of the best questions come from experience with different implementations. Another way to get more exposure to different designs is to start paying close attention to the apps you use. Have you ever _really_ thought about the user flow for your bank app or an exercise or meditation app? These are all examples of product designs in production that you can make comparisons with.

###### TIP

It can be a little tricky finding time to do exploratory work. One way is by writing a ticket that details what you’ll be researching, some of the thoughts you’ve had about the issue so far, and some open questions. Go ahead and add points to the ticket, too, to make it more official.

Think about how designs look for other products and how interfaces behave when you’re breaking them down into the details. As you think of differences, take notes about the questions they bring up.

###### WARNING

A word of caution for when you’re compiling your list of questions: as you think of a question, look through the documentation you were given. Sometimes the answers are already there, and it saves everyone time when you read the docs first. Take about an hour to go through any engineering docs the team has for other projects. Also read through the docs on third-party services that are being used as well as for any packages you’re thinking about using.

You might be wondering what some good questions are. That always depends on the designs and the business logic. Here are some questions that can get you started as you look at the screens you have available for this example project:

- Are there translations to other languages that you need to account for?
    
- What are the brand guidelines, such as colors, fonts, and responsive sizes, that should be used throughout the app?
    
- What permissions do users need to see their information?
    
- What permissions do users need to take actions in the app?
    
- How many different types of users will there be?
    
- How long do we expect users to be active, so we know how long an access token should live?
    
- What is all of the data we expect to handle for users?
    
- Do we want to show this data for different time ranges?
    
- Should the table be sortable by each column or only specific columns?
    
- How do we know which orders should be featured?
    
- What type of data do the input fields accept?
    
- When a user saves changes, what should we show them?
    
- If there’s an API error on the client side, what should the user see or be able to do?
    
- Does the user need to enter more credentials before taking sensitive actions like changing PII?
    
- Should input errors be shown inline with the field or by the Submit button?
    
- Do we need to consider tablet-sized devices?
    
- Will the backend of this app be used by any other apps?
    
- Will the frontend need to be integrated into another app?
    
- Are there any compliances or regulations this app falls under or will get audited by?
    

You can see that this list of questions can get very long, and I’m sure you thought of other questions as well. It’s OK if the list is long. It’ll be hard to get all these questions addressed in one go, so you should definitely prioritize this list based on the feature. It’s better to ask as many of these questions as you can in the beginning compared to halfway to the deadline. If there are small things, then you can probably use your experience to make judgment calls, such as with form handling.

The questions can be all over the place, but the Product or Design team should be able to answer them. There isn’t anything technical in here yet, although the answers to all these questions will drive technical decisions. Getting answers to these sorts of questions early is why senior developers tend to deeply understand the business logic.

To make the best technical decisions you can, you have to understand the purpose of the app’s functionality and how it fits in the overall business. This can give you some foresight into how to build the app in the most maintainable way because you see the direction the business is going. Some of this foresight will come from the data you know you’re going to work with.

# Data-Driven Design

Data drives design. Everything the designs do is built around how to best deliver data and let users interact with it. Take the table screen, for example. The data doesn’t have to be displayed as a table. It could be a collapsible list or a graph. That’s why you have to dig into these questions about the business logic and what you will be working with. It’s one of the reasons that frontend development is usually dependent on backend development. We’ll talk about this more in [Chapter 2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch02.html#setting_up_the_backend).

As you go through the designs, it’s a good idea to take notes on the data that you see you’ll need for each screen. On the settings screen, you can see that you’ll need the user’s name, shipping address, and language preference in one section. Take notes on potential variable names and data types for these that you feel comfortable sharing with the team. That can help spark the initial conversations around how to approach the project.

Also confirm with the Product team that these values are correct and what’s expected by the app. Always check with the Product team that you have the correct data and relationships between the data. They won’t explain it in technical terms, but they’ll walk you through different scenarios so that you can paint the big picture.

This is like the discovery phase of the project for your development team. Once you’ve gone through the designs and you have your questions and notes, get ready to talk about specific features.

## Breaking Down Designs into Tasks

You’ve made it through the first round of refinement for the project! In this next round, you can assume that Product has answered all your team’s questions and you have updated designs. Now you’ll be working with Product to carve features out of this info. This is when you’ll first use a ticket system, such as [Jira](https://www.atlassian.com/software/jira), [Trello](https://trello.com/), [ClickUp](https://clickup.com/), or [Shortcut](https://www.shortcut.com/). [Figure 1-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#example_of_a_backlog_in_trello) is an example of a ticket system in Trello.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0104.png)

###### Figure 1-4. Example of a backlog in Trello

The tickets you make here are going to go through refinement once you start bringing in more developers and other teams. The goal at this stage is to make action items for parts of the designs. The user info screen, for example, has several pieces that can be broken out into features, such as the featured section and the table. You’re trying to get people thinking about how the designs will actually work, and you’ll start bringing in that technical perspective.

You might try to group features based on data relationships. This will help make it easier for backend changes to be made and deployed as the frontend is being built. This is also a stage when you can still point out any big issues you run into as you start thinking through the tech stack. Any time you have a thought for a feature or technical requirements, make sure a ticket gets made. If there isn’t a ticket, the work won’t get prioritized.

## Refining Tasks from Feature Requirements

You’re at the point in the project where Product should have some user stories written for you to refine with technical details. These user stories are part of the documentation you get that define the features you’re developing. They are typically written from the perspective of the user to give you some of the hidden context behind how the user will interact with the functionality. This doesn’t happen all the time, though. Often, you’ll be working with Product to fully define features, but they’ll have some details to get started with from the previous round.

You can assume that your team follows a normal Agile process with two-week sprints and the standard meetings like daily stand-up, backlog grooming, sprint planning, and retro. At this point, start going through the tickets in the Jira backlog in a backlog grooming meeting. This meeting should include the development, quality assurance (QA), Design, and Product teams so that everyone can ask or answer questions and bring up concerns. Feature tickets should be broken down into the smallest pieces of work a developer can do.

This is one of the times when you have to use some art versus science. The “smallest” piece could mean having a ticket to write an endpoint with all the tasks associated with it and having a separate ticket for triggering events. One way to think about it is from a QA testing perspective. What is the smallest grouping of work that makes sense to test together? Entire books and numerous resources have been written about Agile and these kinds of project decisions. One resource you might look at are the [Atlassian articles on Agile](https://www.atlassian.com/agile).

To keep expectations aligned with what the team can output, estimates are usually made on every ticket for how complex the task is. When you’re going through tickets during backlog grooming, don’t hesitate to slow things down. If you spend the entire time on one ticket, then that shows Product where they need to improve, which can be brought up in the retro meeting.

Make sure every ticket has acceptance criteria agreed on by Product, QA, and your team. If the ticket has UI elements, make sure you have access to the designs for both desktop and mobile. Every ticket should also have background information on how the functionality fits in the overall app so that the developers have context. All the developers should be in agreement on estimates for each ticket.

This might seem like a lot of fuss over tickets, but doing all of this up front saves your team time. Getting everyone on the same page at the very beginning helps keep your team unblocked. With this level of documentation for each ticket, there shouldn’t be much ambiguity in the task when anyone on the project gets to it. Here’s an example of info you will find in a well-defined ticket:

# TITLE: [BACKEND, QA] CREATE ENDPOINT FOR TABLE DATA ON USER INFO SCREEN

Description: As a user, I need to see a table with my purchase info. This info should include the product name, price, estimated arrival date, and quantity in each row on the table. This table should also be sortable by clicking on the column headers. It should have pagination so that I can go through my purchase history for the previous 12 months. It should match the designs attached. (See [Figure 1-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#user_info_screen_design_in_figma).)

Acceptance criteria:

- Update data schema to include purchase history definition: product name, price, estimated arrival date, quantity
    
- Run migration on database
    
- Create endpoint that responds with user purchase history for past 12 months
    
- Ensure authentication is working correctly
    
- Implement validation on request data
    
- Write tests for new endpoint
    
- Write docs for how the endpoint works so frontend can use it
    

Points: 5

It should be very clear to any engineer what needs to be done, although the ticket shouldn’t tell them how to implement it. That’s something they can decide after the team has discussed it. Having tickets written out like this will spark deeper technical discussions that will help drive the overall architecture of your project.

This is when you start to take the tasks from Product and refine them even more as a development team. It’s always a good idea to include any highlights that come out of team meetings in the tickets, so everyone remembers why things were done a certain way. You might learn that one of your third-party services only accepts data in a certain format, which leads to you needing to write some utility functions. But you absolutely have to have this because of a product requirement. That will be important context to have for code review and testing.

It may seem like this is a long process, but usually you can define the tickets for the upcoming sprint in a few days. Then once the sprint starts, you can jump in. But before you get to pick your own tickets and start coding, there are a few more conversations you’ll need to have.

# Discussing Timelines with Product and Other Teams

At this point in the project, you can assume you have well-defined tickets and the details are as ironed out as possible. Now it’s time for one of the higher-level conversations. After all the tickets have been estimated, Product will be ready to set a launch date.

###### NOTE

Sometimes Product will come to you with a launch date before you’ve had a chance to estimate tickets. If you find that their timeline is shorter than what Engineering needs to adequately get through the tickets, push back. Always speak up when timelines are tight. This will save everyone from unnecessary stress.

This discussion is going to involve more than just your team. You might need to coordinate with developers on other projects, the DevOps team, the Support team, and the QA team. It just depends on how far the reach of your changes goes. Don’t commit to a launch date until you’ve talked with these other teams. You’ll want to see how your goals fit with theirs and what you can do to make the process easier for them.

## Talking to Other Development Teams

Your project might depend on another team implementing something in their code that your app consumes. You’ll need to give them plenty of heads-up and details so that they can include this in their sprint. Development teams aren’t always aware of when feature releases for one app cross over into their app. Talk to them about what you need and keep it as simple as possible. Mention what your target launch date is and see if that’s something they can help you reach.

On the other hand, the new features in your app might cause breaking changes for other teams, and you need to let them know what updates they need to make and by when. Don’t hesitate to reach out to any teams you think will be affected just to double-check. As you find out which teams are affected, update the team docs to show them as consumers of your app.

Here are some things you’ll need to have ready before you talk to them:

Well-defined technical requirements

Give them a task that’s just as defined as if your team were doing it. Also give them time to ask questions if they need more information.

A general idea of where in their code the changes need to be made

You don’t have to go in their code and implement the changes, but having a clue of where to get started helps.

Context behind the change you’re requesting

A change that seems small to you could change the functionality of something important in their app. Make sure they understand why this change is needed.

A deadline

Of course, you’ll ask how long it’ll take them to get to it, but have a flexible date in mind.

Willingness to do the work

If it would be faster for you to implement the change and the team to review it, be open to that.

When you know you have everyone across the development teams on the same page, then you can talk to the next set of people. These discussions don’t need to happen in this order, but they do need to happen. Whoever is available to talk first can be the first conversation you have.

## Coordinating with DevOps

When it’s time to release to different environments, you’ll need to check in with the DevOps team. That’s assuming there _is_ a DevOps team. Sometimes at smaller companies, you will also be responsible for handling deployments. This book assumes that you’re working with at least one DevOps engineer who handles infrastructure and pipeline changes for you.

Since this will be a completely new app, you’ll need the infrastructure to be set up for QA to do testing, for production, and for any other environments. You may be able to help set up the continuous integration and continuous deployment (CI/CD) pipeline using tools like [CircleCI](https://circleci.com/), [Jenkins](https://www.jenkins.io/), or [GitHub Actions](https://docs.github.com/en/actions), although when it comes to provisioning resources and handling infrastructure concerns, that will be completely up to the DevOps team.

The DevOps team will provide important information, such as which cloud provider the app will be hosted on. You’ll give them important information, like the programming language and frameworks being used. The goal for both teams on a new project is to have at least a couple of environments up for testing before prepping for production releases. Getting DevOps involved early is going to make things go more smoothly as development progresses.

Remember, all these teams that you’re interacting with already have their own priorities. So you have to take into account that they may not get to you right away. That’s why you want to coordinate with DevOps as soon as you have a technical direction. The last thing you want is to put DevOps in a crunched position right before an important deadline. Be ready to jump into a call to go over the project with them so that they understand what performance and usage are expected from this app. During your call, there are several things you’ll want to cover:

Work on developing _playbooks_—the strategies and steps you implement to resolve issues—for outages and other issues in production

When an issue happens, it can be hard to figure out the best steps. That’s why you and DevOps need to think about this ahead of time.

Determine the right number of environments for the app

Some projects can have three environments while others have an environment for every Git branch in existence. Find the right balance of environments that is maintainable on the infrastructure side and flexible for testing and other uses.

Figure out the best deployment strategy

It’s important for DevOps to know when they need to trigger scripts to run and how often they will run. Some parts will be automated, and others will need to be run manually. The best strategy is one that works for both teams.

Decide how to handle CI/CD maintenance

Most CI/CD tools use YAML files to define the configurations for a pipeline, and these config files live in the project’s repo. Sometimes developers make the changes, sometimes DevOps makes the changes, or both can feel comfortable making changes. Just get everyone on the same page.

Agree on the versions of the JavaScript runtime, like Node, Deno, or Bun, that will be supported

Servers aren’t always running the latest version of your runtime, and that can cause weird bugs in your app. Know the oldest version of the runtime your project will work with, just in case.

Find out when credentials are cycled

Some apps are dependent on keys and secrets that are maintained as part of the infrastructure, such as npm tokens. Add a reminder ticket to the backlog so that you can plan to update them before they expire and cause production issues.

Choose a location to store assets

For some apps, this will be included as part of the cloud provider, and for others, a third-party service will be used. Even if you don’t come to an absolute answer right now, make another ticket to come back to this.

Set up time to test the app together before deploying

You know how the app should behave, so DevOps will lean on you to handle some of the environment testing. They can usually get the app running, but you have to verify it’s in the correct state.

Reserve some time for after the initial app deployment

Things always come up with the first few deploys that need to be fixed. Go ahead and put that meeting on the calendar. If nothing happens, you just reserved celebration time for everyone.

One thing to remember while you’re talking to DevOps is that you won’t understand everything they’re talking about and you don’t need to. You’ll start to pick up on things as you work with them, and it wouldn’t hurt to pair with them a few times to see what they do. If you don’t want to dive into the DevOps world, that’s fine. They’ll handle everything on their side from here.

Now you need to talk with the QA team.

## Working with QA

While developers, especially seniors, are expected to thoroughly check their work, sometimes bugs slip through. That’s why QA teams exist: to make sure those bugs don’t reach production. Weird integration issues happen, pull requests (PRs) overwrite functionality, edge cases are found, and a number of things cause bugs that developers don’t catch. Getting new features and bug fixes to QA in a timely manner is important for staying on time for releases. You need to get the new code to them, give them time to test and report back defects, and then give yourself time to fix the defects and send the code back to QA.

###### NOTE

There is a chance that you might not work with a QA team at all. It really depends on the company. In some early-stage startups, you won’t have a dedicated QA team. It’ll just be you and the other developers, maybe even the CEO. In other companies, you might see the Product team do some manual QA on features. Regardless of whether there’s a QA team, you can use the steps in this section to help do your own QA more thoroughly.

You see, this loop can take quite a while. To shorten that time and make sure the highest-priority tickets are tested first, include QA in many of your development calls. They will have questions about features and bugs from a QA perspective that need to be understood. All the test cases they write are basically like the history of the app. Many features will have test coverage, whether it’s automated or manual. These tests will exist even when the feature doesn’t, so QA keeps a record of changes in a different way than other teams do.

Set up a call between the QA team and your dev team, and make sure you’ve discussed:

Automated integration tests

This is a responsibility that’s usually owned by QA, but since the tests live in the project’s repo in many cases, developers can write these tests as well. Also make the distinction between integration tests and unit tests. Developers should always be the ones writing unit tests.

Level of detail in reproduction steps

There’s a balance between not enough detail and way too much. Discuss what works best for the developers and the QA engineers so that everyone knows what to expect. You’ll end up working with a QA engineer to home in on the issue anyway, but it helps to have a starting point.

Including QA in relevant calls, such as backlog grooming and stand-ups

This ensures that QA will be up to date on any upcoming deadlines, shifted priorities, or releases and that they can give updates if they’re running into issues. Try to put any updates for them at the beginning of these calls, such as tickets they can test, so they can use their time best.

Make it clear that QA should feel comfortable reaching out to the developer who worked on a ticket they’re testing. Developers should be willing to pair with QA to help them determine if an issue is a bug or something else, like an environment difference.

QA is everybody’s best friend because they find a lot of bugs that would have caused issues on production. It’s not good for anyone when there are issues on production. QA is not there to tell the developer they’re wrong or that their code is bad. They just want to make sure that absolutely nothing makes us bring out those playbooks for production fires. Be patient with them as they report defects because they’re just doing their job.

You’ll interact with a number of teams once you’re ready to deploy changes; that will really depend on the company. You might interact with the Sales team to help them understand what the product is capable of so that they can set realistic customer expectations.

There’s also the integration or customer success team. These teams usually work with customers one-on-one to help them integrate the product into their systems. They definitely need to be aware of any breaking changes or new features.

Some companies have a dedicated Security team as well, especially if the product is in fintech, health care, ecommerce, or manufacturing. This team may come in right after QA to run security tests, and continuous security scans might also be happening on the app throughout your development pipeline.

One team that will be around in some form or another is the Support team. This is the next and probably the last team you need to talk with.

## Planning with Support

Support are the people who are in direct contact with users who are experiencing problems. Some of the bugs you will fix come directly from Support. They might even link their support ticket to your bug ticket so that they can keep users updated. The Support team is also a source of inspiration for new features. If they keep getting the same requests from different users, it’s worth looking into a solution.

Support does a difficult job of dealing with upset customers. You want to make sure that happens as little as possible. When you’re talking with Support about an upcoming feature release, they need to know how it affects users. Product should have already discussed timelines with them. You’re trying to see what they need from the development team.

Keep in mind that Support team members usually don’t have highly technical backgrounds. They understand how the app works from a user perspective and as an app admin, but they don’t necessarily understand the weeds of the technical details. Knowing where they are coming from will help your communications exponentially. Your job with Support is to translate the technical stuff into the information they need.

Here are some things you can do that will help:

Have documentation Support can refer to for using a feature

Describe anything that has changed compared to existing functionality. Highlight values users will need to enter since user input causes many of the issues they receive.

Listen when Support brings up complaints with the app

Take in the feedback they give you from bugs or other requests and work with Product to prioritize that. This doesn’t mean you implement everything that Support asks for, but it does mean you do a little research into why they’re asking about it and how hard it would be to fix.

Always sync up on release dates and times

Support will sometimes set aside special time around releases because there are going to be more user questions. They need to know exactly when changes are going out so that they aren’t blindsided with an increase in issues.

Do a demo of the finished feature right before release

After the changes have been approved for release, do a quick live demo with Support (and really everybody else). That way, they can see how the app is supposed to work and ask any questions.

Dedicate extra developer time to Support around release days

If Support starts getting overwhelmed with issues, you might need to roll back the changes. Just be ready to pay close attention to them.

Discussing these things with Support is another way to bring up issues that can be addressed early in the process. It’s also a great way to build a relationship between the Engineering and Support teams because you work closer together than it seems. This is the last team you need to coordinate with. Now it’s time to bring back all your findings and notes to make a cohesive plan.

# Conclusion

You’ve made it to the final step of project kickoff. You have your notes from the different team meetings, and it’s time to distill it all and give the highlights to Product. All these conversations will help you come up with a reasonable deadline.

This is when some negotiations will start. One thing to keep in mind with deadlines is that something in development will go wrong. Give yourself and your team room to breathe and possibly overdeliver. Remember, underpromise and overdeliver. Unless the feature you need to implement is time sensitive, such as a third-party service being updated, add a few days of padding to account for weird happenings. After you’ve settled on a date with Product, taking all of the other teams’ concerns into account, summon everyone together for one final, short call.

###### NOTE

It’s important to understand that setting deadlines is extremely tricky in software development. Any number of things can pop up at any time and derail everything. If someone gets sick or a natural disaster takes out someone’s electricity for a week, some tickets aren’t going to get done. You might get deep into development and realize that a tool you’re trying to integrate with is way more complex than you accounted for. You might be waiting to receive work from another dev team and they fall off schedule, putting you behind as well.

All of that is OK. The best thing you can do is communicate these issues with everyone as soon as you find out about them. Then you can start making plans for what to do next. Life happens, and you have to try your best not to let this stress you out every time—because it will happen a lot!

This call will have someone from all the teams you’ve talked to. The only things you want to confirm at this point are everyone’s action items and deadlines. Having this call is about establishing accountability and triple-checking that everyone is on the same page. As prep for this meeting, take all this info out of your notes and make a short doc that everyone can reference leading up to the release.

At this point, you have many of the tickets you’ll be working on and they’re well defined. You’ve documented the cross-team connections. Project kickoff is complete. You have everything you need to finally get started writing code. In the next part of this book, you’ll build a scalable backend in TypeScript using the NestJS framework.