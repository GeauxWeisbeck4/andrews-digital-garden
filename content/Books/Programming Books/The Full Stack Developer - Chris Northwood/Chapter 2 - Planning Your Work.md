---
id: 01JB58FGP72PH2RK6TD4R84QTM
title: Chapter 2 - Planning Your Work
modified: 2024-10-26T16:32:39-04:00
tags:
  - full-stack
  - books
---
# 2. Planning Your Work

Chris Northwood[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Aff2) 

(1)

Manchester, UK

Developers who are just starting out often believe that their job is to build what they’re told to build by their managers or clients. The more experienced a developer becomes, the more they realize that building the right thing is more important than building the thing right. A quick hack that does what the user needs it to do is much better than a beautifully crafted codebase that doesn’t solve the user’s problem, although a working but hacked-together system will quickly stifle progress.

Having an effective way to develop plans for the work you’re going to do is the best way to ensure that you approach the work so that it actually meet the needs of the user. It’s a commonly held misconception that developers exist to write code, but a development team without an effective planning process will often spend time writing the wrong code, or getting stuck and being unable to move forward with the code they do write, stumbling over obstacles and falling victim to risks they’ve failed to mitigate. An effective planning process, including the whole team, can minimize or avoid these problems altogether. A competing concern with the planning process is that it is often also used to develop estimates and communicate to external stakeholders how well the team is performing. This concern can often make a planning process toxic, and give planning as a whole a bad reputation. However, it is possible to implement a planning process that avoids these pitfalls while keeping those people outside of your team happy.

One key difference between a digital organization and a traditional one is the relationship between the development team and the business. In a traditional organization, developers live in their own team, and so don’t necessarily have the domain knowledge around the area of the business that they’re developing software for. Communicating that domain knowledge to developers is hard, so the people who actually had the knowledge to define the organization’s software needs were tasked with communicating this information to the development team via documents with names like “specification document” or “functional requirements document.” As building software can be expensive, it was believed that getting the specifications right from the beginning would minimize the cost of any potentially expensive re-work, so a significant amount of effort went into preparing these documents. The problem is that these documents became a replacement for team members communicating with each other. Furthermore, as the overhead of preparing these documents, and getting sign offs, was quite large, it made more sense to have fewer, larger projects. The documents were as comprehensive and precise as the code that was written to implement it, but had the downsides of being written using the imprecision of human language, and processes made them difficult to change.

One term I’ve not used so far in this book is “agile,” but the eagle-eyed reader will have noticed the essence of the agile movement creeping into the previous chapter. The agile movement has had one of the biggest effects on the growth of digital businesses and the changing structure of development teams. The process described above is often called the “waterfall” model, characterized as the opposite of agile. In reality, no organization really works in a strictly waterfall way. And even the best agile teams will have some sort of mini-waterfall, although a mini-waterfall can feed into itself; you can’t test a thing until it’s built, and you can’t build it until it’s defined, but you can define those tests before it’s built, and building a part of a system can inform another part.

At the core of the agile movement is a manifesto that defines agile as bringing people back into the process of building software, breaking down communication barriers, and embracing change.

> _We are uncovering better ways of developing software by doing it and helping others do it. Through this work we have come to value:_

- _Individuals and interactions over processes and tools_
    
- _Working software over comprehensive documentation_
    
- _Customer collaboration over contract negotiation_
    
- _Responding to change over following a plan_
    

> _That is, while there is value in the items on the right, we value the items on the left more._
> 
> —The Agile Manifesto: [http://​agilemanifesto.​org/​](http://agilemanifesto.org/)

When you read the agile manifesto, it’s important to understand what it really means. “Individuals and interactions over processes and tools” doesn’t mean “don’t use Jira.” It means don’t _only_ use Jira, or let Jira get in the way of communicating.

The third paragraph of the agile manifesto is often forgotten; it’s not a binary choice between the left- and right-hand concepts, but a preference for those on the right. So, have some processes, and use tools, but don’t let those get in the way of individuals and interactions. Documentation can help with communication, so use that, but don’t document every part of what you’re building before you build it. Contracts are important—they’re how you get paid—but building something that the customer doesn’t want just because the contract says to is a waste of everyone’s time. And having a plan makes getting started very easy to do, but you should make sure the plan can change.

Digital organizations (as opposed to traditional ones) are agile because they have broken down barriers to communication by mixing business teams with development teams. Without arbitrary barriers, lighter processes for defining what needs to be done can emerge. Better ideas emerge as well, as communication can now flow freely between the people with domain knowledge and the technical knowledge, and these silos break down. With a lighter process with a lower overhead, each change can be smaller, which minimizes the time between defining a project and it actually getting done. Minimizing this time means that big changes halfway through a build—to respond to some external change or new discovery—are no longer needed, as there is less time for things to change in the first place. When a change does occur, the impact itself is then minimized, as it either requires replanning of upcoming work, or by altering the work currently in progress. Work in progress should be scoped to be small, so the impact of such a change is not huge.

Agile is not a set of processes, nor is it a series of “rituals” like a daily standup, user stories, or retrospectives. Agile is a way of thinking, and must expand beyond the development team and into the rest of the organization. An “agile” team in a traditional organization may, at best, have an effective delivery approach and satisfy internal stakeholders, but unless the whole business can operate in an agile way by applying the above principles, you’ll be limited in how quickly you can get software into people’s hands, rather than actually making a whole organization respond to changes, which is what a modern business needs to do. At worst, an agile team in a traditional organization will be at constant loggerheads with the rest of the organization. For example, the organization may want long-term stability and visibility of their IT systems that agile simply can’t provide; the very nature of being quick to respond to change means you can’t pin down a six-month roadmap, unless you agree not to change it, which is no longer agile. In traditional organizations, if the development team can’t iterate with the business, then they can’t respond to the change that the rest of the business is responding to. Worse, if the whole organization is locked into this long-term planning and stability process, then the organization can’t respond to changes from outside the organization, making it harder for everyone to achieve their goals.

Applying agile techniques in a traditional environment like this is doable, but if the team is asked to build big things all at once, the project becomes like a “waterfall” project, where large chunks are defined up front, and the smallest chunk the business is willing to launch is the whole product. Development and QA happens in sprints against this backlog, but the backlog itself can’t change in response to external circumstances. The agile process should start with the initial idea for a project or feature, not just at the point developers pick it up.

## Identifying Requirements

The role of a developer in identifying requirements can vary a lot between teams. At the simplest level, identifying the requirements of the thing to be built is often a matter of asking the rest of your organization to give them to you, and although this can work well in mature organizations, or where there’s a supplier/client relationship, you can’t always assume that the organization knows what they want. What they ask for is not necessarily the same as what they actually want.

Many teams, especially larger teams, designate an individual who is largely responsible for this. This is sometimes called a business analyst, but even if you have no one with that job title, the role will exist somewhere in your team—perhaps in a product owner, or a project manager, or even a tech lead. One-person freelancers typically occupy this role too.

It’s better to fill that role in your team rather than outside it, but it’s also important that the individual does not become a one-person silo or bridge. It’s important for developers to work with them to truly understand what they are making, and to also feed back into the process. If you understand what the organization is trying to achieve, you will reduce your feedback cycle time, allowing you to complete work more quickly. If you have a question, rather than having to wait for an answer, you can make an educated guess and continue work. In cases where the question is more fundamental to what is being built, then stopping and asking is still important, but by better understanding the requirements, you know which questions are truly important, and which aren’t.

This is especially important for senior developers and leads, or those who want to grow into those roles. Senior developers are often expected to attend meetings with a wider group of stakeholders to help define a product and gain an understanding of what the stakeholders want. At first it may seem that time away from code is not as productive as time spent coding, but for a full stack developer, extending beyond the code actually allows them to produce a better end product, and help those around them work better. The days of requirements being thrown over a wall to developers, and the downsides that entails, are over.

Workshops are one of the most common ways of eliciting requirements. Workshops involve getting a bunch of people together and running activities, with the hope of getting some useful information out of them. The people from outside your team who attend a workshop are often referred to as stakeholders; these are the people who have an interest in what you’re building. For a news web site, this might include people like the editors, or for an online store, it might include the marketing team and buyers. It can also include people from outside your organization; in an e-commerce example, the backend systems might include the suppliers you work with. It also probably includes your end-users.

The format of these workshops varies depending on who the stakeholders are and what phase of the project you’re in. Early on, it is common to initiate a discovery phase to understand the problem you’re trying to solve, rather than jumping straight to a solution. Some may think this is anti-agile, but it’s important to do just enough to avoid starting your journey in the wrong direction. You do not need to fully understand the problem space and fully define your solution before starting to build.

Workshops in the discovery phase might involve a large group of stakeholders, getting everyone together in a room to define the problem that needs solving. The stakeholders might not agree on what the problem is, but by eliciting all of their thoughts you should be able to pull out some common themes that will allow the delivery team to identify the challenge, and define a set of steps to hopefully overcome it.

This is referred to as the double-diamond model, which can be seen in Figure [2-1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig1).

![../images/471976_1_En_2_Chapter/471976_1_En_2_Fig1_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_2_Chapter/471976_1_En_2_Fig1_HTML.png)

Figure 2-1

The double-diamond model, which shows how to diverge to generate breadth of ideas, and then converge to generate depth on both the problem and the solution

In this form of double-diamond model, you get everyone together to explore the entire problem, and then the team that is responsible for devising a solution refines it into the actual work that needs to be done to solve the problem at a high level. First, you identify the general shape of the problem space by first exploring the breadth of it, and then drilling down into any detail and unanswered questions. Then, the second converge-diverge diamond is followed, with first identifying possible solutions, and then eliminating inappropriate ones and defining the actual specifications and details for each part of the solution. Remember that your job is to actually come up with the solution; stakeholders might understand their problems best, but not necessarily the best way technology can help them. This is where the double-diamond model can help, as it involves stakeholders to get the relevant information from them to design an appropriate solution, but a core group (often just the most senior members of the team) then refines that into what actually gets built. This prevents the dreaded design-by-committee outcome, and also the situation most famously described by Henry Ford: “If I had asked people what they wanted, they would have said faster horses.”

There are many other techniques that make use of the double-diamond model, and many other models and ways of running workshops to gather requirements, but this is one example.

The field of User Experience (UX) has grown out of a mixture of the academic field of human-computer interaction with the traditional role of visual and interaction design, and many UX teams follow a user-centered design approach, which puts the user at the heart of the requirements-gathering process. A user experience practitioner is a very powerful ally during a requirements elicitation process. Many of the most successful organizations out there put user needs first and have teams with engineering and UX practitioners working side by side. Often the lines of those roles blur, and some UX practitioners contribute to the engineering, or developers take part in the UX processes, increasing team effectiveness through reducing communication overheads.

These forms of requirement gathering are known as qualitative research—that is, they focus on abstract facts and information and how humans feel. Another way of eliciting requirements comes from a method called quantitative research, where raw data is analyzed to discover information. Quantitative data gathering is most effective when your product is in front of a user, and can give stakeholders a great deal of comfort as compared to qualitative research, which can often be driven by “gut feelings” or other subjective information. On the flip side, humans are not rational beings, and many will trust their gut over statistics!

There is a definite science behind the analysis of quantitative data, and it’s important not to underestimate it. Statistics is a complicated subject, and it is easy to apply an inappropriate form of data analysis, or misinterpret the results, leading to bad decisions.

When working with users, quantitative data is much easier to gather than qualitative, so quantitative data can become more helpful later on in the development of a product when there are many real world users of your application. Continuous delivery is a technique that encourages releasing early and often, and for teams that practice continuous delivery, the use of quantitative data can become much more effective as it allows for collecting data whilst parts of a new feature are beng developed. This is discussed further in later chapters.

The most common form of quantitative data comes from using analytics systems, which report on how users interact with your product, but it can also come from elsewhere in your organization. For example, if building a tool to be used in a call center, looking at the data such as the average wait time can prove enlightening when it comes to discovering what requirements you might need to consider in your product.

## Defining the Work

Once what a product needs to do has been discovered, it’s important to define that work. This definition works as both a method of communicating to the wider team what needs to be done and a way of ensuring that the problem is truly understood to the point where a solution can be expressed in code.

Within many teams, the user story has become almost synonymous with agile, although there are many other ways of documenting a requirement, and user stories can be used in non-agile settings too.

User stories frame a new feature from the point of view of a user, and are commonly expressed in the form:

> As _someone_

> I want to _do something_

> So that I can _achieve something._

Other popular techniques include hypothesis-driven development, the famous functional requirement document, or even more ad-hoc approaches such as free text on the back of an index card.

It’s very easy to write bad user stories. Take for example:

> _As a user_

> _I want to register for an account_

> _So that I can check out quicker next time._

This type of requirement will often come from someone inside the business, not a user, and is then retrofitted onto a user. Do users really care about checking out quicker? Maybe for a site they use a lot, but not for those that are accessed infrequently. Be wary of requirements that have been retrofitted onto a user; they are often not actually plausible, and can result in wasted effort. Framing user stories correctly can change your approach, and help better meet the desired goals. The above user story may be more reasonably expressed as:

> _As the business_

> _I want users to register for accounts_

> _So that I can gain more insight into my customers, and research shows that customers with accounts are more likely to return to the site._

Retrofitting often happens when it is hard to write the “so that” part of a user story. If it’s hard to write that line, the value may not be obvious and needs more definition. The second user story here can be more easily challenged, and be subjected to wider discussion with the rest of the organization. For example, does the value added here outweigh the cost of adding more friction to the checkout process? Also, the definition of what data to capture is different between the two stories.

Another sign of a poorly defined user story is one that simply says, “As a user.” Most users have different needs, and in the first story above, the statement doesn’t apply to all users—only users that are likely to return to the site. When building for users, it’s important to have good understanding of who your users are and the differences between them. Very few systems have only one type of user. It is common practice to develop “personas” that represent typical users and to give them identities to make it easier to understand their behavior.

Sometimes a user story is hard to define because it covers a wide range of related technology and user journeys. These types of user stories can often be converted into “epics,” which are user stories that have been broken down into (or are being broken down into) a number of smaller, more granular user stories. Epics often appear very early on, when an idea first comes around. For example, if you’re building a brand new e-commerce web site, an epic might exist in this form:

> _As a customer,_

> _I want to purchase an item from the web site,_

> _So that I can have that item delivered to me and I can enjoy it._

This can then be broken down into smaller user stories based on the different mechanisms of purchasing something. For example, user stories might emerge around searching the catalogue to find an item to purchase, or different ways of paying for an item. These individual user stories can be much easier to reason through and prioritize. For example, if you potentially have users outside your country, you will often need to build in an address finder, or have shipping cost calculators for those countries, but such functionality can be deprioritized to focus on the local market first and then picked up at a later date. Trying to tackle a user story that is too large can cause problems, as it is harder to see what has been missed, and can cause unnecessary work as it brings unnecessary features along with it.

Bill Wake proposed the mnemonic INVEST to remember the important characteristics of a good user story.

- Independent
    
- Negotiable
    
- Valuable
    
- Estimatable
    
- Small
    
- Testable
    

An independent user story is one that is achievable without any other user stories being done first. This means that you can reprioritize your stories at any point, by moving them up or down your backlog in response to any changes in your organization.

Negotiability means that a story can be changed, rewritten, or discarded at any point until work starts on it. For example, if a story is proposed that has significant technical implications, it should be possible for developers to challenge that story and determine if there’s a simpler way of implementing it that still satisfies the business needs.

A user story must be valuable for it to make any sense. Stories that call for busywork, or fun technical items (for example, trying out a shiny new technology) but actually deliver no value to the organization have no worth as stories, so should not be on the backlog. Some teams go further than simply requiring a story to be valuable and assign “value points” to an item, which are an abstract concept similar to story points that can help with prioritizing items. High value/low cost items tend to be the highest priority, whereas low value/high cost items can be discarded. This is why user stories must also be estimatable. If a user story isn’t estimatable, it is difficult to prioritize. It is also indicative of an underlying issue with the story. If something isn’t estimatable, it may be because a story is not clear or defined enough. This also usually means it’s not clear or defined enough for a team to be able to implement it.

Small is another important characteristic of a user story. Many teams will define a rule of thumb for how large a story can be. Teams running in sprints will often declare a story must be small enough to complete in a single sprint, and others will use a different rule of thumb based on what they feel comfortable with. When a story is large, it is easier to miss subtle details within it, which can lead to wasted effort, and it also becomes harder to realize parts of that story that might have value sooner, as instead the whole story must be defined. Anecdotally, large stories cause the biggest issues in delivery teams.

The final requirement of INVEST is that a user story must be testable. If a story is not testable, it means there’s no way of figuring out if what the team has implemented is correct, or if it has the right impact. Testability here often refers to the work done by a QA or automated tests within a team, but is becoming more widely used to refer to a way of measuring the impact of delivering that feature on an organization (and therefore checking that the story did have value). When a requirement isn’t testable, it’s often due to a lack of clarity on what it is supposed to do and how it is supposed to behave. A lack of testability can also mean that the value isn’t well-defined.

These requirements can sometimes be in conflict with each other. Often, to make a user story independent, you must make it quite large. This is especially true when you start building the foundation of a system. The first item you build might require you to set up build tooling, or have dependencies on other systems—for example, a login system. Some teams choose to address this by communicating that there will be a “first story overhead” to the first item they build, or by having a “foundation sprint” to get things ready. There are challenges beyond this initial foundation stage, too. For example, if a group of related but independent stories presents itself, it is common for the first in this group to require a larger set of technical tasks, which then makes implementing the later stories easier. This can challenge the estimatability of stories, as the order in which they are done will change those estimates.

Another example occurs when the build of a minimum viable product (MVP) is underway. The MVP is the bare minimum a product must do to be useful or valuable, so by its very definition, the product does not have value until the MVP phase is written. All stories in the MVP therefore become dependent on each other, and an item by itself may not be seen as valuable (for example, why have the ability to browse a catalogue until you have a way of checking out?).

In these early stages, taking the mnemonic literally is difficult, although as a product continues beyond MVP into continuing or BAU (business as usual) development, it becomes much easier to stick to the principles, and doing so becomes a key enabler for continuous delivery.

When new stories are proposed, they do not always meet the requirements of INVEST immediately. A new story is often a placeholder to indicate more work is needed to identify the value, or discover enough information to make it estimatable. It is common to have an “ideas” or “discovery” phase of your planning process where these stories are refined before moving on to being a full-fledged part of your backlog.

Once the high-level definition of a requirement is complete and it appears on the backlog, it then becomes necessary to define the rest of the work needed to make it ready for the team to implement. The kind of definition required will vary between teams. Relatively immature teams will require more definition before moving to the next phase, whereas more mature ones can work with less detail. Getting the level of detail just right is tricky! Too much and you’ve wasted time and effort in over-specifying; too little and you risk re-work or the wrong feature being built.

As work progresses from the idea phase into production, it requires differing levels of detail. As a developer, you may have certain expectations about what you are given, meaning there are also often stages before the development stage (such as visual design) that must be completed. Since this definition happens inside your team, it’s necessary to have good communication and clear expectations about what happens during each phase. Senior or lead developers may be brought in early to help other team members (such as UX practitioners) understand any technical restrictions to that process, as well as identify alternative ways of solving a problem.

As a developer, the most common level of definition required is a set of acceptance criteria, and often some wireframes or visual assets if there is a UI component involved. It is unhelpful, though, for these to appear suddenly in a perfectly polished form; it’s useful for the development team to have visibility on work that is coming their way, before it is fully defined.

The method of communicating this definition to the development team is important too, and depends heavily on the way the team likes to work. One effective method of doing this is called behavior-driven development, which relies on a mixture of conversation and written record to establish these acceptance criteria. Behavior-driven development is discussed in more detail in the Testing chapter.

Verbal communication is a much richer method for humans to communicate than written communication, and although definitions often come to developers from other roles on the team, that is not the only direction communication can flow. Asking questions—and, for other roles on the team, being available to answer those questions—is as crucially important as producing a good definition in the first place! There are many effective teams that use verbal communication over a written record, and although this often causes problems with scale, it can work very well. Remember the agile manifesto: “People over processes.”

## Tracking the Work

Even a one-person team will often find that they need to track the work being done. Tracking work is often a minefield, with many different people wanting the work to be tracked for different reasons. The team might want work to be tracked so they can have better visibility on what everyone is doing; external managers might want to track the effectiveness of one team relative to another; stakeholders might want to know when a feature they care about is completed; and project managers might want to know how long people are spending on each feature so they can charge customers appropriately.

However, producing reports and alternative views to satisfy all of these differing needs can be a considerable overhead for a team, so many use tools to automate as much of this work as possible. Despite many claims, there is no “one true tool” for tracking work. A team that uses Jira may find Trello completely unsuitable for what they want to achieve, and one that uses a physical wall with index cards will probably never be able to apply their process into a team that embraces remote work.

Regardless of which tool you use, you’re unlikely to find something that’s perfect for everyone, and you may have to make some compromises to get the best fit for all. The method used to track work should be revisited to make sure it’s working for everyone, and not causing unnecessary work.

In addition to the visual tools you may use to manage your work, there are other tools that can be used to manage the flow of work to your team. Sometimes called methodologies, there are many to choose from—one popular option within agile communities is Scrum. Some other broader methodologies or frameworks, such as PRINCE2, cover a whole range of other techniques (such as requirements gathering and testing), in addition to the process of running a project.

### Scrum

Scrum has become very visible in the field of software development, to the point where Scrum is synonymous with agile in some communities.

However, Scrum is not a silver bullet, and it is not perfect.

The agile movement in general, and Scrum in particular, grew out of the Extreme Programming (XP) world, and one criticism of Scrum is that by discarding the technical aspects of XP, it has resulted in a weaker process. Another is that a focus on “rituals” within Scrum (such as the fixed-duration sprint, daily standup, planning, and retrospective) has given rise to a cargo cult movement, where organizations “adopt” Scrum by taking on these rituals, but it still sits within a larger waterfall movement. This is amplified by the proliferation of Scrum trainers and certifications that teach you the rituals and principles of Scrum, but not how to apply it in the context of your organization. This can lead to the worst of both worlds and a disillusionment with agile in general. When implementing Scrum, it is important to remember that you can adjust the framework for your circumstances, and you do not need to follow a textbook definition of Scrum.

At the core of Scrum are several key artifacts, roles, and rituals. The single most important thing is the backlog, which is a prioritized list of things to work on. In Scrum, the development team works in fixed-length sprints (often between one and four weeks, but they do not vary, and the length of the sprint is agreed upon up front), and agrees at the start of the sprint how many items to accept from the backlog into the sprint. By the end of every sprint, the development team should have produced a “potentially shippable increment” of the product, so value is being added on a regular basis.

A Scrum team consists of a product owner, who should be empowered to make decisions around the product being built and is responsible for building the backlog, and should be accessible to the development team; a Scrum Master, who is responsible for facilitating Scrum within the team and working with the product owner and stakeholders to produce a backlog, and who can facilitate removing impediments or “blockers” that stop work in the sprint; and the development team itself. A development team should be cross-functional and capable of doing all the work to deliver a backlog item within a sprint.

A sprint starts off with a planning session. In the first half of the sprint planning session, the product owner and the development team negotiate which backlog items should be accepted into the sprint—the idea is that the team only accepts items it thinks it can complete in the sprint. A “definition of done”—an agreement on what an item being completed actually means—is determined by the Scrum team. Commonly, a “definition of ready” also exists, defining what it means for a backlog item to have sufficient definition for a team to accept it.

> _An example definition of ready_

> For a feature to be considered ready for development, the following things must be complete:

- The product owner has prioritized it.
    
- UX, product, and dev have developed and agreed upon acceptance criteria for it.
    
- The development team has estimated it (or re-estimated, if the scope of the item has changed).
    
- The technical lead is happy with the proposed approach, if it impacts the technical architecture.
    
- If the feature involves any changes to administrator workflow, the administrators have been consulted.
    
- If the feature requires any changes to the data schema, the data architect has reviewed it.
    
- Any dependencies on external APIs have been identified.
    
- If any new UI assets are needed, they have been identified.
    

_An example definition of done_

For a feature to be considered done, the following must be complete:

- The code should conform to the coding style guide.
    
- All new code should have unit tests written against it, and all unit tests must pass.
    
- The tech lead must have reviewed the impact on the cross-functional requirements of the product.
    
- The code should have been written by a pair, or reviewed by a peer.
    
- The UX designer and product owner should have seen the feature in the QA environment.
    
- The tester should have exercised the functionality and smoke tested the product in the QA environment.
    
- Browser-based automation tests should have passed in all supported browsers.
    
- Any bug tickets identified by the manual QA should have been resolved.
    

Estimation techniques known as “story pointing” and “velocity tracking” are used to help determine how much work a team can accept into a sprint—these are discussed in more detail in the Prioritization section.

In the second half of the sprint planning session, developers break down the individual stories into tasks that make sense for that team. For a truly cross-functional team, these tasks can often be things like “create acceptance criteria,” “produce wireframe,” or “add new API endpoint to backend.” One of the weaknesses of Scrum is that it expects the entire team to be able to work on these tasks, but in reality, there are specialties within a team, so the wireframe tasks may require a designer to work on them before a developer can pick them up.

While the sprint is in progress, each day should start with a daily standup, where the product owner, Scrum Master, and development team get together around the task board and discuss what they did the previous day, what they plan to do that day, and whether there is anything stopping them (blockers). Each person should be limited to one minute, and only one person should talk at a time. An alternative approach is to “walk the board,” so instead of individuals giving updates, the discussion focuses on the progression of individual tasks or backlog items. The standup is referred to as such because standing (for those that can), as opposed to sitting, keeps the meeting short. The standup is not intended to serve as an update for management, and only those who are part of the core team should talk, although others are allowed to observe.

> _There was once a pig and a chicken. The chicken turns to the pig and says “We should open a restaurant. We could call it ‘Ham & Eggs.’” The pig disagrees, saying “You’d be involved, but I’d be fully committed.”_
> 
> —Old fable

Sometimes members of the core Scrum team at a standup are called pigs, and the observers and stakeholders are called chickens. Only the pigs should speak.

If any blockers are identified in the daily standup, they should not be solved in the meeting unless absolutely necessary. Instead, the Scrum Master should make a note of them and mark the corresponding item as blocked. After the standup, team members can discuss as a smaller group any actions that may be needed and allocate those appropriately in order to unblock the task.

At the end of the sprint, the sprint review occurs. As with sprint planning, this occurs in two parts. The first part is the demo, which is a chance for the team to show the completed work and the shippable increment to any stakeholders, and the second is the sprint retrospective. The sprint retrospective is a chance for the team to look back at how the sprint went, and to identify any ways their processes can be improved to make the next sprint more successful. The retrospective must be as open and honest as possible, and is attended only by team members—other stakeholders are rarely invited. The Continuous Improvement section further on in this chapter discusses retrospectives in more detail.

Within a sprint, it is common to have a burn-up (or burn-down) chart, which tracks the progress of the team within the sprint, and to indicate whether or not it’s likely that all the backlog items will be completed. The backlog as a whole can be tracked in this way, across sprints, to track progress towards any larger goals, or to help make estimates.

One example of a burn-up chart is shown in Figure [2-2](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig2). The chart shows the total number of story points committed to in the sprint (you can see a mid-sprint increase in scope), as well as how many have been completed. This can be used to predict whether or not the items in the sprint will be completed on time.

![../images/471976_1_En_2_Chapter/471976_1_En_2_Fig2_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_2_Chapter/471976_1_En_2_Fig2_HTML.png)

Figure 2-2

A sample burn-up chart for a sprint

There is extensive literature available on this form of project management, and the many styles of tracking and potential extensions to it (especially if you’re considering having multiple inter-connected teams working on it), although this summary should be enough for a developer to have a basic understanding of the concept.

### Kanban

An alternative to Scrum within agile teams is Kanban. Kanban grew out of the lean manufacturing movement, and draws analogies between that movement, which revolutionized car manufacturing, and software engineering. Kanban is often used as part of a set of processes known as “lean” software development.

In the late 1940s, Toyota started analyzing its supply and production chain and comparing it to the way supermarkets managed their inventory to keep enough of every item in stock that is sold, but to minimise storage overhead and shelf-time when food could rot. By the 1990s, this had evolved into a more generalized movement called “lean manufacturing,” which was based on the idea that smoothing the flow of work can make production more efficient. If a station on a production line is producing components too quickly, it causes waste due to the chance of spoil or introducing additional storage requirements; but if one station is too slow for the next stage in its process, this causes a bottleneck. Placing minimum and maximum limits on the size of queues between stations in a production line allows you to identify where issues may arise and maximize the efficiency of your plant.

Although on a surface level these concepts seem fairly disconnected, the underlying principles work surprisingly well when applied to software. Instead of a physical production line, we have ideas, which progress through a system at increasing levels of definition until they reach the highest level of definition, which is an expression in code. Although there are clearly differences in the details of those steps, managing the way work flows between the steps is very similar.

Kanban is more a general set of principles than the more rigidly defined practice of Scrum. There is no Certified Kanban Master qualification to pay for. The principles of Kanban are fairly straightforward (although there are many different variations on them):

- Make all work visible.
    
- Limit work in progress.
    
- Practice continual self-improvement.
    

As Kanban is more flexible, it can seem harder to adopt. However, by starting with these principles, and examining some of the common scenarios in which Kanban is used, you should be able to identify a version that is appropriate for your organization.

A team inside a modern digital organization should be empowered to organize their work however they see fit, according to what works best for their team. Although it is sometimes difficult to convince skeptics, if your team is not empowered to manage work the way you’d like, then those constraints can limit your output.

For teams who are new to an agile way of working, Scrum can be a comforting first step, with rules and procedures similar to the waterfall method or change control processes. If Scrum does not work for you, it is not necessarily because you are “doing Scrum wrong”; perhaps Scrum is not a good fit for your team. Drop the limitations that are holding you back and don’t add value, and alter the process until it works for you.

Teams that use Kanban from the outset can look similar to ones that use Scrum. The daily standup often features as a regular event, and there’s still a task board, which will look familiar to those coming from a Scrum background. Figure [2-3](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig3) gives an example layout, but upon closer inspection, you will see there are differences between a Kanban board and a Scrum board.

![../images/471976_1_En_2_Chapter/471976_1_En_2_Fig3_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_2_Chapter/471976_1_En_2_Fig3_HTML.png)

Figure 2-3

A Kanban board with items in each column

One important difference is that Kanban does not work in sprints. Instead, once the status of a task changes, you pull it from one column into the next one. Additionally, “work in progress” limits are applied before something is pulled into a stage of development. Teams work on one item at a time until completion, rather than having multiple tasks in flight at once. When an item is completed, work can then move to the next priority. This encourages a regular flow through a team, instead of Scrum’s focus on only having a potentially shippable increment every two weeks. It can also allow for fast responses to changes—instead of having to wait for a new sprint to take on a new task, or having to change the scope of a sprint halfway through, a high-priority item can be moved to the top of the backlog and picked up like normal work.

Work-in-progress limits can feel like a constraint on a team, but they are designed to keep the team efficient. A work-in-progress limit is a limit on the number of items that are in a particular column at any one time. For example, a team with three developers may choose a work-in-progress limit of three. Working on more than one ticket at once is discouraged, as context-switching can lead to lower productivity. n+1 is also a common work-in-progress limit (e.g., four items for three developers). This allows for the fact that sometimes external blockers can occur, which means a ticket has to be paused for a while until that external blocker has resolved itself. Ideally, these blockers should be identified ahead of starting development, as the longer code sits without being deployed, the more likely the requirements for that code will change, as well as adding overhead if the way that code integrates into the system changes, yet the code brings no value at that point, just overhead. If many blockers are identified at once (for example, if a ticket is much more work than first assumed), and a work-in-progress limit is reached, then the team can swarm around a problem, so multiple people are working to resolve it. Sometimes this isn’t feasible, in which case work-in-progress limits can be broken, but only with buy-in from the whole team.

There are other differences between Scrum and Kanban. People who embrace Kanban often feel empowered to configure the columns on their board as they’d like, and teams are encouraged to start tracking work all the way through the cycle, not just development tasks. Many boards start with a first column of “idea” and a last column of “ready to deploy” or “in production.” If a ticket is on the board, then there is work to do on it. Simply being “ready to deploy” isn’t enough, as there’s still a step to deploy the work before the benefit of it can be realized. For teams that are running A/B tests or embracing hypothesis-driven development, an item “in production” is not complete until any useful information has been identified or until the experiment has finished running.

These columns will often have entry/exit criteria against them, which behave like “definition of ready” and “definition of done” in Scrum, but instead of determining whether or not a story is accepted into a sprint, it determines transitions between columns. These criteria serve as checklists to make sure a story, an item, or work is truly ready, especially between columns that are handled by different people within a team.

Although Kanban doesn’t use sprints, it is common for teams (especially teams that have moved to Kanban from Scrum) to have regular meetings that occur at the same frequency as in a sprint. For example, getting a team together to plan does not necessarily have to result in the formal outcomes of Scrum, but can be a useful way to estimate work and make sure the whole team is aware of any upcoming pieces of work, and allow them to make contributions in that planning or design phase (for example, pointing out technical dependencies). Similarly, retrospectives often run at a regular pace, following a similar format to Scrum, as a way of satisfying the Kanban principle of continuous self-improvement.

Other teams do not feel a need to do this (especially small teams where people are working cross-functionally). For example, items of work might be discussed ad-hoc as they arise, negating the need for a formal planning session, or retrospectives might be run as the need arises. Some teams have a “retrospective area” on the board, where any team member can add a sticky note or index card with an issue on it, and these are discussed and resolved after stand-up. The flexibility of Kanban allows you to find what works for you.

A technique known as Scrumban also exists, of which there are many variants. One Scrumban variant combines the time-boxed iterations of sprints in Scrum with the flow of Kanban. In this method, Kanban is used to manage all the pre-sprint and post-sprint activity—for example, requirements gathering, UX work, or post-deployment analysis of a KPI—but a core development team works in the meantime by taking a batch of tickets and working on them in a sprint. This balances the stability of upcoming work for the development team while giving more structure to the activity around the work of developers. This form of Scrumban is often implemented as a precursor to the elimination of sprints and full Kanban.

An alternative option is one where time boxes are not used, but bundles of items are sprinted on as a group. Many organizations have developed their own variant too, as a way to mix existing Scrum practices with the benefits of Kanban, or to transition to full Kanban. "Scrumban" is a very loose term.

While in Scrum you use burn-up/burn-down charts to track the progress of work within a sprint towards a goal, Kanban also offers tools that can be used to check the team. The most-used tool is that of a continuous flow diagram (CFD). In a CFD, you create a line chart showing how many items are in each column in a system at a particular point in time. In an ideal system, the lines should remain flat, with the exception of "done," which should increase. When Kanban is being used against a fixed project backlog, the "to-do" line will decrease, although for many sustaining products (where Kanban works best), work should come in at about the same rate as it gets completed. Figure [2-4](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig4) shows a typical CFD where there is mostly a smooth flow through the system. This example does show a small bottleneck about half-way through, where there is a bulge in QA, but this was remedied by refocusing dev effort into QA (so the dev line goes to zero and touches the QA line).

![../images/471976_1_En_2_Chapter/471976_1_En_2_Fig4_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_2_Chapter/471976_1_En_2_Fig4_HTML.png)

Figure 2-4

A Kanban continuous flow diagram showing mostly smooth flow

If any of those lines are not flat, as is the case in Figure [2-4](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig4), there is a bottleneck in your system that needs resolving to avoid losing continuous flow and people being spread too thin. Figure [2-5](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig5) shows a team that is being asked to do far too much work; to-do is growing much quicker than done, and there is a bottleneck in dev.

![../images/471976_1_En_2_Chapter/471976_1_En_2_Fig5_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_2_Chapter/471976_1_En_2_Fig5_HTML.png)

Figure 2-5

A continuous flow diagram showing uneven flow through dev and a growing backlog

## Continuous Improvement

The way your team goes about planning and tracking work should not be static. It should constantly evolve in response to how your team grows and changes. In Scrum, the end-of-sprint retrospective (or “retro”) was introduced as a way for the team to reflect on their work and how they improved. Other methodologies, like Kanban, don’t explicitly call for a meeting like this, but it is common for teams to engage in retrospectives on a regular basis.

There are many formats for running retrospectives, but the most common ones look back to identify what went wrong, and then come up with actions to address those deficiencies. There are many books and blogs written on the subject of retrospectives, and this is just one way of running them.

Regardless of how you run a retrospective, there are a few principles to take into account. One is known as the “prime directive” of retrospectives: regardless of what we discover, we understand and truly believe that everyone did the best job they could, given what they knew at the time, their skills and abilities, the resources available, and the situation at hand. The other key piece is that the team should feel that the retrospective is a safe place for them to engage in constructive discussion and suggest improvements. The substance of the discussion should also be available to stakeholders who are not in the room, so any potential improvements can be shared, and wider feedback acknowledged. Keep in mind that these last two components can be in conflict with each other, in which case the principles can be put to one side while those issues are dealt with. For example, if adding transparency means people no longer feel they can talk freely, that’s an issue to be addressed directly with coaching so that veil can be slowly lifted, rather than sacrificing the team’s ability to talk openly.

Another important practice is to keep the retrospective focused on things that are under the team’s control. If a sprint went badly due to external factors (for example, company-wide redundancies, or a DDoS on the web site), although it can be cathartic to indulge in a complaint session, it’s not especially productive to do so.

One possible format for a retrospective might be to start off by reflecting on any planned actions from the last meeting, seeing if any were completed, and determining which ones should be carried forward, if any. It’s then often useful to make a simple observation on how team members think the last sprint went, and see if there’s consensus and whether or not the feedback was positive.

This can flag issues to talk about immediately; if not, it’s then a good idea to reflect back on individual factors from the previous sprint that influence what went well and what went badly. Once these specific events are identified, you can cluster them around themes and discuss what caused them to happen, before moving on and identifying ways to either make sure they continue (if they were good things), or avoid them happening again (if they were bad). Throughout the whole process, it’s key to make sure you’re reflecting on the good things that are worth celebrating and continuing, as well as the things to be improved.

The Retromat[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fn1) is a tool that can generate activities within this framework, though you should make sure you adapt them for your team’s particular situation.

Retrospectives are an important part of your team’s work, but it’s important that they aren’t the only time you reflect and practice self-improvement. Some teams have an area of their task board for sticky notes based on observations as they happen, and if some major event occurs, it’s okay to deal with it then and there rather than defer it to a retrospective. However you decide to do self-reflection and continual improvement on processes, you should make sure it works for your team.

The retrospective is one method of self-improvement, but it’s not the only one. Tackling technical debt is another way, and a key concern for any team. It’s often tempting to place technical debt on your backlog as a thing to be solved, but it often never is, as it’s hard to explain its business value to your stakeholders, and it makes it hard to prioritize. There are two ways around this. The first is to ensure you express any technical items in the same form as any other user story. For example, if your tests take a long time to run, and you wanted to experiment with parallelization, it might be tempting to add a backlog item that says, “enable parallel test running.” Instead, you could say:

_As a developer,_

_I want my tests to complete in under 60 seconds, So that I can make deployments quicker._

This is clearly stated in language other stakeholders can understand, and can hopefully be prioritized by your product owner like any other work.

Another approach to tackling technical debt is to apply the Boy Scout rule: always leave the campsite cleaner than you found it. This time, our campsite is our codebase. Every time you work on a new feature, if you come across some technical debt, tackle it immediately rather than deferring it to later. Alternatively, if you know there is technical debt that needs solving, and there’s an upcoming feature that would benefit from that debt being solved, then simply build it into the estimate of that feature, as the two are linked.

Some teams also have “tool time.” If there are common tools used by multiple teams, but your organization isn’t big enough for a dedicated platform team, then it is common to dedicate one dev per week for general maintenance tasks, like upgrading Jenkins, or other minor improvements. If there are different teams in an organization, this can be cross-team work, so all teams benefit. The flipside is that it can then be hard for an individual team to prioritize, in which case each team donates one person for a set period to make improvements everyone can benefit from.

## Prioritization & Estimation

Regardless of how you choose to manage your work, there’s one question you’ll find yourself asking whenever you finish something. “What should I work on next?”

Sometimes it doesn’t take much thought. If the system is down and the servers are on fire, it probably means you should be fixing the critical bug that’s starting fires. Other times, there’s less pressure, and there’s nothing that’s obviously critical. Even worse are times when there are a number of tight deadlines coming up, and a whole bunch of work needed to be done in preparation.

A backlog is essentially a big, ordered to-do list. The concept of the backlog started in Scrum, but at its simplest, a backlog is ordered such that the most important thing is on the top, and you automatically pick that most important thing off. You can then either continue working on it until it’s done, then start the next one, or if something more important comes along, you stop and work on the new most important thing. The latter scenario can cause lots of half-finished work to build up, which can increase wastage, but the former can mean important work is delayed until it’s too late. Minimizing the size of each story or item on your backlog reduces the chance of you either being interrupted mid-work or a new, more important thing being delayed too long.

There are many different ways to prioritize your backlog, but often in a team it’s useful to have a single person be responsible for it, as there often will be as many different ideas on what is highest priority as there are people on the team. The simplest way to mitigate this is to ask stakeholders what they think the most important action is, but with a diverse set of stakeholders you will often get a very different sense of priority from each of them. This can be useful for identifying which items impact a lot of people, though. Another option is to weigh the cost of implementing a particular idea against the impact its execution will have.

It is also acceptable to say “no” and prune the backlog. If an idea is low value and high cost, then it may never be worth doing it. Alternatively, if it has been on the backlog a long time, then it is worth asking whether the idea is even valid anymore; perhaps the original need has gone away or changed so much that it’s no longer useful.

When it comes to determining the cost of an idea, this is often done using a process called estimation. Estimation is often one of the hardest parts of tracking work, as it’s seldom accurate, despite how badly people want it to be.

It’s rare for a development team to be working on something that’s exactly the same as it’s done before, and if it is, it tends to be a small task to do, so estimation will always include an element of uncertainty. Early methods for estimation revolved around predicting the actual time a task would take. This yielded concrete figures that could be communicated to stakeholders, but the confidence level around those figures was often not clear, and deadlines were often missed, leading to a lack of trust on the development team.

Commonly used with Scrum is the concept of story points. Story points are a useful tool for detaching hard dates from the estimates, but have the downside of making it harder to communicate what they actually mean to stakeholders. A story point is a unitless number that is supposed to indicate the size of a story, relative to other stories on the same backlog. There is no absolute guideline as to what the numbers mean, and even within a single organization, the numbers are not directly comparable between teams. Each item on the backlog is assigned a story point value, and then you can add these up to determine how big the backlog is. The velocity is then the number of story points that are completed in an average sprint (often computed by taking the mean of the total story points that were completed in the last three or four sprints), and with the velocity, you can estimate how many sprints it will take for you to either complete the backlog or reach a particular item on it.

The possible values for story points are usually not linear, to recognize that larger tasks are often harder to estimate than smaller ones. Instead, the Fibonacci sequence, or a variant, is used. A story can be 1, 2, 3, 5, 8, 13, or 21 points (sometimes larger, but often when a story is too large it is good to break it down into smaller stories). A good rule of thumb is to imagine the simplest thing for a team to complete—for a web app, this might be something like a small re-design of a button—and then designate that as “1.” You can then estimate the relative complexity and size of an item compared to that. And this is where it becomes clear that story points are not directly comparable between teams. A team with a relatively greenfield project that has good development tools and an effective automated test suite might be able to complete a “1” much more easily than one with a legacy codebase with few tests. It could also be that a team simply has a different idea of what the simplest things to complete are.

Story points can be used in methodologies other than Scrum too. For example, in Kanban, instead of using sprints as the time frame to determine velocity, you can instead choose a fixed period of time and calculate using that instead.

As you start to use story points and velocity to predict projects that are further out, it is important to recognize that the way velocity is computed (usually as a mean) can hide underlying complexity. This is sometimes known as the cone of uncertainty—things that are close can be predicted with a higher degree of confidence than those that are further out. Figure [2-6](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_2_Chapter.xhtml#Fig6) shows an example of this cone of uncertainty. The cone of uncertainty can be used to plot the likelihood of a story’s true value in points being between two extremes, depending on how far the story is from completion.

Story points can be an effective internal tool, but are often dangerous to communicate outside of the team. Stakeholders who are not fully versed in story points may misinterpret them, especially if there are other teams using them. One common mistake is for management to see them and then assign a team a performance target to increase their velocity. Velocity is a measure used to help planning, rather than a measure of a team’s speed or performance. The easiest way to increase velocity is to start estimating higher numbers, which can be fine as long as the numbers stay relative to each other, but as older stories were not subject to this point inflation, it means historial sprints can no longer be used to determine velocity. A more useful technique is to give stakeholders dates. Dividing the velocity by the number of story points that are ahead of a particular item on a backlog can give an estimate as to when something will be completed, but this doesn’t take the cone of uncertainty into account. Instead, you can apply statistical techniques, such as taking a standard deviation of the velocity calculation in order to determine a “high” and “low” estimate of how many sprints are needed until a particular item is finished, and therefore translated into a date range. This can still infuriate stakeholders, as they tend to like concreteness. In this scenario, the worst-case estimate of the two should be used, and then there’s a reasonable chance of delivering early, which will make them happy.

![../images/471976_1_En_2_Chapter/471976_1_En_2_Fig6_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_2_Chapter/471976_1_En_2_Fig6_HTML.png)

Figure 2-6

The cone of uncertainty diagram, showing that the further a thing is from being done, the more uncertainty there is in the value of the estimate

When story points are used, it is important to re-estimate as tasks move through the pipeline. As a story is better understood, what first seems very complicated can become simpler, and what first seemed simple can develop many hidden complexities. Some teams go a step further and use “t-shirt sizing” for their least defined stories. With t-shirt sizing, stories are rated small, medium, large, or extra-large, which correspond to story points for estimation purposes.

Some teams have embraced a movement known as “no estimates,” and, beyond that, “no backlog.” No-estimates teams make no guarantees to stakeholders and promise to work as quickly as possible, simply picking up whatever is the highest priority at any given time. This works well when the backlog is short. No-backlog teams go a step further; the team does not accept work unless it is immediately ready to address it (this is sometimes implemented as a WiP limit on the backlog). This approach is rare, as it can lead to important tasks being missed due to them arising at an inconvenient time, and can make stakeholder management difficult.

In order to determine these story point estimates, a powerful technique known as Planning Poker is used. In a Planning Poker session, everyone is given playing cards (or an app) with the sequence of story points on them. For each item, each individual on the team chooses a card that reflects the story points they would assign that item, sometimes after some discussion of the exact scope and purpose of a story. The cards are then revealed simultaneously—this is done to reduce influence between team members, so a truly representative sample of the entire team is taken. If there is consensus, then that value is used as the story point value. Otherwise, the team can discuss why they may disagree on any values (perhaps someone forgot some technical challenges that needed to be implemented, or they over-complicated the task in their head) and then further rounds of voting occur until a consensus is reached.

Whereas Scrum uses the story points process with a velocity, Kanban uses time as a measure of work through your system. It is therefore helpful to use Kanban with INVEST to try and make all items in your process a similar "size" in terms of work that needs to be done, and any variances get averaged out. To use estimation with Kanban, you must measure how long an item takes between it being created, work starting on it, and then it being completed. The difference between creation and completion is called the lead time, and the difference between work starting on that item and its completion is the cycle time.

Cycle time is the absolute minimum time that a ticket can be delivered in, if work starts on it immediately, and can often be minimized by the team through process improvements. However, the lead time is what external stakeholders care about, as that’s how long it takes for their ideas to be delivered, aside from just the time the work actually takes to do, but also includes any waiting times for the work to be picked up. If a team has a long backlog, then it can mean that lead times are long, which indicates that the team has too much work to do. If the lead time is increasing, that's also showing that the stakeholders are asking too much of the team (the cumulative flow diagram should show this too).

## Managing Bugs

Bugs (sometimes known as defects) are a result of unfinished work, and while there are always likely to be undiscovered bugs in your system, having too many discovered bugs can become crippling to a team.

Many teams track bugs alongside other work that needs to be completed, and this works well for them. Others have separate bug trackers, which can cause issues when teams have to look in multiple places to find bugs.

Like any other pieces of work to be completed, bugs need to be defined. Often this is done by an analyst or tester, with a tried and tested format of:

1. 1.
    
    steps to reproduce
    
2. 2.
    
    expected behavior
    
3. 3.
    
    actual behavior
    

When you’re in a time crunch, it can be very tempting to ignore bugs in favor of hitting a feature deadline, but doing so increases the amount of work in progress. Bugs represent something that’s been missed (either by a developer, or at an earlier stage in the definition process) in a piece of work, and it’s not fair to call that piece of work completed if it is buggy.

Regression bugs are even worse, because they’re taking something away from the user that they have come to rely on, and rather than just stopping us from completing a piece of work we thought we had finished, they add an additional piece of work back into the project that’s currently in progress.

For a full stack web developer, many bugs occur as a result of browser incompatibilities or are device dependent. These are the most annoying bugs to fix, as technically the bug is not in your code (if you’ve written your code according to standard). Since adding a workaround to fix this bug incurs a cost, it is often effective to have a bug triage, where bugs are discussed and prioritized.

Some bugs that surface are simply not bugs at all. Rather than being issues that we decide not to fix, they are things that have been identified as bugs due to a difference in understanding between the person doing the testing and the person who implemented it. These types of bugs can also be discussed in a bug triage session, where the intention can be clarified.

A bug triage should not be a formal meeting, but rather a casual get-together for people on the team to discuss bugs . Scheduling a triage can often delay a fix (for example, having to wait a whole day to implement a 30-minute fix), but for some teams it’s necessary to book time with stakeholders—such as a product owner—to make those decisions. For other teams, a bug triage can be as simple as tagging someone in a comment, or sending an IM.

An obvious bug occurs when a tester identifies something that they believe a developer has missed, and the developer agrees that they have indeed missed something. When everyone is in agreement, there’s no need to go through a formal triage session for this; the developer can fix it and the tester can verify it fairly quickly. A triage should not become a replacement for ad-hoc communication channels within a team.

The main danger with bugs is when they sit languishing for a long time somewhere on a backlog, so this should be avoided. Remember that minimizing work in progress is important, and leaving work open causes cognitive overhead and can also impact the quality of our product. A very effective technique for managing bugs is to either fix them immediately (so bugs always go to the top of the backlog), or decide that it’s not worth ever fixing. The latter can be uncomfortable for some teams, but if a bug is not important enough to fix now, then why would that ever change? Perhaps you can envision a future where there is no high-priority project work, in which case these bugs could be picked up, but if a bug can be left, then it’s often because it only causes a very small or insignificant impact, and in reality, it’s unlikely that new feature developments will ever have a lower impact than that. It’s better to kill off these bugs than let them sit on your backlog. If a bug is important enough to be fixed, then you should do so now, rather than defer it. It’s causing issues on your web site or app right now, and it's clearly worth fixing.

## Continuous Delivery

As touched upon above, qualitative data can be a very effective method of discovering information about the product you’re building. However, it’s very hard to gather qualitative data without a product being out there in the real world. Continuous delivery (CD) is one of many ways to approach the delivery of an idea all the way into production, and is organized around the idea of minimizing the time it takes to get an project into production.

Continuous delivery works like a pipeline. One end of the pipeline is an idea or problem, and the other end is running software. It is an imperfect metaphor, as you will often want to then use the released software to generate insights into your users and the performance of that feature to inform new features. One key difference from other approaches is that software always moves forward through the pipeline. Instead of rolling back, with continuous delivery you "roll forward," so if a release is bad, a version with the fix or with the faulty code reverted is pushed forward, rather than an old version of the same code .

At its core, continuous delivery is about minimizing the time and friction required to make a change to an application. A team practicing continuous delivery should formally define the pipeline of product delivery from idea to delivery, and articulate how something moves from one phase to the next. A pipeline might consist of several phases, such as:

- Discovery, where research is done on an idea to get a high-level understanding of the problem space and what solutions might be necessary;
    
- Definition, where formal acceptance criteria and design work yields the necessary information for development work to proceed;
    
- Development, where the product is built and functionally tested;
    
- Acceptance, where high-level acceptance testing is done and any required stakeholder signoffs are obtained;
    
- Deployment, where the change is put into production;
    
- Verification, where any analytics are investigated to make sure that the change is having its intended impact.
    

A checklist at each phase defines what is needed for a change to move forward. Sometimes these high-level phases are broken down even further—for example, the development phase may have additional technical architecture, build, and code review phases. This kind of flow might seem similar to a waterfall process, but individual items should flow through this pipeline, rather than large batches of multiple items where the definition of the whole batch is done up front. Different stages of this pipeline can involve multiple people with varying skills, so different tasks can be worked on in parallel at different stages, but each task being independent and worked on by different people.

By allowing individual tasks to flow through this pipeline, team members are encouraged to engage in appropriate activities more frequently, based on the idea that doing something more often means you get better at it. Any rote tasks, such as some kinds of regression testing, or deployment ability, or data analysis, are then encouraged to be automated to increase the efficiency of the pipeline. Applying the techniques discussed earlier in this chapter, like INVEST, can also give you items in a suitable form to take through a CD pipeline .

In my experience, continuous delivery works well for teams that organize themselves using Kanban. Although Scrum does not necessarily limit you to only releasing at the end of a sprint, this often seems a natural place to do so. With Kanban, by limiting work in progress, and working on an item until it is complete, then moving to the next one, teams break free of an arbitrary cycle mandated by Scrum, and into a more regular flow of ideas into production.

If your backlog consists of user stories that satisfy the INVEST mnemonic, you will likely see success with continuous delivery. By making each user story small and independent, you can deploy it to production the moment it’s finished, having tested it to ensure that it satisfies the requirement. Making lots of small changes reduces the risk of each individual change and minimizes the time between the work being done and being able to capitalize on that work.

It’s often surprising to realize that there’s a strong link between how you plan and go about your work and how you manage your source code in a repository like Git. There are two dominant approaches to managing source code in the world of software engineering. One is often referred to as feature-branching, and the other as continuous integration (CI), or trunk-based development. Continuous integration is often confused with the set of tools that support the practice, but these tools can also help in feature branch–based development approaches, so it's important to focus on the principles of CI, not just the use of CI tools.

In a world without CI, every time a new feature is developed, a new branch is created in the source control system and all of the work for a feature done in that branch, without having to worry about accidentally destabilizing the mainline or having half-complete work deployed to live. Sometimes there is a branch for each developer. Once the work has been done, or a release is due, which could be several days or even weeks later, the branch is then merged into the mainline. There are downsides to this approach, as it can mean a large number of changes happen at once, and if there are many feature branches, then yours may have deviated from the mainline enough that automated merges cannot happen until you to do some rework. This is sometimes called “merge hell.” If a feature is large, there may be incremental benefits during the development of that feature to the end-users that is not revealed until after the merge, increasing the time between doing the work and the value being realized.

CI was created as an alternative method to address these downsides. Every developer commits to the master branch frequently, usually at least once daily. In a pure application of continuous integration, developers always commit directly to master, but it has become common to use a variant where short-lived feature branches are used as a mechanism for code review. This enables code review before pushing into master, but each branch is very short-lived (often only a few hours) to still enable most of the benefits of continuous integration. One advantage of feature branching before the widespread adoption of CI was that the mainline was always stable and complete, as features were not merged until they were done and tested.

Continuous integration tools, like Cruise and Jenkins, were created to run automated tests on every commit to mainline, to bring that same level of confidence around stability to teams that were using continuous integration. However, feature branch teams also saw the value of these tools, and still set them up to run after every merge, or even on every branch in isolation. Be wary of confusing the concept of continuous integration with the use of continuous integration tools!

Teams that do continuous integration often run into an issue where they want to commit code that is stable but does not necessarily represent a complete feature. It’s common for these teams to commit code that is either “dead” (i.e., only uses tests that exercise whether it works or not, but is not enabled in the actual application), or behind a “feature flag.” A feature flag (sometimes called a feature toggle) is a config setting of the application that indicates whether or not a particular feature is available to the user. In a web app, this might enable/disable various URL routes on the backend, or hide UI widgets that mean code can never be triggered. This allows you to develop in pre-production environments, but not enable it in a production environment until the entire feature is complete. This means that the code is being enabled gradually, rather than all at once, which can be an effective way of managing risk, albeit with the additional overhead of managing it.

For larger refactors, for example, if you’re replacing one feature with another brand new one, it’s common to have both versions of the code in place, and then a feature toggle that switches between the two. Doing things in this way makes it easy for you to do other tests with your code, such as only enabling it for a certain percentage of your user base to test it whilst rolling it out, or to perform A/B testing, where you have two versions live at once and see which performs better by looking at statistics. This is covered in the Constant Learning chapter.

Another concept that is related to continuous delivery is continuous deployment. Continuous deployment takes continuous delivery to the next level. In continuous deployment, every commit (that passes the tests) is actually deployed to the production environment, whereas in continuous delivery, every commit is capable of being deployed to the production environment, but the actual deployment still happens when the team defines it as possible. Continuous deployment requires a complete level of automation from commit to production, as there is no scope to stop for a manual QA phase. Teams that practice continuous deployment often still do manual QA, but often in the production environment itself, rather than beforehand in a QA environment. Continuous deployment is covered in more depth in the Deployment chapter.

## Summary

Effectively planning your work before you do it increases your effectiveness, as it ensures you properly understand the problem you’re trying to solve. Historically, this planning was done formally with a large amount of definition up front, but in many situations that has been replaced by agile methodologies.

Agile methodologies focus on only planning what is actually needed right now, leaving you free to plan or re-plan other parts of your project in a way that reacts to change.

Planning starts with defining a backlog—a prioritized list of what the team will be building—and these items should be structured so that they are not purely technical to allow for them to be prioritized in line with the organization’s goals. Items of work then move through a workflow where additional detail can be added, perhaps by stakeholders in other disciplines such as UX, adding elements such as visual design, until finally it is deployed to your end users. Two major agile techniques are Scrum and Kanban, which take different approaches to tracking and managing work. Both Scrum and Kanban encourage teams to reflect on their workflows and update them often to ensure they meet the unique needs of that team. Where they differ is in how they allow prediction of when future work may be delivered, with Scrum using story points to determine a velocity, and Kanban using the average time of previous feature delivery.

In addition to planning new work, you must also manage bugs that occur during development, perhaps as a feature is being developed, or potentially hidden until later. These bugs can still be treated as backlog items, but capture their requirements in a different way.

Continuous delivery and continuous integration are two distinct but related approaches to managing the engineering work to enable the process of delivering a feature. Continuous delivery applies automation to make it easy for a backlog item to transition through the different stages—from idea to realized and deployed system—and continuous integration is how developers coordinate changes onto one shared codebase.