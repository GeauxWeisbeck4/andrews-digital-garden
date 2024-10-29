---
id: 01JBAQP9V0R6ANR9CH2KKDR8EN
modified: 2024-10-29T09:49:36-04:00
title: Chapter 9 - Pattern Thinking
tags:
  - systems-thinking
  - systems
  - books
  - programming
---
# Chapter 9. Pattern Thinking

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_eb09.png)

> No pattern is an isolated entity. Each pattern can exist in the world, only to the extent that is supported by other patterns: the larger patterns in which it is embedded, the patterns of the same size that surround it, and the smaller patterns which are embedded in it.
> 
> Christopher W. Alexander et al., _A Pattern Language_ (Oxford)

I confess, I am a pattern thinker by nature. When I catch a cold, I think about the factors that contributed to getting sick. Have I been stressed or overtired? Was I exposed to groups of people or someone sick? I compare my symptoms to colds I’ve had in the past and to not-colds like seasonal allergies or Covid. I research, and by “research” I mean Google. Are there new, better treatments available? Would my symptoms benefit from a doctor’s visit? What has helped me get well in the past? I alter my schedule to accommodate a few days of recovery.

Then…I try to “just power through!” My longstanding reactive pattern to getting sick is to ignore it, despite being fully aware of the inevitable outcome. Like the Friday afternoon, early in my software engineering career, when I was brain-foggingly sick and didn’t want to admit it. I pushed a small JavaScript change to production that broke the entire e-commerce process.

I use viruses as an example because they are a good example of systemic patterns.

> The application of complex systems theory to viral dynamics has provided new insights into the development of AIDS in patients infected with HIV-1, the emergence of new antigenic variants of the influenza A virus, and other cutting-edge advances.
> 
> Ricard Solé and Santiago F. Elena, _Viruses as Complex Adaptive Systems_ (Princeton)

What happens in a person’s physical system after exposure is interrelated to factors like physical condition and diet. I have an autoimmune disease that wreaks havoc with my immune system if I eat certain foods. For a long time, doctors were not looking for those types of patterns, so they didn’t make the connection between my diet and recurring illnesses.

Characteristics of each virus plays a role. Your body behaves differently if you’ve been exposed to a virus before. (Sometimes.) Outcomes after exposure are unpredictable. Not everyone exposed to Covid, for example, gets sick. A leading expert, who was waiting in line with me to board a plane at the Reykjavik airport, said that there aren’t just superspreader events—there are superspreader people. Some of us exposed to Covid will transmit it to nearly everyone we come in contact with. Some of us get Covid and no one else gets sick. We don’t know why this happens or even if, by the time you read this, we still understand transmission patterns this way.

In all systems, we watch patterns, and some of those patterns are surprising. This is life in systems. The closer you look, the more complexity you discover.

# What Is Pattern Thinking?

> Systems thinking often involves moving from observing events or data, to identifying patterns of behavior over time, to surfacing the underlying structures that drive those events and patterns.
> 
> Michael Goodman[1](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id705)

Like most phrases in systems thinking, there are multiple definitions of pattern thinking. A common definition of pattern thinking is “Has this event happened before, and were the circumstances similar?” But be careful: if you imagine patterns only along a linear timeline, you will stay entrenched in linear thinking even as you apply systems thinking approaches.

Patterns that repeat are also, often subtly, changing as they repeat. When you are able to spot a repeating pattern and understand the ways the pattern is reinforced and consider the forces reinforcing it (or not) and the ways the pattern changes…you are pattern thinking.

When you look at a pattern, you are mentally isolating it from the complexity in which it operates. This means that the definition of pattern thinking depends on your perspective, the frame through which you look at a pattern.

In software, interest in patterns was popularized by books like _Design Patterns_[2](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id706) (1994) and _Pattern-Oriented Software Architecture_[3](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id707) (1996). We have framed the word “patterns” to describe object-oriented approaches to software development, like the factory pattern or state. We continued to evolve our thinking about patterns, describing relational patterns in software, like client-server and event bus.

Enterprise integration patterns arose with software system complexity.[4](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id708) Interface development using design systems and components is, fundamentally, applying pattern thinking to create reusability and consistent user experiences.

When we talk about DevOps, stream-aligned teams, and microservices architecture, we are talking about patterns.

In systems, we expand the scope of our definition—we use bigger or different frames. System patterns depend on the circumstances, so there are fewer generic patterns we can apply and significantly more observations are needed to understand how the system operates.

Patterns in our software systems are always evolving. At the end of this chapter, I will give you pattern examples using our fictitious MAGO system, which is facing a profound pattern-change problem. You will likely recognize some of the patterns. You are also guaranteed to be facing pattern challenges that are different from MAGO’s.

###### Note

Pattern thinking is not simply learning patterns you can apply. It is learning to discover, discern, and describe patterns in your circumstances. And it is learning to transform patterns to steer a system in a different direction.

# Patterns Produce Events

To illustrate, let’s use the example “There’s a bug in production.” The bug is a syntax error that a linter would have caught. Your team has been extra busy, pushing changes faster than usual. You are missing little things because you have less time for peer review. You decide to add a linting step to the merge process. For a while, this decreases the bugs.

This is an example of pattern thinking: rather than viewing each bug as its own discrete event and reacting to it, you see the pattern—a repeating event with a common feature (syntax errors). You see that the circumstances are encouraging it. You intervene (prevent or alter the course of events) with a relatively small change that improves the system as a whole (by keeping it tidy with linting).

Here, though, is the no man’s land between linear and nonlinear: does linting change the root cause of the problem?

Perhaps. There is nothing wrong with adding a linter and calling it good. We don’t need to overthink everything, all the time, and overly engage in root-cause analysis. Many teams use linters because they are helpful. A valid concern about systems thinking is that it adds conceptual load onto people who are already at capacity. There’s no need to overdo it. You can, however, look deeper into the situation and see what you see.

According to the Iceberg Model, underneath events are patterns, and looking at them is a great first step toward systems thinking. Underneath patterns are structures…forces acting on patterns to cause or reinforce them. Structures arise from our mental models and, as we’ve said previously, if you don’t transform the mental models, you won’t effectively change the patterns.

Why is the team pushing code to production faster? Why has their peer review process become strained and sometimes ineffective? Why wasn’t linting already part of the delivery process; was there a reason?

Perhaps the team was ready to move faster and syntax errors in production simply reflect that they’ve been overcautious. They were perfectionistic in their approach, unwilling to accept that it’s perfectly normal to miss errors in code. We all want to believe that we can write perfect code, but the mind doesn’t work that way. When we are generating something novel, we miss things. Everyone does; it’s unavoidable. A linter might be the safety net that allows the team to take more risks and increase their pace.

Perhaps they had a manager who was reinforcing this perfectionistic structure. The manager’s core mental model was “Good developers write perfect code quickly.” Patterns of blaming and micromanagement were slowing the team down. They were rarely pushing bugs but they were also rarely pushing code.

Now they have a new manager, who encourages them to stretch more and try harder things. They have more bugs because they also write more code. At first, they make more mistakes, but over time, they make fewer mistakes. A linter supports them as they grow. Perhaps they are also improving test coverage, supporting the change in multiple ways. From a systems point of view, for the moment, I would call the root problem solved.

Perhaps the team is under pressure to deliver faster. In my experience, this is the most common cause and not always a bad thing. Teams have crunch times, when they are down in the weeds together trying to hit a launch deadline. Sometimes, despite their best efforts, things get messier than usual. They might tolerate a leftover TODO that they wouldn’t under normal operating conditions. Once the crunch time passes, they tidy up. Meanwhile, they add linting.

In this case, their structures and mental models are flexible enough to adapt to circumstances. Overall, they do excellent work and they know that sometimes it’s harder to find low-level errors than other times. Their work over time is trustworthy, so they don’t worry too much about the crazy days.

More often, though, the pressure isn’t coming from the natural rise and fall of activity in software development. When I ask people to identify the core mental models and structures that lie beneath bugs in production, the most common theme is fear. An organization, for one reason or another, is reinforcing their own chaos. Some examples include the following:

- Believing they are a ship at sea during a storm and constantly fearing they might sink, they drive everyone to row faster.
    
- Believing that they can only gain competitive advantage, be first to market, through sustained authoritarian leadership, bullies are promoted and rewarded.
    
- C-levels are constantly fighting each other for control, which translates into constant drama and distraction for the teams. People begin their week working on one thing, only to be told they also need to work on three other Priority One issues simultaneously—without missing their deadlines.
    
- The frailty in the software system has, over time, created an “always on fire” culture that has become the norm.
    

A sustained core mental model of “Deliver quickly or we die” will rarely deliver anything faster over time. Nobody thinks well when they are constantly on fire. When delivering as fast as possible is the only goal, a syntax-error bug in production acts like oil on the fire of fear that’s already raging. More pressure is put on the team—the bugs are perceived as their fault, not the natural outcome of fearful mental models.

_Adding a linter won’t solve this core problem_. It’s just a Band-Aid. What happens next? Harder-to-spot “bugs” in the system arise from poorly architected relationships, the team gets burnt out, products are quickly brought to market untested by users (who are unpredictable at the best of times). More managerial oversight is added, increasing the conflicting expectations on the team.

This is a vicious cycle I’ve seen time and time again. Whenever we make changes in a system, the impact of the change depends on _why_ we are making that change. What are the mental models and reinforcing structures involved? The efficacy of our changes depends on how well we understand the patterns that are acting on the system and how well our change addresses core mental models.

Over the years, I’ve gotten tired of never-ending solutions that don’t solve the core problems. Perhaps you’ve felt this too? As relational complexity increases, our command and control approaches become more noise than signal. Pattern thinking can help us amplify signal rather than amp up the noise…if we are watching how relationships produce effect.

# How Relationships Produce Effect

> We can’t impose our will upon a system. We can listen to what the system tells us, and discover how its properties and our values can work together to bring forth something much better than could ever be produced by our will alone.
> 
> We can’t control systems or figure them out. But we can dance with them!
> 
> Donella Meadows[5](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id714)

Patterns are the choreography of a system. Everything is in relationship to everything else and patterns we don’t explicitly design will emerge, despite our best efforts to control them. Behaviors in systems arise from relationships among parts and the way those parts share information. Structures, networks of relationships that create behavior, reinforce thinking and behaviors, in software systems and in people.

Pattern thinking is looking beneath the surface of events to discern how relationships produce effects. Relational patterns are, perhaps, more familiar to us outside of technology. You, as an individual, have patterns of thinking and behavior. When you get into a relationship, you, as a couple, have patterns of thinking and behavior that neither of you exhibit alone. Patterns you might discuss in couples therapy. When you have children, the relational patterns have an increasingly complex and long-term impact. Psychologists call this a “family system” because familial relationships produce effects beyond what any one person says or does.

Not just direct effects, but indirect as well. A family system is in relationship to external systems, like employers, grandparents, schools, religious beliefs, and the communities in which the family lives. All of these relationships will influence the way a family thinks and behaves. When you can’t sleep because you are feeling stressed about work and family pressures, you are experiencing how relationships produce effects.

The same is true for software systems. Pattern thinking is understanding how the system of software, the people building it, and the organizations around them produce effects. Pattern thinking is also discerning ways to improve patterns in order to improve the system as a whole.

This is a messy subject to explore because each software system will have unique patterns. And each software system will share common patterns with other similar software systems. At the intersection of these unique and shared patterns is systems architecture—the process of figuring out how to improve relational patterns in a particular system.

When we shift into pattern thinking, we move away from demanding concrete answers to complex challenges and into the whitewater world of interrelationships. When you understand patterns, you understand that everything is in flux. Like a sailor, you learn to navigate toward your destination while respecting the forces acting on you. You can’t ignore the ocean, the weather, or the need to provision regularly. You can discern how to operate wisely in whatever circumstances you find yourself in.

System patterns aren’t usually visible at the code level. Modeling is essential for pattern thinking. Modeling together makes patterns visible and synthesizes disparate views about what happens in a system. I always use a whiteboard (digital or physical) to facilitate discussions about patterns. Collective modeling shifts people away from their usual mindset into the world of bounded shapes and interrelationships.

We can use frameworks to structure patterns. Software frameworks, like Spring Boot, Symphony, or Angular, save us design time by enforcing patterns. In my experience, though, wholesale adoption of any one framework is insufficient to support systems thinking. You’ll still need to model, and understand, the patterns in your system. You’ll still need to synthesize other people’s expertise and experience while designing in a way that will support your situation. Sometimes, that work will lead you away from a framework; sometimes it will lead you toward one.

We more often draw on metaphors to describe patterns, as I do in this chapter. This can be frustrating for those of us who equate metaphor with “too abstract.” I understand that frustration. To understand and model patterns, we need “just enough” abstraction to make patterns visible and sufficient data-driven observations to ensure we are describing reality.

In [Chapter 10](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch10.html#modeling_together), we’ll dive deeper into modeling. Modeling patterns is both an art and a science, an opportunity to generate insight in both traditional and novel ways. As you read this chapter, I encourage you to imagine how you might model the types of patterns we are exploring.

## Same Event, Different Patterns

In the previous “bug in production” example, I showed how the same event can be caused by very different patterns, structures, and core mental models. Here is another example of how the same event can be influenced by very different patterns and sociotechnical structures.

An organization experiences a major, unpredicted outage in a business-critical part of the software system. Everyone who can get the system back up and running is put on high alert. They triage the problem, figure out its root cause, and resolve it. The outage was a rare occurrence, triggered by a cascade of outages in third-party dependencies. There was very little they could do to prevent it, though they recommend some changes to reduce impact if it happens again (which is unlikely).

Conversely, an organization experiences a major, unpredicted outage in a business-critical part of the software system for the 32nd time this year. Everyone is on constant high alert, and leadership wants to know who is to blame. They put increasing pressure on the development teams to write perfect code in a software system that is increasingly frail and unpredictable.

This is a self-reinforcing pattern, but not in the way you might expect. Figuring out the root cause of recurring outages requires deep knowledge and experience with the software system. Because of the blame-driven culture, people aren’t staying long enough to develop that expertise. They leave for roles that treat them more respectfully. The people who stay become reticent to share ideas. Patterns within these outages are clues to the causes, but new developers are re-climbing the same learning curves as their predecessors. The few senior people with sufficient expertise are burnt out and condescending to new developers, so they are avoided whenever possible.

Design patterns emerging in the industry would improve the performance of the system as a whole. The developers are not to blame. The current frailty is caused, in large part, by trying to scale software suited for one paradigm into a world that’s changed. Unfortunately, the long-term engineers reinforce “this is how we do things here” patterns and structures, strongly resisting changes suggested by newer people they feel don’t understand their problems.

The whole system is dead in the water despite constant drama and activity.

HR is told to find top talent, the elusive 10x developer who can solve their problem…but top talent recognizes toxic delivery patterns and steers clear. Top talent also want to build with modern tools. Until the organization changes its core mental models, and builds structures that support cooperative evolution, the outages in production are going to continue—regardless of how many 2 a.m. patches are pushed to production.

## Where to Look for Patterns

Pattern thinking is detective work, watching what is observable and sussing out the (sometimes invisible) connections among events. There are often multiple forces acting on observable events. We use pattern thinking to help us decide which ones have the most impact, under the circumstances.

When there is a bug in production, for example, that event has four important relationships: to time, context, other parts, and the structures it exists in.

### Relationship to time

Does the event repeat? If so, when? Has the repeating event changed over time? Has the frequency changed? Are the bugs in different parts of the code that always reveal themselves during a deployment?

Some patterns help us to improve a system over time. Best practices, when done well, establish healthy patterns for code quality, delivery, and interactions among parts. Some of the same patterns can later degrade the system over time. We see the symptoms of this decline in what we call “tech debt.”

Time is always a factor in systems thinking. As things change, the value of our established patterns change. The best way of doing things yesterday might be the worst thing to do today. Our past selves have generated challenges for our future selves…sometimes we are cleaning up a mess, and sometimes we are pleasantly surprised by the good results.

### Relationship to context

Context is an understanding of the circumstances. A system serves a purpose in relationship to the context. In every context, circumstances are changing. The purpose of Netflix’s system has always been to provide movies on demand. But the circumstances in which it operates has changed dramatically. Imagine the differences between a system designed to mail DVDs to monthly subscribers and one designed to provide streaming movies on demand. Operating well in the context of distributing movies depends on understanding, and responding to, the ever-changing circumstances.

At the software level, we are used to thinking about context. Does an event happen in one context but not in others? A bug that appears only in Safari indicates a relational problem between the source code and the browser.

On a systems level, leverage points, the most-impactful changes to make in a system, completely depend on the context. What works for Netflix or Spotify or Facebook won’t work for any other company, at least not in the same way they worked for Netflix, Spotify, and Facebook. The patterns in each circumstance will be, at least somewhat, different. Also, what works for Netflix, Spotify, or Facebook today might not work for them next year.

What we prioritize depends on context. Some software systems need to be fast; some need to be perfectly accurate; others need to be exceptionally easy to change. We might want all three, all the time, but we prioritize the patterns that matter most based on our context.

### Relationships to other parts

Decoupling and modernization have become common phrases over the last 10 years. These are pattern changes that happen, primarily, in relationships among the parts. You’ll find many resources that help you understand API design, event-driven interactions, microsites, and microservices. Conferences and talks increasingly include domain-driven design[6](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id725) and Wardley Mapping[7](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id726) topics to help us design parts of a software system, and their relationships, to match the context (the domain) in which we are building them. We are encouraged to use domain language to describe technology parts and interactions. Popular books like _Team Topologies_[8](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id727) and _Accelerate_[9](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id728) integrate people relationship design with technology design.

Despite all this attention to relationships between parts, I rarely see project or program management approaches that support emergent systems design. By emergent, I mean the whole becomes greater than the sum of its parts. Generally, we “manage” by breaking down the parts and adding control structures, like SaFE, rather than orchestrating interrelated but somewhat-independent activity.

Control structures are partly necessary: we need to coordinate what an individual works on today with the overall goal of the system. We need relational patterns, in the people and the system, that are, whenever possible, self-organizing and deeply aware of context.

I’ve also seen groups figure out how to apply “just enough” control in the midst of their system design and delivery process. We need to amplify that thinking, wherever we find it, so that we can improve the way we structure relationships among people and technology parts for the systems age.

### Relationship to structures

A pattern is always held in place by structures of thinking, behavior, information sharing, etc. Sociotechnical structures reinforce patterns and keep them from changing. Hierarchical leadership is a structure. Delivery methods and project management processes are structures. Test-driven development is a structure. Templates defining the ways we should communicate thinking (or not) comprise structures.

In a technology system, there is, of course, infrastructure. Infrastructure is deeply involved in establishing and governing patterns in a technology system. It can also be a leading source of hard-to-find-and-fix systemic issues. In systems I’ve worked on, for example, the caching structure was both keeping the system running efficiently and the leading cause of unpredictability.

Decision making is also always interrelated to structure. Here is an example:

Developers working in a legacy system want to build test coverage so they can decrease bugs in production. They invest too much time, they say, fixing problems when their code inadvertently impacts code elsewhere in their Big Ball of Mud codebase. A short-term investment in tests will pay off in the long term.

But decisions about where the team invests their time are made by product managers who are pressured by leadership to deliver more change, faster. Improving the system is rarely prioritized because each initiative has a budget that only covers new development. The organizational decision-making structure leaves no space to resolve these competing needs.

Remember the carboat example? One team wants a car, and one team wants a boat, so the engineers build a carboat, which nobody wants. Groups within an organization are pushing and pulling technology decisions in different directions. How we grapple with decision-making patterns will inherently architect the system. Few organizations structure teams to operate cross-functionally, but that is where the patterns we want to improve usually exist.

### In your own thinking

The most important place to look for patterns is in your own thinking. When I was an engineering lead, I helped build linear thinking approaches that worked for us, at the time. However, as relational complexity increased, I realized that I wasn’t practicing sufficient pattern thinking in my work.

That’s not a bad thing. In the world before “modernization,” my teammates and I delivered code that was well encapsulated by the software’s framework. For example, pushing PHP code into an ever-expanding CMS framework didn’t require as much pattern thinking as designing an event-driven microservices platform. My need for systems thinking reflected the changes in the world around me.

The pattern thinking that enables me to design software in one context is different from the pattern thinking that enables me to succeed in another. Imperative patterns, like setting A as equal to B + C, differ from reactive patterns, like changing the value of A whenever B or C changes. The differences aren’t just in the code; _they are in the entire structure of the system_.

Sometimes, the hardest thing to change in a system is your own thinking.

## Three Types of Patterns

As software professionals, we can shift our thinking toward patterns in numerous ways. One way is to look at the three groups of patterns involved in software development.

### External patterns

Software is being influenced by patterns that exist beyond the boundary of our software-building experience. We tend to imagine that our users have static needs that we can understand and design toward. This is somewhat true. But those needs are constantly being influenced by experiences they are having in the world outside of our domain.

There was a time, for example, when the user interface could be scrunched and boxy, in frames, with limited fonts and perhaps even an animated gif. Now users would view that as an untrustworthy scam. They expect white space, responsiveness, variation in visual shapes, and high-resolution images. Fifteen years ago, a user might wait two minutes, perhaps even longer, for something to load. Now they’d be long gone.

Expectations change as users experience emergent patterns, which weren’t necessarily what any one company determined.

We build software that fits a paradigm, and then, the paradigm shifts. When that happens, our software thinking is especially vulnerable to misidentifying what matters to change. We need more than modern infrastructure tools; we need to think about the mission, purpose, and patterns impacted by external changes.

External patterns can also be social forces that reinforce (or not) hiring practices, the way we define leadership and follow “authority,” who gets money to develop things (and who doesn’t), and how we are all taught software skills in educational institutions. The social culture at technology conferences influences the way teams build software and vice versa. External trends, like DevOps practices, interrelate with our internal patterns.

### Patterns in the technology system

Patterns govern where we put new code. Layers are patterns—the application layer is where we put software logic, and the presentation layer is where we put look-and-feel logic. We use patterns to make our code reusable. We design patterns that structure parts in relationships, services, for example. Patterns are formed by the relationships among those parts, when and how and what they communicate to each other. Patterns are formed by when and how and what they _don’t_ communicate to each other. Events form patterns—every relationship in a technology system is governed by patterns.

Nearly everything related to data management follows some agreed-upon patterns of queries and storage.

### Process patterns

As I’ve said in previous chapters, systems are sociotechnical. We can see this clearly in our day-to-day experience of patterns. What is the process governing delivery? How are decisions made? How are roles defined and bounded? What is the definition of “done”? What tools and structures do people use to communicate?

Nowadays, I see thinking divided into “product” and “tech” in ways it wasn’t early in my career. I’ve seen a huge variety of thinking patterns. Waterfall decisions trickling down from leadership. Self-organizing, cross-functional teams architecting their own software. Weekly releases. Continuous deployment. I’ve seen groups that help and support each other and groups that despise and sabotage each other.

To illuminate people-process patterns, follow the money. If you model how money is allocated to technology initiatives in an organization, you will learn a lot about why the system is how it is. Much of what you experience every day will be related to how money flows.

All of the human aspects of software and systems development are subject to patterns that we can (potentially) consider and improve. This book focuses a lot on process patterns not because they are the most important, necessarily…but they are the ones that will need to change if we want to change other types of patterns.

When we are thinking in systems, are we thinking about external patterns, internal (to the software) patterns, or process patterns? Yes! Systems thinking, for us as software professionals, is thinking about how these three types of patterns _intersect and interact_ so we can discern where to intervene.

Discernment is the key word. You can’t change all patterns at once…you wouldn’t want to, but even if you did, you wouldn’t succeed. Patterns exist as a confluence of thinking, behaviors, mental models, and longstanding socially conditioned structures. They rarely change easily. When we think in patterns, we are looking for small changes that will have a big impact.

Patterns can be hard to find and confusing. To simplify, there are seven questions you can ask that might help you uncover them.

# Seven Pattern Thinking Questions

When you are trying to identify patterns in a software system, here are some good questions to explore, model, and consider:

- _How does information flow?_ How and where is it created, shared, stored, and shaped? If it’s in motion, is it transformed? Does the information change as the context changes? Is this flow monitored? Should it be?
    
- _What are the events_ that happen in the system? When one thing happens, what happens next? What core activities define the system’s purpose? When do these activities happen, and what do they change? (In my experience, changing patterns related to events is an impactful place to intervene.)
    
- _What are the boundaries_ in the system? What are the encapsulated parts, and why do they exist? Do they mirror the capabilities of the domain? How strong (or not) are these boundaries, and how strong (or not) are the relationships among them?
    
- _What are the building blocks_ in the system? Are there components, widgets, modules, classes, or other ways to reuse logic? How do they interrelate and form structures?
    
- _What is the delivery process?_ How are priorities set? Quality ensured? How are decisions about what matters, and what doesn’t get made? What are the core mental models and structures supporting this process?
    
- _How are people organized?_ What are the hiring practices? How are teams structured? What groups exist in the organization, and how do they interact with each other? What are the practices expected from a manager? How is leadership defined?
    
- _How is discourse structured?_ Who is involved in decision making? Are reasons given for decisions? Does information flow across teams? Up and down the organizational hierarchy? Do teams communicate as peers? Do people listen to each other?
    

When there are blockers and stuck places that you can’t see, because of counterintuitiveness, exploring these questions can often help you see what you are missing.

# MAGO: Looking at the Patterns

MAGO is in the midst of a vast sea of pattern changes. As the world became increasingly interconnected, MAGO’s system patterns could not keep pace. For example, in 10 short years, MAGO’s weekly publishing rhythm became asynchronous, 24/7 multi-channel, multimedia content delivery. Software parts and people were quickly becoming both independent and interdependent.

Even though they aren’t a technology company, MAGO teams built a lot of software. But they had little control over the relational patterns among software parts. Over time, an ad hoc collection of mostly dissociated software parts were “glued” together by people moving information by hand.

Nobody in the organization understood how the system patterns worked together to form a whole. But pattern design is where the most pain _and_ the most opportunity lie for MAGO.

## Patterns in Relationship

In relationship to time, the entire organization had a synchronous structure set up to support weekly delivery of content. Digital content, at first, stayed within this structure, including a weekly release schedule.

But as the paradigm shifted, development and delivery schedules became asynchronous. As did user engagement, advertising strategies (as they shifted from static to dynamic), product design approaches, and team interactions. The system became a patchwork of time-driven processes in silos.

In relationship to context, the mission of MAGO hasn’t changed. The system’s purpose—to provide captivating and relevant content that people are willing to pay to read—has stayed the same. But as the contexts in which that mission was accomplished expanded into many realms, including personalized experiences in the browser or app, their ability to understand and track user behavior in multiple contexts has been non-existent.

When someone comments on an article shared on social media, for example, MAGO doesn’t know if that person is also a subscriber or someone who watches videos or someone who reads the newsletter. The relationship patterns between MAGO and readers became disjointed.

In relationship to other parts…well, here’s where chaos reigns. Software parts, and the teams who build them, and the content creators who need them, came online in a haphazard way. When there was communication between software built in-house, by vendors and SaaS solutions, rickety duct-taped bridges were built using exports or batch processes, or someone just did it by hand. There was so much siloing that no one person knew what all the technology parts involved were. Where there wasn’t good encapsulation, the software became a Big Ball of Mud rather than parts in relationship.

In relationship to structure, there was constant intrigue. Budget was given to teams building new products and initiatives that potentially generated return on investment. There was no mental model, yet, for investing in a _digital distribution system_. The organization valued “sweating their equity,” making capital investments in technology that would last 10–20 years or more. They were resistant to seeing themselves as a technology innovator (even as their position in the industry meant that the technology teams were often ahead of the curve). They approached digital transformation as yet another project, another capital investment, rather than rethinking their core mental models. When transformation initiatives failed, which they inevitably did, the people leading them were blamed, and more “management” was added.

For each individual involved in the sociotechnical system, the internal thinking patterns they’d learned over a decade of working in the old paradigm (for many, that was CRUD software) did not translate well into the new one paradigm. People who did make the conceptual leap left for organizations who were leveraging modern approaches to systems design, event-based interactions, decoupling, continuous deployment. New leadership was hired to drive change but struggled to communicate the change to minds looking for solutions that fit into the current structure.

## External, Technology System, and Process Patterns

Over the course of time, these three types of patterns changed around MAGO. The external pattern changes, for the most part, were not triggered by MAGO’s system goals. They represent how MAGO is a subsystem within a broader information system. The technology and process patterns reflect MAGO’s attempts to adapt.

### External patterns

The external situation has changed dramatically, with key impacts in MAGO’s core area of concern. Information flows and changes context much more readily, and the landscape is less predictable.

|Then|Now|
|---|---|
|Information was difficult to get unless you went to the library or bookstore. Timely information was shared on evening news programs, in magazines or newspapers, which had a daily, weekly, monthly rhythm. People waited for it.|Information is ubiquitous and always at your fingertips. Timely information is shared constantly, in streams that readers can’t keep up with. Multiple information sources can be scanned and cross-referenced quickly and for free.|
|Revenue from subscriptions and advertising was dependable and sufficient.|Subscriptions and advertising continue to be revenue sources but MAGO now competes with a world of free content and ubiquitous information. The revenue processes are transmuting so fast, nobody knows what a sustainable and sufficient future for MAGO looks like.|
|Staff roles were coveted and long held. Everyone worked in the office. Teams, even technology teams, were stable, with little turnover.|Staff roles increasingly manage relationships with contractors, freelancers, and vendor teams. The organization has staff all over the world, with only a core group that works full-time in the same office. People stay for 2–5 years on average.|
|Information was organized on pages.|Information could take any length and form and be in any kind of digital relationship with other information.|

### Technology patterns

There has been a shift in technology from patterns that are static to those that are dynamic, asynchronous to synchronous, analog to digital.

|Then|Now|
|---|---|
|Enterprise software with batch export scripts was used to share information to other enterprise software when necessary. Data was stored in multiple data stores.|A system of software reacts to asynchronous events. There’s an increasing desire to stop copying data from one place to another and have a single shared source of truth.|
|Relationships among information parts were structured by a single, hierarchical taxonomy.|Relationships among information parts are dynamic and evolving.|
|There was one technology team; no such thing as a systems architect.|Many teams and multiple systems architects.|
|Analog tools were intertwined with digital tools in the content generation and delivery process.|Fully digital process with automation when possible.|
|No test coverage.|Layers of test coverage.|
|Weekly releases.|Many siloed release processes.|

### Process patterns

Patterns in the process of creating MAGO’s product have shifted from being centralized and consistent to more responsive and distributed. Their linear approaches don’t fit their emerging people system.

|Then|Now|
|---|---|
|The sole focus was on delivering the magazine itself, supported by technology.|The focus is on delivering technology initiatives to support content and reader engagement.|
|Weekly planning meetings took place for both content creators and technology teams. Agile teams delivered in bi-weekly sprints.|Ad hoc team creation and delivery processes with little integration among them.|
|Hands-on production of materials.|Hands-on migration of content and assets across the digital ecosystem.|
|Budget supported better content.|Budget supports better technology.|
|“The way we’ve always done it” still worked.|“The way we’ve always done it” is a blocker.|
|Ad sales were king.|Social media engagement is king.|

## Applying the Seven Questions to MAGO

In order to understand how the patterns have changed, we can use the seven questions to ask “What are the patterns in the world that MAGO now inhabits?” This will help them design patterns to stay viable in that world. I would use these answers to begin modeling systemic relationships that will generate these patterns:

- _How does information flow?_ Information that once flowed in a closed system, or one that could be understood as closed, now flows in an open system. It is consumed by many types of software as it flows.
    
- _What are the events_ that happen in the system? Whenever content or assets are published, there are multiple destinations that shape and consume them. Users are engaging all over the ecosystem.
    
- _What are the boundaries_ in the system? The boundaries in the system match the evolution of digital tools. The website, the app, the social media team. A rethinking of boundaries is critical.
    
- _What are the building blocks_ in the system? Layers, information, and storage are some of the building blocks across the system. Each layer has its own logic and components. The look and feel always needs to match the brand. The information needs to be structured for consumers and context. Storage needs to be fully secure. Some storage needs to be highly performant, and some storage will be rarely accessed.
    
- _What is the delivery process?_ It depends on what is being delivered.
    
- _How are people organized?_ People are organized according to what type of technology product they produce. An app team, a web team, a video team, etc.
    
- _How is discourse structured?_ There is more politics than collaboration. Who has a voice totally depends on both charisma and the power structure they are in.
    

Now that we understand something about the patterns in the MAGO system, where do we begin? There are, in fact, many ways to begin. As long as the approaches are pattern-aware, there isn’t a right one…only options that align MAGO with the patterns they need to succeed.

# Your Practice: The Seven Questions

In [Part III](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/part03.html#part3), you’ve developed a proposition and strengthened the reasons that support it. Next, turn your attention to the patterns influencing your circumstances. Can you find patterns, using the seven questions, that help you expand your thinking about how to improve your circumstances?

- _How does information flow?_
    
- _What are the events_ that happen in the system?
    
- _What are the boundaries_ in the system?
    
- _What are the building blocks_ in the system?
    
- _What is the delivery process?_
    
- _How are people organized?_
    
- _How is discourse structured?_
    

You can write the answer to these questions. You will, however, likely gain more insight if you try to model them. Make pictures of how these patterns work and ask for feedback in areas where you need more information.

# Support for Your Practice: Pattern Thinking Outside of Technology

I have learned valuable lessons about technology patterns from people who are not technologists. My personal favorite book on pattern thinking, and the book that many systems architects geek out about, is _A Pattern Language_ by Christopher Alexander and others. This book has nothing to do with software. Yet Alexander triggered the pattern language movement in computer science, which led to changes in object-oriented programming and Agile development.

Mark Bittman’s _Animal, Vegetable, Junk_ and Michael Pollan’s _The Omnivore’s Dilemma_ (Penguin Press, 2006) describe patterns in our food system. I have used many quotes from Bittman’s book to describe technology system patterns. The overlap is intriguing.

Another personal favorite is _Design Unbound: Designing for Emergence in a White Water World_ by Ann M. Pendleton-Jullian and John Seely Brown. Dr. Pendleton-Jullian has taught at MIT and Stanford and writes about architecture—of literal buildings, not software—yet her thinking reflects many of the challenges we face. Your pattern thinking practice will be well supported by learning about patterns beyond software.

[1](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id705-marker) [“Systems Thinking: What, Why, When, Where, and How?”](https://oreil.ly/8R-Qh) _The Systems Thinker_, accessed May 1, 2024.

[2](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id706-marker) Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, [_Design Patterns: Elements of Reusable Object-Oriented Software_](https://learning.oreilly.com/library/view/design-patterns-elements/0201633612/) (Addison-Wesley Professional).

[3](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id707-marker) Frank Buschmann, Regine Meunier, Hans Rohnert, Peter Sommerlad, and Michael Stal, [_Pattern-Oriented Software Architecture Volume 1: A System of Patterns_](https://learning.oreilly.com/library/view/pattern-oriented-software-architecture/9781118725269/) (Wiley).

[4](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id708-marker) Gregor Hohpe and Bobby Woolf, [_Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions_](https://learning.oreilly.com/library/view/enterprise-integration-patterns/0321200683/) (Addison-Wesley Professional, 2003).

[5](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id714-marker) [“Dancing With Systems”](https://oreil.ly/Y3Uy9), _The Donella Meadows Project,_ August 2012.

[6](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id725-marker) Modeling software to match the domain using input from domain experts.

[7](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id726-marker) An approach to mapping business strategy and value streams.

[8](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id727-marker) Matthew Skelton and Manuel Pais, [_Team Topologies_](https://teamtopologies.com/) (IT Revolution Press, 2019).

[9](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch09.html#id728-marker) Nicole Forsgren, Jez Humble, and Gene Kim, [_Accelerate_](https://learning.oreilly.com/library/view/accelerate/9781457191435) (IT Revolution Press, 2018).

table of contents

search

Settings

[8. Designing Feedback Loops](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch08.html)

9. Pattern Thinking

Add to playlist

Learning Systems Thinking

[IV. Designing a System of Thinking](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/part04.html)