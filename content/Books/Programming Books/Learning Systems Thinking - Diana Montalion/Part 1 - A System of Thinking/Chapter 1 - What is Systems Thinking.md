---
id: 01JB5P5150Z4VJ9300J1H4BSMN
modified: 2024-10-26T20:32:17-04:00
title: Chapter 1 - What is Systems Thinking?
tags:
  - systems-thinking
  - system-design
  - programming
  - books
---
# Chapter 1. What Is Systems Thinking?

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_eb01.png)

> Vision without systems thinking ends up painting lovely pictures of the future with no deep understanding of the forces that must be mastered to move from here to there.
> 
> Peter Senge, _The Fifth Discipline_ (Currency)

You might imagine that when you finish reading _Learning Systems Thinking_, you will have learned to think in systems. Nothing could be further from the truth. Systems thinking is a practice, a perspective, a framework, an emerging language…we could even call it a way of life.

Reading a book about tennis won’t teach you to play tennis. You must go outside and play tennis. It’s the same with systems thinking. Experience is needed to change your thinking. I hope that while you are reading this book, you will go outside and play with systems.

This book describes a system for thinking about systems. Reading it will give you context, guidance, vocabulary, and practices. What does a world in which I “think in systems” look like? How do I navigate toward it?

What do I do there? Why does it matter? What practices, principles, and tools will I need to be successful?

My favorite quick introduction to systems thinking is [Figure 1-1](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch01.html#fig_2_tools_of_a_system_thinker_credit).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_0102.png)

###### Figure 1-1. Tools of a system thinker (Credit: [emmasegal.co](http://emmasegal.co/), illustrator)

For technology professionals, this book is a journey from thinking about software to thinking about systems of software. It is also, perhaps more importantly, a journey away from reductionism and thinking about people and technology as separate entities.

It is a book about thinking. Because what we think, and communicate, is what we push to production.

Many of our approaches to developing software arose in the machine age. We cling tightly to the reductionist and mechanistic thinking they relied on. Forty years ago, our systems were primarily physical and mechanical. We read the printed newspaper, books from the library, letters in the mailbox. When we got lost, we asked a human for directions. In some ways, the type of thinking we needed, and were taught, fit our time.

Now we are in the systems age. We’ve created a vast, interrelated, and interdependent digital landscape of software to replace newspapers and libraries and postal mail and maps. Software shapes our online spaces, stores, and social experiences. Software structures infrastructure for other software.

Relational complexity, the number of variables we need to consider in order to make software decisions, is running riot. As software becomes systems of software, we have reached the limits of our traditional ways of thinking.

> In the Systems Age we tend to look at things as part of larger wholes rather than as wholes to be taken apart.
> 
> Russell L. Ackoff

This book is unashamedly about thinking. About abstract thinking. Many (many, many, many) times in my career, I have heard derision toward thinking (and communication), as if they’re not matterful skills and activities. “That’s too abstract. Make it concrete!”

I’m all for concrete. My house is built on a concrete foundation. But my stove and my bathtub and lawn and garden and dogs are not concrete. Concrete thinking is important, but it isn’t _everything_. These things matter too: Creating concepts that guide our integrated decision making. Synthesizing multiple points of view. Looking under the surface of problems to discover their root causes. Transforming the mental models that structure our legacy software systems.

In the wilderness of systems, there is a lot less concrete.

Systems thinking expands our toolsets as knowledge workers. It steps us outside the constant, pointless culture war about architecture versus engineering as a practice. Systems are nondualistic—the choice isn’t “ivory tower architect” versus “hands-on coder”…the choice is “which tool, practice, or approach will help us understand what we need to understand?”

Can you shift perspective and think differently?

The first step toward systems thinking is willingness. Willingness to see things differently, think deeply, practice self-awareness, and become curious about patterns and complexity. Willingness to change your mind and step into the unknown.

As Morpheus says to Neo in _The Matrix_:

> I can only show you the door. You’re the one who has to walk through it.

Let’s walk through the door together.

# Linear Thinking Is the Default

> [B]lack-and-white logic can sometimes work even for complex systems. But it ignores the ways in which parts interact with one another. Reductionism may serve to explain how a bird flies, but not how a flock of birds move in unison. It may describe internal combustion, but not traffic patterns. It may describe electric patterns in the brain, but not consciousness, and it’s unlikely that anyone or anything—not even the world’s most powerful computers—will ever fully analyze the interactions that make for healthy soil.
> 
> Mark Bittman, _Animal, Vegetable, Junk_ (Mariner)

We are taught to think linearly. Linear thinking is so ubiquitous, many of us don’t recognize it as one type of thinking. We call it, simply, thinking: predictable, rational, repeatable, procedural, dualistic, top-down, and concerned with control. We rely on linear thinking to design, build and deploy, run, and maintain software.

Our experience in software has strengthened our linear thinking skills. Governed by our “if this, then that” causal thinking, we expect software systems to behave exactly as we intend them to behave, in all circumstances. In code, we speak a language designed to be unambiguous.

We expect the people who build software to behave in predictable, procedural, top-down controlled ways. Our preferred communication style reflects these expectations—straightforward, concrete, and concerned with control.

Linear thinking is reductionistic, understanding a whole by breaking it into parts. Object-oriented programming is reductionism. Software architecture approaches divide and conquer, “manage” complexity through modularity, and “decompose” problems into subproblems. We break software (or a system of software) into parts or components. We model boxes with lines between them. We fit people (teams) into those boxes.

When we outgrow that model, we re-create the boxes.

This book does not argue against linear thinking. Linear thinking isn’t bad, and systems thinking isn’t good, like the Witches of the East and North in _The_ _Wizard of Oz_. We can’t operate as knowledge workers without linear thinking. In some circumstances, excellent linear approaches are exactly what are needed! Linear thinking and approaches help us in a lot of ways:

- _Break down a complex_ _problem_ into its component parts so you can understand and solve it.
    
- _Analyze cause and effect._ Find the “bug” causing an unplanned effect. Design a change to produce a desirable effect.
    
- _Imagine the steps_ involved in building something new _and take those steps_, adapting as you learn more.
    
- _Learn new programming languages_, tools, frameworks, rules, and processes. Apply those skills.
    
- _Identify weak ideas_ and implementations, then work to improve them.
    
- _Iteratively build_ new software behaviors, learning from the result.
    
- _Identify and follow best practices_. Change them as circumstances change.
    
- _Test ideas_ to discover where they fail. Track valuable operational data.
    
- _Improve efficiency_ by editing solutions until they are elegantly simple.
    

Linear and nonlinear thinking are interrelated. This book is a matterful addition to your bookshelf because linear approaches cannot resolve systemic issues.

We are in the systems era. As relational complexity increases, we need to think _differently_. Many of our challenges are systemic. We need to expand our skillset so we can think, communicate, and act as healthy systems.

# Systems Thinking Is Nonlinear

> Linear relationships are easy to think about: the more the merrier. Linear equations are solvable, which makes them suitable for textbooks. Linear systems have an important modular virtue: you can take them apart and put them together again—the pieces add up.
> 
> Nonlinear systems generally cannot be solved and cannot be added together. . . . Nonlinearity means that the act of playing the game has a way of changing the rules. . . . That twisted changeability makes nonlinearity hard to calculate, but it also creates rich kinds of behavior that never occur in linear systems.
> 
> James Gleick, _Chaos: Making a New Science_ (Open Road Media)

In a linear world, I would plant seven kale seeds, and 50 to 55 days later, I would harvest seven mature kale plants. When a rabbit nibbles one of my plants, I put a fence around the garden to keep rabbits out. They stay out. When the days become unusually hot, I know growth will take longer, so I adjust my expectations. This is the mindset we use when planning software initiatives.

In my actual garden, I sometimes get nine plants because kale is biennial and I planted some last year. Sometimes I get zero plants. When that happens, maybe it’s rabbits; maybe it’s deer reaching over the fence because their favorite foods are in low supply. Maybe there’s been too much rain or not enough rain or too much heat in May or October. Maybe the soil doesn’t have enough nitrogen or the cabbage worms or slugs ate them. Maybe the birds poked holes in the leaves while eating the snails who were eating the kale.

Most likely, the cause of zero plants is _some combination of these things_, impacting each other in difficult-to-predict ways. This, in its simplest form, is what we mean by nonlinear. Systems are not fully controllable and unpredictable. Relationships among parts impact what happens.

You may have built a single piece of software that, for the most part, operated in a relatively controlled environment. When I did that, I couldn’t always predict what would happen when I pushed a change to production. But for the most part, I could mitigate the risk of unexpected outcomes.

For many of us, myself included, that time has passed. Software is everywhere, built on top of other pieces of software, interacting with multiple information sources, spanning the breadth of an organization with hacky bridges. New features are used in ways developers didn’t intend. Modern software becomes “legacy” three minutes after we launch it. Software operates in ever-changing circumstances and depends on a flow of information in flux.

Some bad news: nonlinear approaches are invariably more difficult than linear ones. Your life is not about to get easier. Does this make you want to throw the book across the room? You already have so much to do. Now I’m telling you that nonlinear approaches involve doing the work _and_ figuring out how to do the work _and_ improving the ways we work _and_ clarifying why the work matters _and_ learning…all the time.

Learning all the time. Thinking deeply and also thinking deeply about thinking.

The good news is: you will become _more_ _effective_. The work won’t feel so much like work, which means you’ll have more energy. Systems thinking _improves your capacity for doing difficult things._

Nonlinear approaches increase signal and decrease noise.

In the world of information systems, we’ve built an entire digital ecosystem in less than 30 years. Software governs most of our human communication processes. That ecosystem is emergent…the sum is greater than the parts. Behaviors and trends arise from the relationships among the parts. The “like” button didn’t just give Facebook users a quick way to comment; it transformed social constructs in unpredictable ways.

We are doing difficult things. We can’t design nonlinear systems with linear thinking. For that, we need to expand our thinking toolset. We need to think in systems.

# What Is Systems Thinking?

> For those who stake their identity on the role of omniscient conqueror, the uncertainty exposed by systems thinking is hard to take. If you can’t understand, predict, and control, what is there to do?
> 
> Donella Meadows, [The Donella Meadows Project](https://oreil.ly/M6NwO)

Nonlinear thinking is expressed in a myriad of forms. This book is called _Learning Systems Thinking_. That phrase, “systems thinking,” has been defined in numerous, sometimes contradictory, ways across technology, business, and academia. The vocabulary to describe it is still emerging. Nonlinear thinking integrates more than systems thinking. Strategic thinking, for example, is systems thinking plus creative navigation toward change. Pattern thinking, parallel thinking, and systemic reasoning…all mean thinking in systems.

Learning teams are nonlinear thinking teams.

To some extent, it doesn’t matter whether or not we share an unequivocal definition. The practices and the vocabulary we use to describe them are evolving. The definitions here will get us started. It’s okay if they morph over time.

To define systems thinking, let’s first consider what “system” means. Here are some definitions:

- A set of things working together as parts of a mechanism or an interconnecting network. ([Oxford English Dictionary](https://www.oed.com/))
    
- A system is a whole that consists of parts, each of which can affect its behavior and properties. The parts are interdependent. ([Russel Ackoff](https://oreil.ly/w8o6u))
    
- An interconnected set of elements that is coherently organized in a way that achieves something (a goal). ([Donella Meadows](https://oreil.ly/9qjGG))
    
- An arrangement of parts or elements that together exhibit behavior or meaning that the individual constituents do not. ([The International Council on Systems Engineering](https://oreil.ly/m5Obl))
    

A system is also the set of principles or procedures according to which something is done; an organized framework or method. This is important because systems are sociotechnical. The principles and procedures people use and the framework or methods that structure our work are interrelated and inherently design our systems.

However else we define “system,” it includes the ways we structure our thinking and approaches.

As software professionals, we are concerned with _software_ systems. So for our purposes:

> A system is a group of interrelated hardware, software, people, organization(s), and other elements that interact and/or interdepend to serve a shared purpose.

Caution—this clear description gets muddy fast. Defining “purpose” depends on your point of view. Every piece of software serves a purpose. WordPress is a digital publishing tool. But that’s not what I mean by purpose. Purpose is a property of the whole and doesn’t inhere in any one component. What mission do the elements serve?

Imagine an organization whose mission is to tell the world about the health benefits of plant-based cooking. WordPress might be an element in that system. Other parts like social media publishing tools and Google Ads and an asset manager and recipe generation platforms might also play a role. The writer’s labor in developing the information is an element in the system. As are the readers. Without the inputs (content) and outputs (consumers), the software doesn’t serve a purpose.

Now that we’ve defined “system,” what is “thinking”? Thinking has a surprising variety of definitions. When we are _systems thinking_, we use our minds to reason about something. Which is an interesting sentence…if I am using my mind to think, who is the “me” using my mind?

We aren’t going to open a philosophical conundrum, delightful as that might be. But I do want to make the point that systems thinking is not only thoughts but also your awareness of thoughts. Thoughts that are intertwined with experiences, memories, judgment, and sensory information (like the look on someone’s face).

In this book, we extend the definition of thinking to include structuring and communicating those thoughts. Whiteboarding is thinking. We don’t simply think about code, we write code; we don’t simply think about systems, we make artifacts. An artifact can be a document or a model, a Slack message or a conversation, anything that conveys thinking. In this way, systems thinking is hands-on.

What, then, is systems + thinking? Here are two “systems thinking” definitions:

> Systems thinking often involves moving from observing events or data, to identifying patterns of behavior over time, to surfacing the underlying structures that drive those events and patterns.
> 
> [Michael Goodman, in Systems Thinker](https://oreil.ly/2HDdj)

> [Systems thinking] recognizes and prioritizes the understanding of linkages, relationships, interactions and interdependencies among the components of a system that give rise to the system’s observed behavior. Systems thinking is a philosophical frame, and it can also be considered a method with its own tools.
> 
> [The Alliance for Health Policy and Systems Research](https://oreil.ly/JDTCb)

In this book, I am defining systems thinking as a _practice._

###### Note

Systems thinking is a system of foundational thinking practices that, when done together, improve nonlinear thinking skills.

# Systems Thinking Is a Practice

The practices in this book don’t represent the Definitive List of All Necessary Practices. Systems and nonlinear thinking are evolving, especially in the world of software systems, where we are just beginning to apply them. These practices are an excellent starting point, and you will discover more on your journey.

How we categorize the practices you’ll find here doesn’t really matter. For example, they blend “hard skills” and “soft skills” (in my experience, the soft skills are harder). Practicing will expand the thinking tools in your toolbox. Over time, practicing will improve your ability to apply these tools to “wicked” problems (problems with many interdependent factors that are in flux).

Systems thinking is a practice that happens in your own mind and among people. It restructures the thinking and communication processes around you, like the organizational structure at work. Practicing systems thinking inside of a linear-thinking organization is helpful but not transformational. We practice systems thinking alone, with others, and as a leadership practice.

The four parts of this book, and the chapters within each, are designed to introduce you to the capabilities of a systems thinker.

- [Part I](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/part01.html#part1): Basic concepts and practices
    
- [Part II](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/part02.html#part2): Working with your own mind
    
- [Part III](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/part03.html#part3): Working with input from other people’s minds and the system itself
    
- [Part IV](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/part04.html#part4): Leadership in a nonlinear world
    

Through practice, you’ll develop qualities that will enable you to do hard things with others and empower you to increase your value as a knowledge worker. We are all facing systems challenges—we need each other to figure out how.

# Qualities of a Systems Thinker

There are no “12 Steps to Systems Thinking and the First One Will Surprise You!” listicles that will teach you everything you need to know in 10 minutes. There are no systems thinking whiteboard tests to pass or certification exams. You never finish learning about systems; they will teach you forever. So how do you know you are becoming good at systems thinking?

As you practice systems thinking skills, you will take on more complex challenges. With each new challenge, you will discover there is much more to learn. The paradox in systems thinking is that the more you know, the more you know that you don’t know. We could say that the most valuable quality of a systems thinker is they know they don’t know.

Here are some recognizable patterns and behaviors in people who think nonlinearly. People who are good at systems thinking regularly engage in some key actions:

- _Practice thinking._
    
- _Recognize the difference_ _between_ reductionistic, analytical thinking _(linear) and_ taking a systemic perspective _(nonlinear)_. Discern when to apply one or the other (or both).
    
- _Describe solutions that are meaningfully connected to the context_ and system-level purpose.
    
- _Know that people systems are inextricable from technical systems_. View challenges as inherently sociotechnical.
    
- _Shift perspective easily to explore challenges_ from different points of view. Comfortably engage shifting mental models as circumstances change.
    
- When faced with recurring problems, _seek to understand the systemic structures_ and feedback loops that block change.
    
- _Demonstrate high levels of self-awareness_ and metacognition, especially about thinking patterns that are reactive, fallacious, or biased.
    
- _Avoid adding noise and blame_, recognize reactions, and shift toward responding.
    
- _Approach life with an “always learning” mindset._ Structure discovery, learning, and exploration with others proactively.
    
- _Communicate well-reasoned ideas, recommendations, and theories_. Articulate the reasoning behind conclusions.
    
- _Listen respectfully_ and helpfully work with others to strengthen collective insights.
    
- _Understand how interrelated and interdependent parts act together to create patterns and processes._ Investigate the ways those patterns are reinforced.
    
- _Create conceptual models_, alone and with others, to guide impactful decisions. Have sufficiently diverse modeling techniques and use them improvisationally.
    
- _Accept_ that _uncertainty_ is a natural, welcome, and inevitable part of life.
    

As a systems architect, I do systems thinking full time, but you don’t need to take that path. You can develop systems skills to complement your software-thinking skills. A complementary career ladder for most software professionals might look something like [Figure 1-2](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch01.html#fig_3_a_career_ladder_for_systems_thinkers).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_0103.png)

###### Figure 1-2. A career ladder for systems thinkers

Systems thinking is a mindshift, away from linear and reductionist approaches toward nonlinear ones. The “right” way to achieve this mindshift depends on your skills and circumstances. There are many right ways.

Chances are, you’re already shifting. If you’ve adopted microservices or continuous integration or other interrelated software approaches, you’ve experienced some of this mindshift. If you do collective modeling, connect what you build to why it’s valuable, or think about the relationships among software parts, you’re thinking about systems. If you’ve improved patterns and processes in order to “fix” a recurring problem—systems thinking.

If you find that ambiguity troubling, buckle up, because systems thinking is full of ambiguity. You won’t be careening around blind corners recklessly, though. You are shifting your expectations and enjoying the journey without always needing to know exactly where you are going.

When software systems are changing, software delivery skills like Go or AWS implementation tools or hypermedia API design are valuable and necessary. But they do not predict success. Brilliant software developers, product managers, and strategic leaders are often sincere and hard working, yet making little progress. We are all blocked by two obstacles:

1. We are spectacularly terrible at nonlinear thinking. We are constantly tangled up in our opinions, cognitive biases, fears, assumptions, conditioning, and logical fallacies. It takes practice to skillfully and consistently untangle ourselves.
    
2. We don’t know that we are terrible at it. In fact, the worse we are at nonlinear thinking, the more certain we are that we are good at it!
    

Therein lies a paradox: we must be good at nonlinear thinking in order to see that we aren’t good at nonlinear thinking.

Fortunately, as Carl Jung said:

> Only the paradox comes anywhere near to comprehending the fullness of life.

# MAGO’s Quandary

Throughout this book, I will describe a fictitious organization with increasingly common, real-world systems challenges. Meet MAGO.

For decades, MAGO published the most popular, internationally distributed magazine in the world. Their state-of-the-art publishing software and distribution system ensured each edition reached millions of people worldwide, on time, every Sunday.

In 2010, MAGO launched a website. Over the next few years, everything in print was also shared digitally, their articles, graphics, ads, and collections highlighting timely topics. Like most organizations, they built their digital presence with a single piece of software and then extended, customized, and scaled it by adding lots of caching. They encouraged the world to come view the information available at a URL. Page views quickly rose to millions per day.

MAGO, like most organizations at the time, translated a printed page into a web page. Their data architecture, delivery workflows, and software choreography revolved around the concept of a “page.” HTML was invented to structure digital pages. MAGO didn’t realize that the emerging global information system that is the internet would shift the paradigm. Today, what does “page” mean in a world where people share content everywhere?

An entire ecosystem of organizational infrastructure arose during this shift. Decoupling the subscription workflow, tracking and analyzing reader behaviors, monitoring system availability, morphing assets (like images) into varying shapes, sizes, and types. Business processes were digitized, as were payroll, hiring, and events management (something MAGO was known for). Communication tools became inextricable from productivity, enabling distributed teams to meet, plan work, and track progress.

The world around MAGO’s website became increasingly interconnected. They needed to also show up on search engine results, social media platforms, news aggregators, and video and audio platforms. The world, not just the website, needed a steady diet of content. Discussions about published articles, which MAGO hoped to track, moved away from comments on a page and happened wherever people chatted.

People accessing information on their desktops wanted different information than people on their phones. MAGO built an app, then another. Soon that wasn’t sufficient. People expected information to morph in the browser depending on the device they used to access it or where they were geographically. The demand for information _in context_ was quickly becoming the norm.

As bandwidth increased, so did the demand for multimedia content. MAGO expanded staff to create extended video stories, podcasts, and custom interactive graphics to show data trends. Each of these innovations required software systems to support them.

The biggest change was the increase in asynchronous workflows. Publishing no longer fit into a weekly, or even a daily, rhythm. Various content and the software supporting it went through various delivery workflows. More people needed to keep these workflows in sync as the foundational systems structure, delivering pages, was quickly eroding. Information was increasingly ubiquitous, and the shape of information was transmuting. In fact, the shape of everything was changing.

To keep up with the relentless pace of modern technology, the MAGO team built a lot of software. As it became increasingly difficult to continue extending the original, now legacy, digital software, product people went rogue and built separate websites for new content. The software created, over time, an ad hoc collection of mostly dissociated parts. Very few people could name all the parts or knew how they functioned. The system looked like a [Rube Goldberg machine](https://oreil.ly/L92o6), with production people moving data by hand (see [Figure 1-3](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch01.html#fig_4_mago_s_legacy_software_works_just_barely_most_of)). It emerged that MAGO was facing a Quandary.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_0104.png)

###### Figure 1-3. MAGO’s legacy software works, just barely, most of the time, but it’s held together with duct tape and chewing gum

As relational complexity increased, it became clear that continuing to expand the current system was not going to work. How many pieces of software can be duct-taped together? They could effectively ignore this problem—until the pandemic. Six months after lockdown began, the maintainers of their core software went out of business.

MAGO’s initial reaction was “replace the software now!” But the company went out of business because organizations no longer buy the 20-year-old software that MAGO relies on. The paradigm had shifted. Rather than keep going the way they’d been going, MAGO’s Quandary became: How do we design a system that meets the demands of the modern systems age?

As we explore systems thinking and nonlinear approaches, we will come back to MAGO’s Quandary.

If you happen to know that I’ve architected systems for organizations like _The Economist_ and the Wikimedia Foundation, you might wonder if MAGO is a thinly veiled tell-all. While the fictitious examples you’ll read in this book are inspired by real-world experiences, they reflect modern patterns and challenges faced by many, if not most, information systems. Any resemblance to actual people or specific organizations is simply because everyone is trying to figure out how to evolve into the systems age.