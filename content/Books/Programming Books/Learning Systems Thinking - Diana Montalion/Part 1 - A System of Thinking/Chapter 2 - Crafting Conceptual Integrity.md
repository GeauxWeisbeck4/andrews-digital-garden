---
id: 01JBAPY4DE85GYHGSQM3QK3TZN
modified: 2024-10-28T19:22:08-04:00
title: Chapter 2 - Crafting Conceptual Integrity
tags:
  - systems-thinking
  - systems
  - programming
  - books
---
# Chapter 2. Crafting Conceptual Integrity

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_eb02.png)

> Conceptual integrity is the most important consideration in system design.
> 
> Frederick P. Brooks Jr., _The Mythical Man-Month: Essays on Software Engineering_ (Addison-Wesley)

Our ideas design our systems. When an idea floating in our stream of consciousness takes shape, becomes meaningful, matterful, helpful, or relevant, the idea becomes a concept. Whether we recognize it or not, the coherence and interconnectedness of our concepts shape our technological systems.

Concepts are our primary tool in systems design. Everything running in production represents our concepts—the ideas we prioritized, communicated, structured, and adapted with others, then crafted into code. Concepts also structure the way we think about the technology systems we encounter or inherit. If we want to change what is running in production, we need to first change our concepts, the way we think about what is running in production.

When our ideas are cohesive and in good relationship with each other; when they are supported by healthy, shared patterns and principles; when we push code changes that improve the system’s ability to serve its purpose, we create conceptual integrity.

Concepts become actionable when people in an organization communicate them. Ideas get into production through a process of structured communication. Sometimes that structure is two engineers whiteboarding; sometimes it’s a RACI model. The impact of that structure is massive. As [Conway’s Law](https://oreil.ly/rFa1g) states, “Organizations, who design systems, are constrained to produce designs which are copies of the communication structures of these organizations.”

When our concepts can’t work together in harmony to serve a purpose, our software systems will reflect that lack of integrity. Integrity is the measure of how well a system operates as a whole. Linear thinking has skewed this measurement toward prioritizing efficiency and profit generation. Efficiency is important, but it causes systemic problems when it’s dissociated from the impact. For example, an efficient and profitable system that generates poverty for workers or pollution for the environment _includes_ those aspects of the system. Conceptual integrity is understanding the whole system, not just the parts we optimize.

In _The Mythical Man-Month_, Fred Brooks describes the lack of conceptual integrity as a software system with “many good but independent and uncoordinated ideas.” Many good but independent and uncoordinated ideas…describes nearly every software system I’ve encountered.

When conceptual integrity is low (or missing), you might see situations like the following:

- We can’t share data across the digital ecosphere because of silos. (In the technology system and the people system.)
    
- Software is wired directly to other software with that one Python script someone wrote 10 years ago. We can’t break that chain, ever.
    
- We can’t work effectively across teams or roles. Teams openly distrust (or even dislike) each other and defend silos. Patronizing communication is considered normal.
    
- We can’t “do DevOps” because the legacy software has become a giant ball of mud with painful weekly (monthly, quarterly, yearly) releases.
    
- Product teams avoid building things “the right way” because it’s too hard. They’ve built seventeen “side products” that are duct-taped together. Three of them are doing the exact same thing for different teams.
    
- There is no shared, semantically meaningful data structure among the software. One database talks directly to another database, skipping the logic (software) layer because, well, that was the easiest thing to do.
    
- It’s difficult to tell what the parts are there to _do;_ there is no domain language in the software.
    
- Technical debt is how we describe the lack of cohesion in the system.
    

Conceptual integrity means that parts of a system are in good relationship to each other. What is a “good” relationship? It depends. On the spectrum between a tightly coupled monolith and decoupled software silos, there exists the land of ambiguity. In that space, there is more than one “right” answer, and it’s difficult to know what is “right.” Conceptual integrity helps us design interdependence, information sharing, and patterns that shape system dynamics by helping us discern what might work, right here, right now, for this system.

A microservices architecture that solves a problem for one organization might cause a disastrous problem in another organization. You can’t do what Netflix does and expect the same results. Information systems that worked brilliantly in 2019 became totally obsolete in 2020 when the pandemic hit. Conceptual integrity helps us discern, to stay open to multiple interpretations, yet still design and build relationships in systems.

Relationship design is systems design.

# Relationships Produce Effect

> You think that because you understand “one” that you must therefore understand “two” because one and one make two. But you forget that you must also understand “and”.
> 
> Donella Meadows, _Thinking in Systems: A Primer_ (Chelsea Green Publishing)

Donella Meadows defines systems thinking as how “parts together produce an effect that is different from the effect of each part on its own.”

_Relationships produce effects._ Software becomes a system of software when “parts together” achieve something that could not exist without the “together” part. When “parts together” no longer fit our circumstances, we usually can’t simply change a piece of software. We need to change the way software interrelates.

Creating conceptual integrity involves understanding the effect produced by relationships. For example, there may be two pieces of software that each does something valuable, but the relationship between them produces a bottleneck in the system.

When we think about systems, we consider some key things:

- How do the parts interdepend?
    
- How do they share information?
    
- What are the patterns that keep the relationships functioning?
    
- What are the patterns that block the system from evolving?
    
- Can we improve the relationships and patterns to deliver the highest-value outcomes?
    

Linear thinking can have a major negative impact on conceptual integrity. Reductionism, breaking down complexity into parts in order to design, build, and control those parts, inherently reduces our understanding of how relationships produce their own effects. Interdependence, information sharing, and patterns shape our system’s dynamics as much as the parts themselves.

Let’s use a neighborhood as an example. We can map property boundaries and show that they contain structures like houses and X number of people. We can label the spaces providing services, like a police station or elementary school. Will this map help us _understand_ the neighborhood? Will it help us predict how the neighborhood will change over time as circumstances change?

In a neighborhood, infrastructure like water and sewer lines are in relationship to the ecosystem (like wildlife). Local agriculture feeds (or not) the people and impacts the system design (where to put grocery stores, for example). Our concepts are in relationships with each other…the demographics of a neighborhood, the groups people form by proximity, will influence the way a neighborhood grows. Shared language, for example, can form a boundary around a neighborhood that won’t be visible on a map.

All of these relationships form circumstances, and those circumstances depend on patterns. When events change those patterns in unpredictable ways, the parts of the system are impacted in unpredictable ways. A sudden event, like the closing of a factory, can trigger complex systemic problems that don’t have a quick and easy solution. Events that happen more slowly, like decreased demand for local services or gentrification, have a similar impact over time.

Changes can be restrictive, like the loss of water access. Or expansive, like improvements in digital connectivity. A change in traffic patterns turns a quiet area into a noisy one. A global pandemic impacts neighborhoods designed for commuters. Interdependent changes will transform a neighborhood in unique ways. None of these changes are shown on the property model.

Events transform a neighborhood _because of the relationships among elements._

When our software systems don’t behave as we intended, too often we consider that a failure. Yet, in my experience, unexpected results are far more common than “everything worked perfectly on the first try.” Why not embrace the fact that systems will surprise us? If we pay attention, the surprises can also teach us. In software systems, relationships are driving the digital revolution. Like all revolutions, danger and opportunity are working together to serve a purpose…conceptual integrity helps us keep our attention on that purpose.

# Systems Thinking Is Sociotechnical

> So, what is a system? A system is a set of things—people, cells, molecules, or whatever—interconnected in such a way that they produce their own pattern of behavior over time.
> 
> Donella Meadows

For many software professionals, the biggest mindshift when thinking in systems is also the most important one: software systems are sociotechnical. When I say our systems are _sociotechnical_, I mean that our thinking, behaviors, and communication patterns are inextricable from the software systems we produce. I mean that the relationships among our mental models, our ideas about how the world works, powerfully influence how we act. Our software systems reflect what we think we know. And we also have to consider external factors, the world around us and cultural pressures.

###### Note

You can’t improve the technology system without improving the people system. And vice versa. From a systems perspective, they are one and the same.

Thinking in systems, developing conceptual integrity, includes the whole ball of wax, not just the code running in production. As you’ll see in [Chapter 3](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch03.html#shifting_your_perspective), underneath our decisions about code is a world of patterns, structures, and mental models influencing us. We are also tricked by cognitive biases and logical fallacies, as you’ll see in [Chapter 7](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch07.html#collective_systemic_reasoning). Our thinking is influenced by code we’ve already delivered! Patterns of behavior in the software system impact the way people think about the software.

Three microservices running in Google Cloud are not necessarily a system. By themselves, they are a collection of software. A collection becomes a system when there is a relationship among the parts.

A team is a collection of people. A team becomes a thinking system when there are relationships among the team members. The team is also in relationship with other teams and the organizational structure. There is a relationship between the team and the mission they serve in the world. There is a relationship between the team and the software they build (including the languages and tools they use to build it).

Linear thinking doesn’t simply describe how we think about code. It shapes what we expect from people too—that is, we expect them to be predictable, rational, repeatable, procedural, dualistic, and concerned with control. People’s ideas are rewarded, punished, adopted, or ignored depending on how well those ideas match the organization’s ideas about “how we do things.”

In a traditional, linear, top-down approach to change, “leadership” thinks about and communicates strategies that are implemented by teams. Reality, though, challenges this approach. For example, without input from the team, how does leadership know how long something will take to build? Deciding that a new feature must be delivered in three days doesn’t make it possible. “Agile was invented because reality refuses to bow down to power” (Joe Eaton, systems engineer).

Reality exists in the relationship between the strategy and its real-world implementation. Vertical hierarchies don’t leave space for uncertainty, for understanding how strategy and implementation are integrated into an impactful change. Sometimes you change one thing and all hell breaks loose. Sometimes you give users the feature they say they most want and nobody uses it. We can’t be certain how a change, in the midst of relational complexity, will play out.

Say a service that returns critical information is too slow. The team adds autoscaling to make the responses faster under load. It works as intended until the third-party software that the service uses to translate time zone data stops responding. The interdependent software’s database couldn’t return asynchronous results fast enough. The critical information isn’t merely slow now—it’s dead, until a fix is applied.

Initiatives like “digital transformation” or “modernization” or “monolith to microservices” can’t be a (strictly) top-down initiative because they aren’t linear changes. If an organization tries to design and deliver a system inside of a linear thinking structure, it will struggle, and likely fail, to deliver lasting change. It’ll deliver _something_, but that something will lack conceptual integrity.

Here’s a metaphorical example: One group in an organization wants a car. Another group wants a boat. Rather than resolve these different perspectives at the systems level, both groups push their new product. The engineers are told to build a carboat. Everyone hates it; nobody wanted a carboat.

I’ve seen _so_ many carboats. The two groups needed a systems-level change, but there was no process for reconciling their needs into an evolving systems design. Only a battle of wills that, from a systems perspective, everybody lost.

Some groups flip the top-down approach and adopt a strictly bottom-up approach. Individuals and teams should do whatever they want; they are the experts. Bottom-up communication structures, by themselves, aren’t a magic bullet. Anarchistic approaches potentially create the antithesis of conceptual integrity, “many good but independent and uncoordinated ideas” (Brooks).

In between top-down and bottom-up is systems design. Empowered teams aren’t empowered by total control and independence; they are empowered by the ability to self-organize collaborative work. To share knowledge in ways that enable them to produce a positive impact on the system as a whole. In non-human systems, self-organizing systems develop hierarchy to serve the needs of subsystems. Hierarchy, in systems, is there to improve information sharing. We can learn a lot about sociotechnical systems by studying other systems.

The book _Bad Blood_ by John Carreyrou (Vintage) describes the limitations of human hierarchies. In this true story of a billion-dollar fraud perpetrated by a Silicon Valley startup, there are familiar social patterns at scale:

- A disconnect between engineering reality and leadership demands. Communication runs down the hierarchy but not up. Essential feedback loops are missing.
    
- So many communication silos.
    
- A disconnect between PowerPoint presentations and reality. Product-driven demands reflect no concern for the system as a whole.
    
- The inability to solve complex problems together because trust, psychological safety, and effective feedback loops, all necessary for systemic reasoning (which we will explore in [Chapter 7](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch07.html#collective_systemic_reasoning)), are missing.
    
- Performance is measured by aggressive monitoring of hours worked rather than actual value produced. Forceful pressure is applied to “inspire” people to work harder or deliver faster.
    
- Having to defend novel or unwelcome recommendations to near exhaustion is normal.
    

Most of us have not experienced anything as extreme as Theranos, the company described in _Bad Blood_. Theranos exhibited all the harmful patterns, all the time. But many familiar daily-life norms in technology work, the fact that we think of them as reasonable, masked the harm that these norms can create at scale.

In my experience, successful teams that effectively deliver difficult changes have also been enjoyable teams. The ability to think well together is not only energizing and productive; it’s a business-critical skill. “Psychological safety allows for moderate risk-taking, speaking your mind, creativity, and sticking your neck out without fear of having it cut off—just the types of behavior that lead to market breakthroughs” (Laura Delizonna, Stanford University).

Exhausting teams, overfocused on power and control, have been my worst career experiences, not because they were difficult to work on (they were) but because they delivered very little meaningful change.

We inhabit, you could say we embody, the systems we develop. Small changes in the way we think and communicate scale to big changes in the software system.

That’s how systems work.

# Counterintuitiveness

When we want to change a system, we aren’t looking for “fixes,” we are looking for leverage points. Leverage points are places in a system “where a small shift in one thing can produce big changes in everything,” [as Donella Meadows puts it](https://oreil.ly/4V8f-). We’ll explore leverage points in [Chapter 10](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch10.html#modeling_together)—there are topics we need to explore before we dive in too deeply. Finding leverage points is the most difficult, and most powerful, practice in systems thinking.

For now, let’s imagine that you’ve discovered that a relationship between two teams, and the services they develop, is blocking an impactful change. For example, the two services share information by directly coupling a relational database. Other services need this information, and more tight coupling is planned. You realize that if you structure the information using semantically sensible keys and then share it via an API, you will enable other services to make use of the information they need. This change will bring new services online quicker, services that meet the system’s emerging needs. Plus, as a bonus, you can replace legacy migration scripts with a tidier and more sustainable process long term.

You tell everyone, “Hey! Look! This idea will solve a big problem!” And nobody believes you. They don’t believe you because of counterintuitiveness. Donella Meadows says, “And we know from bitter experience that, because of counterintuitiveness, when we do discover the system’s leverage points, hardly anybody will believe us.”

The most powerful insight of my career was learning about counterintuitiveness. Changes that “make sense” in a system make sense to us because they match what we already think and know. When we discover a leverage point, a change in a system that we previously didn’t see, it isn’t “intuitive.” It doesn’t seem right to us because it works against what we know. The common, inevitable, reaction is doubt. We don’t believe it; we don’t listen; we need to experience a learning process.

The “right” answer to a systems challenge will rarely be the one fix that our linear minds offer. “Right” is in quotes because there is rarely a “right” answer, only the best possible answer under the circumstances. We have developed an intuition for familiar systemic patterns. To solve systems challenges, we usually need to change those patterns. We are blind to, and uncomfortable with, ideas that run counter to our “intuition.” When we are thinking in systems, the best answer will often push against what we “know.”

Jay Forrester, systems pioneer at MIT, describes how many organizations know exactly where they need to make a change. They recognize the leverage point, but…

> Then I’ve gone to the company and discovered that there’s already a lot of attention to that point. Everyone is trying very hard to push it IN THE WRONG DIRECTION!

In the oft-quoted _Mythical Man-Month_, Brooks expresses what is, perhaps, the most famous example of counterintuitiveness in tech:

> Adding manpower to a late software project makes it later.

Counterintuitiveness isn’t a bad thing—it is inescapable. We always have blind spots. Systems thinking is proactively looking for them. Actions we take in a software system are _always_ an experiment; we can never be certain we’ll get the desired result. We increase our chances when our reasoning about that action is sound, relevant, and cohesive. In [Chapter 7](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch07.html#collective_systemic_reasoning), we will practice systemic reasoning, the art and science of creating sound and cohesive recommendations.

As you embrace the practice of systems thinking, you are also embracing this fact: you will go the wrong way, make a bad situation worse, fix a mistake with another mistake. In systems, we aren’t trying to do the right thing every time because that is impossible. Systems are always in flux.

# A System in Flux

We design software to operate in a particular circumstance, at a particular time, serving a particular purpose. But things change. Software interrelates with an ever-changing world. Which means we are ever-changing our conceptual models. Conceptual integrity isn’t a static thing; it’s a quality we are constantly creating, in a system and in ourselves.

We learn through experience; both the good stuff and the frustratingly recurring stuff teach us. As we learn, we evolve our concepts. The integrity of those concepts, at any given moment, depends on flux, what is flowing in and what is flowing out.

When we move from a monolith to microservices, for example, we are learning new implementation skills, which are significant. We are also learning new ways to think about the world. The monolith fit the world it was born into. The microservices are needed for a new world. What’s changed?

A systems perspective depends on understanding what changed. How does the system no longer serve its purpose? (And if the system _is_ serving its purpose just fine, why are you redesigning it?) In the new world, the new paradigm, where microservices improve the system, we don’t simply need new technology tools—we need new patterns and relationships, in the tech and in the people building it. What needs to change?

To think about this change, let’s use Donella Meadows’ simplest drawing of a system ([Figure 2-1](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch02.html#fig_2_donella_meadows_basic_system_model)). The middle box is the state of a system: the information, physical parts, and activities it contains at any given time. In our example, it’s the monolith, the one large application. Inflows are what we put into the software, and outflows are what we get as a result.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_0202.png)

###### Figure 2-1. Donella Meadows’ basic system model

When we want the software to output something different, we are identifying a discrepancy. What we want the software to do is different from what the software currently does. This can be a bug or a way that the software isn’t serving the system’s purpose. Users increasingly access information on their phones, for example, and the information was designed for a bigger display window. This can be a change that improves the system’s ability to serve its purpose. “Users can’t see their past purchases, which inhibits our ability to form long-term relationships with our customers.” We set a goal, “Enable users to click a button and see their past purchases.” We change the inflows by pushing new code or information or both, until users can see their recent purchases.

In a linear thinking process, this is the end. Delivered! But users clicking that button will generate more insight into the discrepancies. Does the data display quickly or does it lag? Does the new code break something elsewhere in the codebase? Was the data we stored about a transaction the same data users want to see? Users ask, “Can I quickly reorder something I’ve ordered in the past?” “Can I see the status of an order I haven’t received?” This system model isn’t linear—deliver and done. It’s a constant state of learning: observing, analyzing, discerning, designing, and redesigning.

Now imagine that this basic model is a microservice, a single service in relation to other services. The core model is the same. We identify a desired change (a discrepancy between what we want to happen and what is happening). This application has a state, like the monolith.

But where is the state of the system as a whole? When you have software parts in relationship, like in [Figure 2-2](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch02.html#fig_3_a_system_of_software), you can see that there are asynchronous states in a system.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_0203.png)

###### Figure 2-2. A system of software

This demonstrates what I mean by “relational complexity.” The state of the system becomes something ubiquitous, inside the software and in the relationships among the software and in the relationship between the system and the world around it. This figure shows one inflow and one outflow for the system as a whole, but often, there are more than one.

If there is a discrepancy, where in the system do you make a change?

The answer to that question is “it depends.” Fortunately, systems thinking helps you discern what “it depends on” as you learn how relationships produce effect.

Changes in a system, regardless of the level of relational complexity, happen over time. Over time, the discrepancy in any given area becomes zero. _Time is always a factor_ in systems because the state changes over time.

The states change over time nonlinearly. The basic model hides the complexity in most real-life systems because we rarely have one goal at a time. Also, you can’t control everything happening outside the state box while the discrepancy diminishes. While you are changing a system, the world is changing around it.

# A System of Ideas

> The true system, the real system, is our present construction of systematic thought itself, rationality itself, and if a factory is torn down but the rationality which produced it is left standing, then that rationality will simply produce another factory.
> 
> Robert Pirsig

The state box I used earlier to describe software can hold intangible things too, like trust. If you lie about the status of work in progress, the state of trust will lower (over time). If you are transparent, trust increases.

Imagine that the state box is full of ideas. Ideas about how the system works, how it should work, what needs to be changed, and how people should work together. Ideas about top priorities, methods for delivering software, OKRs, and which capabilities are business-critical. The box includes everyone’s opinion about which tools are best to use, how to ensure quality, and what defines a good culture.

In [Figure 2-3](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch02.html#fig_4_donella_meadows_basic_system_model_plus_thinking), I’ve added thinking.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098151324/files/assets/lsth_0204.png)

###### Figure 2-3. Donella Meadows’ basic system model plus thinking

Thoughts flow in and actions flow out. In the state box, there are a myriad of attitudes, points of view, opinions, recommendations, knowledge about the technology system (both accurate and inaccurate), and its impact on people…all tumbling around like clothes in a dryer, becoming entangled.

The same thing is happening in our own minds.

When we think in systems, we create cohesion, harmony, a sense of order from that pile of idea laundry. We integrate and synthesize disparate ideas, knowledge, points of view, and activities until we can see and express what matters most to do, under the circumstances. We look at the patterns and relationships, improve our understanding of how things work, until we can see a path toward matterful change.

# Time Is Always a Factor

> People assume that time is a strict progression of cause to effect, but **actually** from a non-linear, non-subjective viewpoint—it’s more like a big ball of wibbly wobbly…time-y wimey…stuff.
> 
> Doctor Who

As I said earlier, _time is always a factor_ in systems. We like to imagine time as linear and synchronous. We create Gantt charts and set milestones and take “next steps.” We manage time with calendars that structure our social attention. We use time-tracking apps to record the time it takes to complete tasks. We are very, very careful about our software’s uptime, query and response times, and on-time delivery.

These are all helpful tools, when used with awareness. I write (this book, for example) in regular, structured sessions. I put on my noise-canceling headphones, start my background-music timer, and write for 44 minutes while ignoring (usually, mostly, kinda) mental urges to check email-Slack-Mastodon or do the Wordle.

Yet a lot of “writing” happens in the time between sessions. Insights, thoughts, and mental images arise in my mind at all hours. When I’m deep in a project, other habits stop working for me because my attention has shifted. (I take longer to respond to email, for example.) I get pulled back in randomly.

For example, while writing this book, I get feedback from Shira, my editor, who reveals a blind spot that needs my attention. I drop what I am doing to explore it. No matter how good you are at systems thinking, you will have blind spots. In my case, while writing, I struggle with the curse of expertise. I describe a concept using a concept that you, the reader, might not know yet. But I don’t notice because my mind thinks that everyone knows whatever I know. I need people like Shira to have my back. You also need Shiras, because you have and will continue to have blind spots.

Writing is chaotic, and if I hold the chaos at bay, I can’t create something novel. If I surrender to the chaos and get swept up in it, I can’t create anything at all.

Wibbly wobbly, timey wimey. Linear, synchronous progress exists inside a nonlinear, asynchronous reality that is co-creating something new. That’s okay, we don’t need to fix that. We just need to recognize that nonlinear time impacts the way we code, the people processes and the patterns we design.

When I’m writing code, I enjoy writing it, running it, and seeing (immediately) what happens. Direct, logical feedback. If it’s broke, I fix it.

Nowadays, this feedback loop is increasingly time-delayed. Changing one event in one part of the system might trigger unwanted behavior elsewhere. My IDE can’t catch that. Layers of caching, eventual consistency, or explicitly designed time delays make “seeing” what was happening in real time across a system challenging. I need insight, through observability, into what happens over time. And I think about what I’m learning about time as I’m coding.

Similarly, the timing of people processes is changing. When I worked as one of three engineering teams pushing code to a monolith, we did code reviews as if we were one team. We followed the best practices for that software (defining the way we wrote tests, for example), and the syntax rules were shared by everyone.

Now, different parts of the organization build different software that interrelates (or not). Or we are building the same software but because of siloed budgets and very little time or tolerance for designing systemic relationships, we stuff conflicting outflows in simultaneously.

We are also designing relationship patterns, and those are all about timing. Fifteen microservices and/or four teams, working on different software parts, will be inherently asynchronous. Changes are happening asynchronously—the changes we intended and the changes we did not intend.

Questions we consider:

- How do the circumstances change over time?
    
- What happens in response to a high-impact event?
    
- What is the root cause of that event? (Why do things happen how they happen and when they happen?)
    

###### Note

Caution: I have heard teams describe asynchronicity as the ability to be “independent.” For example, decoupling the frontend software from the backend software so that the two teams can work independently. There is still a _relationship_ between those teams, and with the rest of the organization. Those relationships are part of a system.

Even when a team tries to “manage” complexity by building a fortress surrounded by a moat filled with crocodiles to protect the boundaries of their software, that team is still part of a sociotechnical system. A system that includes crocodiles. The software is still part of a system. We still need to understand how the people, and the software, are interdependent. (And understand why we are building fortresses.)

We often use the word “manage” in relation to time. We manage projects and people, complexity and infrastructure. But when projects, people, and patterns act asynchronously, and in unexpected ways, we find ourselves in a muddle.

Systems thinking shifts our attention toward “orchestrating,” a more subtle and artful approach. People and activities are viewed as interdependent and interrelated, like a symphony. The whole has cohesion and understandability even though the parts are played in their own time.

Independent and interdependent; manage and orchestration…these are examples of the many conceptual shifts we make when we shift from thinking about software to thinking about systems of software. In the next chapter, we’ll explore more conceptual shifts and introduce the Iceberg Model, a core tool for thinking about systems.

# Support for Your Practice: Riding on the Front of the Train

Practicing systems thinking and nonlinear approaches has welcome benefits. You can tackle more-complex problems, recommend changes that improve conceptual integrity, grow and learn constantly, build things that matter to the people who use them. With practice, you will be correct more often (in the long run), and, more importantly, you will enjoy thinking with others.

There is one challenge that is, perhaps, not as welcome. Because of counterintuitiveness, you will sometimes be alone, thinking differently, with no one (yet) validating your perspective. In the face of invalidation, sometimes you will change your mind because your thinking is unsound. Sometimes you’ll stand your ground because your thinking is sound. Always, it can be challenging to discern the difference.

I have struggled with this challenge, both personally and professionally.

“Nobody thinks like you, Diana.” I am standing in the kitchen, leaning on a table edge, across from my partner (at the time) who is sitting on the counter. He is ranting about a work situation, full of pent-up frustration. I’ve just offered a strategic recommendation for changing his situation. He didn’t welcome it.

Fuss, Don’t Fix. That’s what I call this habit. Blaming our frustration on external circumstances but not changing those circumstances. That’s what he’s doing. He is certainly not alone, I Fuss but Don’t Fix. You probably do too, at least sometimes. There is, as we will see in the next chapter, a lot of blaming circumstances in systems.

“Nobody thinks like you.” He was rejecting my thinking because it didn’t fit “the norm” in his situation. Ironic, given that the norm in his situation was frustrating him. I was thinking differently.

Perhaps you’ve had this experience? If you haven’t, as you improve your ability to think in systems, you will. Having a perspective that is “outside the norm,” feeling alone with a reality denied, is, I’m sorry to tell you so early in our journey together, part of systems design.

My architect colleague, Mark, calls it “the front of the train.” He first used that phrase during a meeting in London. We were sitting on uncomfortable chairs in a borrowed office, facing each other. I was jet lagged and Fussing Not Fixing a situation I’d flown over to discuss. “They just don’t get it!” I said. “This is important!” I said.

“Diana,” he replied gently but firmly, “you are on the front of the train. You look out and see a forest. You say, ‘Look at the trees!’ People riding in the other cars say, ‘What are you talking about? That’s a lake!’ You don’t get to be mad at them. They aren’t looking at the trees yet. Riding in the front of the train is your job.”

Front of the train thinking is systems thinking and design. Which brings us straight to the hard cheese: there is no technology you can adopt, group you can join, role you can be promoted into, tool you can use, or solution you can recommend that will work universally. This book can’t tell you how to “fix” situations using management techniques, using Kubernetes clusters, or hiring more “juniors.” People will want you to give them templated solutions, easy answers to complex problems. You can’t do that, most of the time.

There is no magic bullet. Even if there was a magic bullet, you’d be standing there holding it, and nobody would believe you. People don’t see magic; they see what they expect to see. You need to show people, over time, how the magic works.

You can’t know if you are right, but being right is always temporary anyway. Nonlinear approaches value seeing things as they are (and can be). Working with others to get input that strengthens your thinking. Gracefully letting go of wrong views. And being patient, kind, and curious about other people’s perspectives while you learn (and learn and learn) to be “right” together.

What works in any circumstance…depends. Depends on the context, depends on the people, depends on how integrous (or not) the communication flow is. Depends on what you say and why people don’t want to hear it. Depends on a myriad of factors.

Systems thinking is figuring out what “it depends” on. Then communicating new ways of seeing until people can see it. To become good at communicating, you’ll need to practice thinking and communicating your thinking.

# Your Practice: Writing as Thinking

A powerful tool for practicing systems thinking is writing. We will dive deeper into using writing as a thinking practice in [Chapter 4](https://learning.oreilly.com/library/view/learning-systems-thinking/9781098151324/ch04.html#self_awareness_as_a_foundational_skill). Let’s begin with an exercise. Grab paper and pen, open a note-taking app, talk to Siri, however you’d like to work. Set a timer for 20 minutes and explore these questions:

Remember a time when you (and your teammates) had a goal, a change that you wanted to make to a system.

- What was the goal? What was the process of resolving the discrepancy, moving from the current state to the goal state?
    
- How were the ideas being shared? What was the structure or template used? Where were they stored?
    
- Whose ideas were they? In retrospect, were there ideas that might have been helpful but weren’t available or heard?
    
- What was the output? How well did the output match the goal? How do you know?
    
- What was the impact of time? Was there a longer-term impact? Did the change(s) trigger changes elsewhere?
    
- Did counterintuitiveness play a role? Did you discover later that the problem persisted in a different form?
    
- Do you see any opportunities for improving the process you just described?
    

# Counterintuitive MAGO

When MAGO experienced the loss of their core software, the mandate from the CEO was: Replace the Software, Quick! This makes sense; the organization faced a business-critical crisis, and this pointed directly to the problem.

Except that the problem didn’t just happen. It had been creeping up on them for 20 years. And during those decades, they inadvertently made the problem worse by adding complexity to the system that solved the short-term problems (serving digital content in multiple contexts) but made the system’s relational patterns difficult, if not impossible, to change.

The concepts that informed the state of the system arose from producing a publication (a magazine, in print and digital). As the system grew, these concepts became blockers. What does it mean to distribute content to many platforms, serving many contexts? How is that different from producing a publication? If MAGO replaces the software with a similar software, they will be making the problem worse, because now they will have spent millions of dollars on something that won’t help them maintain competitive advantage.

What should they do instead? Stay tuned!