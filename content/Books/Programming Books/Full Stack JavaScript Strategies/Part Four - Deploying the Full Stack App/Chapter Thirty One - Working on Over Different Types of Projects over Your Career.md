---
id: 01JH1P31DJPTAFY78VJNTB8ZAH
modified: 2025-01-07T19:47:42-05:00
---
# Chapter 31. Working on Different Types of Projects over Your Career

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 31st chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Throughout this book, you’ve built a greenfield product, which means you built it from scratch and made all the initial decisions on how the code should be developed. While you’ll get a chance to work on projects like these, it’s a less common scenario. Often there is an existing codebase with established practices, and there are core functionalities that can’t be easily changed.

As you go to different organizations over the course of your career, you’ll pick up a lot of new skills and patterns from existing projects. You’ll join teams of other senior devs and learn new paradigms and methods for building apps. When you get a chance to do something similar to what you’ve done throughout this book, you’ll go in with even more experience. A greenfield project will stretch you in different ways after you’ve worked on legacy projects for a while.

In this chapter, I’ll go over:

- Considerations for brand-new apps
    
- Considerations for existing apps
    
- How all of this affects your career
    

This is where you can explore your experience and how it differs between greenfield and legacy projects. I’ll give you some key things to look for depending on what type of project you’re working on.

# Considerations for Brand-New Apps

You have to make a lot of decisions early that will determine the future maintainability of a greenfield codebase. You’ll be able to refactor things and swap out components and packages for better ones in the future, but having a well-defined base is going to make this a smoother process. While you’ll spend a lot of time and energy creating diagrams, writing docs, and developing processes in the beginning, it’s effort well spent as the codebase evolves over the years and you aren’t the only dev working on it.

###### NOTE

Remember that you won’t go in having thought out every possible scenario. You will discover some things as you go and get feedback on what you’ve built so far. But you can set up a strong foundation to help spark those conversations earlier rather than later. There’s a balance you and the Product team will have to find between preplanning and actually building something.

Working with a greenfield app is your chance to set up the codebase in a way you wish other projects you’ve worked on had been initialized. With all your knowledge, you’ve experienced some of the pitfalls and seen mistakes made in other organizations. Whether it’s for a startup, a mid-sized organization, or an enterprise-level organization, a brand-new app has a lot of the same considerations. You’ll need to consider the tools the organization favors as the starting point, and then you can start bringing in your experiences. Something that will help is having your own checklist that you’ve developed over the years.

I’ll walk you through the things I do on every greenfield full stack app. This can be used as a lightweight checklist to get you started. Each of these points in the checklist involves many details that we’ve covered in the previous chapters of this book.

## Understand the Problem You’re Trying to Solve

Before you even set up a new repo, you need to understand the problem that the software you’re building is trying to solve, like we did in [Chapter 1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch01.html#kicking_off_the_project). This will involve numerous calls with the Product team and senior leadership so that you can nail down the functional needs. You’ll even need to talk to higher-level business colleagues to understand more about the business domain and how they foresee the software helping in that area, what the weaknesses of the software may be, and their plans for how the product solves a domain need. Then when you talk to Product, you’ll have a clearer understanding of the roadmap.

At this point, you’re trying to look at the roadmap and make technical decisions that will grow as the app grows. When you get a firm understanding of the problem, you can start choosing tools and services that will work best and require the least amount of refactoring years from now. It may help to create a software bill of materials (SBOM) to keep track of which tools solve which technical problems, their security implications, and how well they integrate with one another.

# GENERATING A SIMPLE SBOM WITH AUDITJS

There are a [number of tools](https://owasp.org/www-project-cyclonedx/) you can use to generate SBOMs of different detail levels, but a quick one to use in your repos is [AuditJS](https://github.com/sonatype-nexus-community/auditjs). It will generate a list of all the dependencies in your repo, the current versions, and the vulnerabilities they have. [Figure 31-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch31.html#sbom_example) is an example of what that will look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_3101.png)

###### Figure 31-1. SBOM example

## Build the Data Schema

In [Chapter 3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#building_the_data_schema), you saw that deciding how to model the data is an important step because it will affect the tools you choose and the way the app can grow in the future. Some standard things you want to build into your schema, whether you use a SQL or noSQL database, are audit values such as a “created at” date, an “updated at” date, and a user ID for who made the update. Many ORM frameworks add these fields automatically. These are applicable to any table you create. You should also consider relationships between your data so that you can decide on the most performant way to store and reference values.

This is where tools like [dbdiagram.io](https://dbdiagram.io/home) or [Miro](https://miro.com/) will help you organize and present your ideas for the data schema, data types, and relationships like you did in [Chapter 3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#building_the_data_schema). Take your time and refer to the roadmap as you come up with your initial ideas for the data schema. Keep expandability in mind because this is going to need to scale over time, and it may eventually be integrated with other systems, such as a data warehouse or a data pipeline, to get insights directly from the tables. Keep naming conventions in mind as well because that will help clearly and accurately describe what data you’re storing.

## Decide on an Architecture

The system architecture is a huge decision because it’s what the dev team will be locked into over time. That’s why this was covered throughout the book, in [Chapter 5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch05.html#third_party_services), [Chapter 6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch06.html#background_jobs), [Chapter 10](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch10.html#backend_performance), and [Chapter 23](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch23.html#full_stack_deployment_setup). It’s important for you to pick a simple but flexible architecture to get started with. Your options include any combination of backend monoliths, microservices, frontend monoliths, and micro frontends. Based on what you know about the business domain and the current roadmap, you can make some guesses for how the app will grow and let that drive your architecture. Make sure to create as many diagrams as you need to show how everything connects and then keep them up to date.

It’s a tedious task to diagram the architecture, but including everything in this initial process will help you catch potential pitfalls early. Try to build out a full diagram and then make iterations to refine it. Include your background jobs, workers, events, endpoint connections to the frontend, and any third-party services you can think of. You don’t have to have everything for the app figured out at this stage, but a good estimate will help give you and the team a strong starting point as more decisions come up.

## Pick Your Cloud Provider

Your cloud provider is going to determine everything from the services you have available to the bill the organization has to pay every month. Do some cost analysis for three or four cloud providers and compare the costs to the services you need. Be realistic about your user base and how it’s projected to grow. Look into other services that integrate into the cloud platforms. Creating a spreadsheet with your needs and doing a comparison between platforms will help you and the organization make a cost-effective selection.

You also need to consider what skills devs have or would be interested in developing because building a team requires people who can work with the tools you choose. Bring security considerations into this evaluation, especially if you’re working in a heavily regulated domain. Think about how hard it would be to migrate to a different cloud provider if something were to happen. If you can, test out the services in a few cloud providers to determine how easy it is to find help with issues and how configuration will work.

## Build the Backend

This is when you start implementing the architecture you’ve designed. You can begin choosing the tools you want to use in the codebase to make development smoother, like you did in [Chapter 2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch02.html#setting_up_the_backend). It helps to set up some scaffolding to have boilerplate folders and file structures that the team can get started with based on your architecture and data schema so that you have an idea of how you want to start adding features.

Connect the app to the database and create your seed script, like you did in [Chapter 3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#building_the_data_schema). It will also help you to write an endpoint or two just to have something to test with. You can start with smaller features, such as an endpoint that gets an order from the database and one that updates a customer profile. The goal is to get something working so that you know you have all the key connections, permissions, and environment variables in place before the dev team starts focusing on the code.

## Build the Frontend

Similar to the backend, you’re going to implement your frontend architecture and spec out the app structure you want to go with. If you’ve decided on a monolithic frontend, you can start with the folder structure so that you organize components, utility methods, and API calls early. Or if you go with a micro-frontend architecture, as we discussed in [Chapter 13](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch13.html#setting_up_the_frontend), you can start making the different repos that will become the components for the overall frontend. This is where you want to choose frontend tools and decide how you call endpoints and how you handle styles.

Try to get a main container component and a smaller component created to test how the app runs. Then make an API request from the smaller component to make sure you can connect to the backend without problems. Check for some initial responsiveness with your designs and start scaffolding some of the shared functionality. Adding some tests at this phase will also set the foundation for a level of code coverage that the dev team agrees on.

## Integrate the Backend and Frontend

After you’ve done the initial checks with the frontend setup, you and the team will start adding more features. Even with all the planning, documentation, and diagramming, there will still be weird connection issues, such as what we addressed in [Chapter 23](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch23.html#full_stack_deployment_setup). You’ll run into things like the API not having permissions enabled for the frontend, names and types not matching across the frontend and backend, and [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) issues. This is likely when you’ll start seeing areas of both sides that can be refactored.

###### WARNING

A [huge, very public incident](https://www.usnews.com/education/best-colleges/paying-for-college/articles/another-fafsa-delay-alarms-students-parents-colleges-higher-ed-advocates) happened when the teams working on the FAFSA form didn’t do these checks. There’s only so much you can develop in isolation, and even then you should be talking to one another.

At this point, you’re getting a full view of how the app will grow and the considerations the team needs to make going forward. Reference and update your architecture diagram as you find complex areas and new relationships between the frontend and backend. This is when you can start making more strict decisions and conventions for the project going forward. You can use this as a chance to refine your PR review and deployment processes, too.

## Set Up Your CI/CD Pipeline

In [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline), you built a simple CI/CD pipeline. This involves setting up any environments you want to deploy the frontend and backend to and figuring out where to store secrets and credentials. This will likely fall on the DevOps team to handle, but you’re expected to help them check that the app is deployed and working as expected. That means you’ll look at which endpoints are being called to make sure the environments are connected on the frontend and backend correctly. You’ll do data checks to make sure data is coming from the right environment.

Check your cloud services to ensure that assets are in the correct locations and are configured with appropriate access levels. Set up any automations for your branches that will help get your changes out consistently. This is a good time to check the bundle size of your build to see if there’s room for optimization. If you’re going to run your apps in containers, check that the containers are configured for your environments.

## Perform QA Testing

Work with the QA team to fully test the app and write test cases to cover all the functionality required by Product as well as any edge cases you have found. Get Product involved and do demos as you build and deploy features to get some user acceptance testing. This is also a good time to discuss automated testing as a task that can be shared between QA and the dev team.

If there isn’t a dedicated QA team in the organization, you can perform testing based on the feature specs and designs you have. Check how the app works in different browsers, with different network speeds, and with multiple types of users. Force the app to throw errors and see how well it handles them. Look for discrepancies in values you get from the backend compared to what gets displayed. Check to see if data is being synced at the expected time and updating tables correctly.

## Check Your App in Production

The first time you deploy your app to production, there will be things you need to adjust. You might find that there are some configs or resources that need to be updated to match the load in production. Or you might realize that you only have user roles set up in other environments and you need to add them. This is a time to check how secure your app is by looking at network responses to see if any PII is leaked. See if you can force the app to give you access to resources that should be protected.

Look at things from a user’s perspective here because this is exactly what a user will interact with. As you fetch data from more third-party services, you’ll notice that there is certain data you can test only in production. Try to have test accounts and test users or a way to toggle between user roles available in prod to avoid using real user data. Check your logs to make sure they’re recording events and your alerts are working.

# Considerations for Existing Apps

Existing apps are sometimes called “legacy” apps. These are codebases that existed before you joined the team that originally created them, and they will make up the majority of the apps you’ll work on. Most organizations will need you to come in and add new features, maintain the codebase, and make improvements where you can. This is where things become less clear because not every team has adequate documentation or a good understanding of the problem they’ve solved. You can bring fresh ideas to something that everyone is used to working around.

## Get Access to the Services You Need

Any existing app will already be connected to the relevant services, so you’ll need access to all of them. This is something that gets overlooked until you realize you need to do something you can’t. It’s very common that this is how you end up determining what you need credentials for, but it’s not very efficient. Ask for access to all the services when you’re setting up the project for the first time and make sure you have access across all the environments.

Some of the services you’ll need to log into include the cloud platform, logging, storage, monitoring, and any other third-party services, such as Stripe or Twilio. If you don’t see these services and the steps for how to connect to them anywhere, update the onboarding docs to include them. It will be fresh in your mind, and you can document any tips that will make the process easier for the next dev.

## Get a Dev Instance Running

This will be one of the first things you do when you join the team. There will be tools you need to install, services you need to configure, and quirks you need to know about to get the app running locally. After you’ve pulled down the code and installed all the packages, you should see if there’s anything else in the app’s documentation that will help you get started.

Once you’ve gotten as far as you can with the docs, reach out to others on the team to help you with any further setup. As I mentioned before, updating or creating these docs will help onboard any devs joining the team after you, so this work will be greatly appreciated. There will be things that have been forgotten for the initial setup that you’ll bring up, such as local environment variables or commands that need to be run in a certain order. Make sure you have the frontend, backend, and database connected locally so that you can work every part of the app.

## Look at the App in Production

The best way to learn what the app does when users work with it is by using the app in production. Since you’ll be coming in with little context or knowledge of how things are supposed to work, this is a great time to ask the Product team about the product you’re jumping into. Make sure you’re working with test credentials in production because you should not use real user credentials and start clicking around on things.

You might reach out to the QA team here as well to get some of the test credentials they use for production. Taking a tour through the app in production is a quick way to find out if you have permission to access the functionality you’ll need to debug things in the future. This will help the dev team get you anything you need that was overlooked in the onboarding documentation. This is a good time to ask about where and how user data is stored so that you can check there when you need to debug.

## Look at the App in Nonproduction

At some point, you need to make sure you have access to the developer view of the app. This might be in staging, develop, other environments, or all of them. It’s another checkpoint to make sure you have the right user roles to access all the systems and functionality you need to get work done. This is where you might reach out to another dev and get them to walk you through the different environments and the release process. Take notes here because there will likely be a few steps you need to go through to switch between environments.

See if there are small tickets you can finish quickly to make sure your changes will be reflected across these environments and that you’re doing the release steps in the correct order. This is your chance to see how long it takes deployments to finish, the type of code coverage the app has, and any automated security checks that are in place. You should also see if there are any admin pages for internal users or if feature flags are used anywhere. This is an opportunity for you to learn about some of the hacks your team uses to test scenarios for Product validation or demos.

## Read Through the Code

This will help you understand the folder structure and the architecture of the codebase. When you start going through the code, work backward from something you’ve seen in the UI. This will guide you from the frontend all the way through the database and other parts of the infrastructure. Do this with a few features to guide yourself through the code in a structured way. Some devs try to just look at the folders and their contents to get a sense of what’s happening, but that doesn’t always give you context.

Try reverse engineering some features to learn about the implementation and patterns the team uses. This will give you more targeted questions to ask as well as an idea of the debugging process. You can also try correlating the code to what you read in the technical and product docs. That can help you understand relationships between engineering teams and the expectations for the separate codebases they work on.

You can also get a code walkthrough from someone else on the team. Then you’ll be able to ask questions in real time as you start to understand how things work. When you do get your first few tickets, consider pair programming with someone who has been on the team for a while. You might have an idea for how something should be implemented just to find out that it’s already done a certain way.

## Take Notes About Potential Refactors

As you become familiar with the code, you’ll see areas that you might have implemented differently. If you see parts of the code that can be improved for performance or if there are packages that can handle some of the more complex functionality, point that out. As you work in different codebases, you may find they all have a lot of functionality in common. In that case, you could advocate for a new shared codebase to move that common functionality to. That’s how internal UI libraries and SDKs are created.

When you notice things you would do differently, ask the team why they were implemented the current way. There could be a strong reason for using an older version of a package or a less popular programming pattern. You need the context for why decisions were made. So take notes on the things you would do differently and talk to the team about them. Once you have the context, then you can make your suggestions based on performance optimizations, DX improvements, or security enhancements.

You might come up with suggestions, such as a template codebase that all new projects can be created from to keep consistency between codebases. Or you could suggest new roles or permissions to streamline dev access across resources and tools. Adding more types to TypeScript projects is another place where you may find room for improvement. Something I’ve seen on projects that have consultant devs rotate on them is a need for a better folder structure and general code organization. It’s OK to be the leader on bigger initiatives like that early on as you start taking your first features.

## Ask Questions and Document the Answers

If you’re having a hard time understanding how a certain part of the app works, that means you’re not the only one. You’ll find these questions come up while you’re onboarding and getting your local environment set up to work on your first feature. When you have a question, start a doc and include the answers as you talk to team members about them. I’ve found that sometimes no one remembers how they set up, and it takes some looking through random Google Docs and people’s local environment variables to figure it out.

One of the best things you can do for an existing app is to add documentation around these unclear parts of the app. That includes technical docs, product docs, and anything else that may help the next dev who joins after you. Don’t hesitate to ask everyone questions throughout your onboarding time and after. You’ll learn how the organization works much faster when you reach out instead of waiting for someone to ask you.

## Improve the Code Quality

That means you do things like DRY (don’t repeat yourself) the code out when you see repetition and make shared utility functions and components. You should feel comfortable asking the team about patterns in the code and why they were selected. Look for ways you can simplify files and folder structures. Remember, this type of work can be done incrementally, so you don’t have to rewrite the app yourself. Use your review to open discussions with the team on best practices and approaches you’ve seen.

# DON’T GO OVERBOARD WITH DRY

Here’s a word of caution about DRY that came up from a discussion with Ethan Brown:

> I used to be an enthusiastic proponent of DRY, but I’ve since moderated my advocacy. I’ve found that in practice, DRY has its downsides, and they can be big:
> 
> 1. Adding layers of abstraction that may reduce the amount of code but at the expense of clarity: you can end up with code that ostensibly does something simple, but you have to step through half a dozen functions (for example) to understand what’s really happening. As well intentioned as this can be, sometimes repeating yourself really is the best choice.
> 
> 2. The time and effort spent creating a beautiful abstraction doesn’t pay for itself, either because you spend a lot of time abstracting something that’s only done a handful of times (and doesn’t grow over time), or you miss the mark of the parameterization and you make ostensibly abstract code that has to be tweaked constantly for future use cases.
> 
> 3. Not observing patterns for a sufficient amount of time to understand the most appropriate abstraction. I’m absolutely guilty of this…sometimes I see this shining cathedral of abstraction in my mind and I get excited about that…only to find later that there was a better way.
> 
> I still think DRY is a useful context, but I prefer to present it with some more nuance these days.

Also take this as a time to learn about different ways to implement an app. On the backend, you’ll see microservices, monoliths, SQL, and noSQL, and all of them are valid as long as they can be justified. This helps you expand your toolbox more because you’ll get the experience other devs have already brought to the team. There will be things you have opinions on and things you will adapt to. Just keep an open mind and open eyes as you move through the codebase and learn how the team has been working on the code.

## Add Tests

Some existing apps were created without test coverage being a high priority. As you work on new features and refactor existing code, don’t be afraid to add more test cases. This will be something you can bring to the team to improve development going forward. If you notice that coverage is low, you could take the initiative to lead the team in slowly increasing it. Writing tests for existing code is a great way to learn about how the software is supposed to work and understand the scenarios that the user may run into.

If there is already high test coverage, see if you can introduce e2e tests if they aren’t in place. You can write tickets to add tests to existing functionality and see how well documented the app is. Working with Product here is going to give you insight into how the teams work together and what the expectations are around product ownership. We addressed tests in several chapters in this book, so you can use that skill to suggest improvements or new metrics to measure as a dev team.

There is a balance for adding tests to an existing project because it can be an expensive undertaking. There may have even been a well-reasoned and thorough argument for not writing tests. Tests take time to write and then take time to maintain as the code changes. So work with your team to figure out the reasoning behind having tests or not having them.

## Learn What Different Alerts Mean

You might receive email alerts when an error happens in the apps and be unfamiliar with them. Take some time to understand how alerts work for the team, where errors stem from, and where the alerts are shared. Make sure you have access to any logging tools or the server where log files are stored.

As you become more comfortable with the apps, you’ll be expected to look into errors and warnings. Go through existing logs and learn some preliminary stats on common errors. Then use the logs to see which part of the stack is producing them. You can add this to your notes as you learn the process for handing them. When an error comes up, see if you can pair with another dev to learn how they get to the root cause and take more notes. This will introduce you to systems that you may not work with regularly. You’ll learn how communication is handled and how resolutions are made. Not every alert will be related to the code, and you’ll learn how to decipher what’s relevant to your team.

As you work on both greenfield and legacy apps, that will help you determine the direction you want to go in your career. You’ll gain experience in many areas, and you’ll start to gravitate toward certain areas. That’s when you have to make decisions about where to add depth to your knowledge and figure out which career path suits you best for this point in your life.

# Your Career

Over the course of this book, you’ve learned many things you need to do to set up a full stack app so that it’s maintainable in the long term. You’ve learned how to communicate with almost every team in the organization and why it’s important to do so. You’ve reviewed the product roadmap and helped add a vision for the technical roadmap. You’ve helped lead your dev team in discussions and with feature implementation.

At this point in your career, you’ll be adding more depth to your skills as you get exposure to more complex technical tasks and start mentoring devs who are earlier in their careers. This brings many senior developers to a crossroads with the path they want to take next. With all the knowledge you have about the frontend, backend, database, business domain, and the way the teams work together, you have a solid foundation for all the options ahead of you.

## The Technical Path

Some devs like to stay on the technical path and move to roles with more responsibility, such as architect, [staff engineer](https://noidea.dog/staff), and principal engineer. You’ll still be able to write code, but you’ll have a more prominent teaching impact. As you dive deeper into the technical side, you’ll start working at more of a system level than at the code level. You may have some tickets to work through, but most of your tasks will focus on how parts of the system will work together.

For example, instead of implementing individual endpoints, you’ll come up with how the endpoints should connect to the database, how they trigger other events, naming conventions based on the domain, and the data schema for them. You’ll be the one generating and documenting the diagrams for how all these things fit together and then presenting it to the team for feedback. You’ll be responsible for stepping in when others on the team get stuck on a debugging issue that involves multiple parts of the system and external services.

As you move farther along the technical path, you’ll be responsible for keeping packages up to date and understanding how their dependencies affect the codebase. You’ll bring more of a vision to unite multiple teams under the engineering department and help develop and drive a technical roadmap. This means you’ll spend more time thinking about how the app will work in the future and what everyone can do to prepare for that. That includes things like building internal tools to unblock the dev teams, tracking key metrics like PR review time and how well estimates are made, and understanding how dev tasks fit into the overall strategy of the organization.

Another skill you’ll develop is how you communicate. As you pass the senior dev level, how you communicate with others becomes more and more important. You’ll be mentoring early career devs, helping Product manage risks when estimates are off, and helping other devs get acknowledged for the work they do. The farther you go on this path, the more your role becomes enabling the rest of the team to get through their tickets quickly and accurately. This typically means you spend less time writing code and more time thinking about how to improve every part of the system to keep everyone moving.

This can also take you to the tech lead role if you’re more interested in the leadership side of the technical path. That usually involves more project management because you’re working with the team to meet timelines and you’re working with Product to deeply understand the goals and explain to them any technical blocks. At this point, you’re working with an engineering manager to help maintain or improve the culture of the team. As a tech lead, you might also jump in and work on some tickets if it’s necessary to meet a deadline.

If you choose to continue on the technical path, it will benefit you to stay on top of industry trends so that you can help the team implement the latest best practices. This will also give you awareness of new tools that may solve a problem the team has. Spend some time deepening your knowledge of the organization’s structure as well. Knowing how the different teams and departments relate to one another will help you figure out who to talk to when questions arise.

## The Management Path

The other path you can take is on the management side. This includes engineering manager, associate director, director, and vice president roles. These roles typically remove you from the code completely as you start to focus on the coaching, growth, and strategic sides of the engineering department. You may still write some code, but that depends on the size of your organization and how it defines the role. In these roles, you’re usually involved in more meetings with the individual team members to understand how they are doing and how they’d like to progress.

You’ll help create the culture for the engineering department and set up career levels that each developer can grow into. You’ll do things like review how much the teams are spending on tools and if there are better alternatives. A large part of what you’ll do is help everyone on your teams grow in their own careers. You’ll be listening to them in your one-on-one meetings and taking notes on what they tell you. It’s important that you give them good feedback on how they’re performing.

When they’re doing a good job, make sure you acknowledge them both in your one-on-ones and publicly. That will encourage them to keep up the good work as well as help them get the recognition they’ve earned, which is essential when it comes to promotions. On the other hand, it’s also important for you to give them critical feedback. The only way people on your teams can improve is if you tell them the areas where they can strengthen.

If other team members have brought up concerns about their work, make sure you let them know as soon as you can. It’s better to alert them long before reviews so that they have plenty of time to make changes. This is a great chance for you to coach them on ways they can improve and give them resources to teach them the skills they need. As you have one-on-ones with everyone, you should be asking for feedback on what you can do better for them. It can be hard to get your team to give you critical feedback, so try to lead this effort by example. A huge part of your job is understanding how you can better serve your teams, so the main way you can grow is to get feedback from them.

###### TIP

Some other good sources of critical feedback include exit interviews and anonymous surveys. The people on your team may feel uncomfortable directly telling you the ways you can improve, so this can take some of that pressure off.

If you move to a position where you’re a manager of managers, create documentation to help guide them through their own one-on-ones and give them tips on how to have difficult conversations with their people. This is an interesting role to be in because you likely won’t ever touch any code, but everything you document will serve as a starting point for other managers who report to you. So write docs on interview questions, things they can ask in one-on-ones, how they can keep people engaged in the organization’s culture, and how they can help their people grow. All this is part of helping them grow into better managers.

Something else you’ll do on this path is have more meetings with Product and stakeholders such as the VPs of other departments and maybe the C-suite executives to understand the plan for the business and how your teams and department fit into that. You’ll be directly involved in mapping out the Product roadmap and keeping the technical roadmap in alignment with it. This is when you’ll be involved in hiring decisions and help determine when it’s time for the team to grow to support the organization’s goals.

Communication is essential to success on the management path because that’s almost exclusively what you do. You won’t have tickets anymore, and your involvement with sprints and everyday development tasks will decrease substantially. The majority of your work will be meetings and documenting the outcomes from them in a way that translates into work for all your teams. You want to be the shield for your teams so that they aren’t pulled into a lot of meetings for clarification.

You have to learn how to balance your personal views with organizational decisions and communicate what needs to be done effectively. You need to feel comfortable pushing back on decisions that will negatively affect your teams. Part of management is advocating for your teams and making sure that their needs are considered in large decisions. Speak up when something seems odd to you because that’s one of the most valuable things you can do in your position.

###### NOTE

Here are some good resources when you’re considering the management path:

- [_The Manager’s Path_](https://www.oreilly.com/library/view/the-managers-path/9781491973882/) by Camille Fournier (O’Reilly)
    
- _Engineering Management for the Rest of Us_ by Sarah Drasner (Skill Recordings Inc.)
    
- _Resilient Management_ by Lara Hogan (A Book Apart)
    

## Professional Journal

Something I highly recommend is keeping your own personal documentation on what you see and do across the different projects you work on. That will be your template for how you approach anything. You can take some of the practices you learn at each organization along with you on your career. This is going to help you at every point because you’ll start to run into similar scenarios everywhere.

You should take notes on situations you’re the proudest of and situations where you struggled the most. This is invaluable when you’re interviewing for new roles because you’ll inevitably be asked some situational or behavioral questions. It’s also good to have when it’s time for reviews in your existing role. It can be hard to remember what you’ve done in the past 6 months or year, so write it down.

When you learn a cool new tech thing because it’s a standard practice at an organization, add it to this journal. You may be surprised how often similar issues occur across teams, organizations, and industries. If you’ve ever had that feeling where you think you did something before, this journal will help you validate that. I’ve had this happen a number of times where I run into difficult bugs or feature specs and I added them to my journal. Then I was able to reference them when the same scenario happened elsewhere.

This journal should be on your personal computer, not a company one. You never know when you’ll lose access or have the computer wiped when it’s not your own. Keep in mind that this isn’t a formal document. This is just for you, so it can be in any format you want. The sidebar has the format most of my professional journal entries follow. For tech issues, it tells me what happened and what I did to fix it. For situations I’m proud of or struggled with, I try to tell the story of what happened and how it played out.

# PROFESSIONAL JOURNAL ENTRIES

Here are a few examples of entries from my personal journal:

**Issue with finding a custom npm package in the CircleCI pipeline**

An error was being returned saying that the specific package couldn’t be found. The solution was:

- Delete the _package-lock.json_
    
- Reinstall all of the packages with npm i
    
- Push the changes in a PR to the branch with the pipeline issue
    
- Merge the updated _package-lock.json_ and let the pipeline run again
    

**Issue with unit test passing when run individually, failing when run with all the other tests**

Data was being mutated each time a function was called, so the test would only pass sometimes. The solution was:

- Finding out there was an array method being used that was mutating the original array
    
- Using the spread operator on the original array to create a new instance of it
    
- Using the array method on that new instance
    

**Proud moment: tracking down a two-year bug in the UI**

On a project, there was a bug in the UI that was hard to track down because we used a micro-frontend architecture, so there were at least six teams responsible for small pieces of functionality, and they all did things differently. There was even one team that wrote their frontend in Vue while the other teams used React. I was able to manage a large number of dependency conflicts and found that one of the micro-frontend teams was force-installing their package into the container app because they didn’t want to upgrade their internal dependencies. That led to all the other teams building workarounds for this one error. It took about 2.5 months to successfully fix the dependency conflicts between six apps, but that also fixed a bug that had existed for years. By the time I finished these upgrades, it allowed each team to refactor their code for higher quality, and it improved performance of the overall app because I was able to remove unused packages, decreasing the bundle sizes for the micro-frontend teams and the container app team.

## Moving to Other Areas

After traveling farther down a path or going down another path entirely, you might decide that you need a complete change and you want to try something else. Maybe you’ve always been interested in data engineering, and you have some background in database management. Or you wanted to try being a DevOps engineer because you’ve done a little work with the infrastructure that sparked your interest. It’s fine to take up a new specialty and work on that for a while.

###### NOTE

Regardless of the path you choose, it will help you greatly if you take some form of leadership training. See if there’s anything your organization recommends or has access to. Several books have helped me learn about leadership, even if some of them are unconventional:

- _How to Win Friends and Influence People_ by Dale Carnegie (Simon & Schuster)
    
- _Thinkertoys: A Handbook of Creative-Thinking Techniques_ by Michael Michalko (Ten Speed Press)
    
- _Spark: How to Lead Yourself and Others to Greater Success_ by Angie Morgan, Courtney Lynch, and Sean Lynch (Houghton Mifflin Harcourt)
    
- _Wherever You Go, There You Are: Mindfulness Meditation in Everyday Life_ by Jon Kabat-Zinn (Hachette Books)
    

If your organization has the capacity, you may be able to help out on different initiatives and improve your skills in the process. You could also choose to take a less senior role in a different area, such as becoming a junior data engineer. Your core software skills aren’t going anywhere, and working in a different part of the tech world can improve your creativity because you can see how things work from another viewpoint. This is a time when you can get certifications or help work on smaller functionality your organization needs. There is a trade-off when you do this exploration because the current state of our tech stacks is constantly changing, so you might not stay up to date on the latest and greatest.

You can always jump around to different specialties as well. Maybe you try data engineering and you don’t like it. You can come back to software development until you figure out what the next thing is. You might find that there isn’t a next thing and you’re content with where you are. Not everyone wants to climb the ladder and get promoted as soon as they can, and that’s fine. It’s your career, and there is no such thing as “normal.” You have to do what’s best for you and your life because you’re the only one living it.

# Conclusion

Congratulations! You’ve made it to the end of this book. By now you’ve done everything on the frontend and backend that it takes to build a maintainable full stack app. You know what it takes to evaluate tools, handle cross-team communication, keep track of technical decisions, and help out your team members when you need to. You’ve learned about the subtle and unspoken skills that go into being a senior dev, such as writing solid documentation for everything and bringing your experience to the table.

I sincerely hope that you’ve found this book useful and you’ll refer to it if you ever get stuck at any point in your development process. If you found any of this helpful, I’d love to hear from you about your experiences! Or if you thought something was off base or I missed something, I’d love to hear about that as well. While this book doesn’t cover every potential consideration you’ll run into, it does go over a lot of the standard ones. As you move on through the years and through your career, always keep in mind that there’s something new to learn all the time. As software engineers, we never know everything, so keep an open mind and a sense of humility, and that will take you a long way.