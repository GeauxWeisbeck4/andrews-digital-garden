---
id: 01JH1P24XBZYE8BWVK5S992T3D
modified: 2025-01-07T19:47:13-05:00
---
# Chapter 30. Understanding the Business Domain

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 30th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

One thing that will set you apart is deeply understanding the business domain you are developing an app for. Any dev can write software to meet requirements, but when you understand how a specific domain works, you’ll be able to make better-informed decisions. The domains you may encounter could be industry specific, such as advertising and media, energy, government, financial services, health care, hospitality, manufacturing, logistics, retail, telecommunications, and travel. Domains could also include multiple industries or no industry at all.

A business domain expert is someone who deeply understands the industry they work in and many of the intricacies and problems in it. You’ll find that often the domain experts aren’t the people who create the product requirements or the devs who build the product. They’re the people more on the business side who have seen a gap in the industry and are building a solution with help from others, like you. A good example of a domain expert is someone who has worked in health care for years and who has seen the problems that staff and patients deal with.

In this chapter, I’ll go over:

- Domain-specific knowledge
    
- App needs for different business domains
    
- Architectural and system design decisions
    
- How to learn from other teams
    
- Documentation
    

You don’t have to be that person who’s worked in an industry for years to become a domain expert. But you do have to have a genuine interest in learning as much as you can about a specific domain. That means you’ll have to step away from the developer world and talk to domain experts and people who currently work in that domain. Doing some research on what’s currently happening in an industry will help you gain some of that knowledge. It’s equally important to understand the history of an industry so that you know what has already been tried and the results. This is one of the fun things that many developers at all levels tend to overlook in favor of furthering their tech skills.

###### NOTE

Make sure you don’t assume you have all the answers, even with your new domain knowledge. Domain experts have more experience and understand the processes and systems better than you. They often have undocumented systems that accomplish specific tasks. We want to understand those steps to help automate them with software, but we can’t do that without the experts’ help validating that things are doing what they’re supposed to.

# Domain-Specific Knowledge

At this point, there are a lot of different paths you can take in your career and your skill development. Regardless of whether you want to stay on the individual-contributor path or move over to the management track, knowing how to learn about a business domain will be a huge asset.

Every product you work on is providing a service to the organization’s customers. The organization is building this product because it’s something that has a competitive advantage in some way. This advantage may not even be a technical one because a nontechnical advantage is also important. Maybe a logistics company has found a new way to organize inventory, and the software is just an aid to that. When you get into the business domain, unless the domain is technology, your technical knowledge isn’t what you should lean on because it’s secondary to the organization’s focus. Remember, you are always building software to solve some kind of problem. The software is how you deliver a solution to the users, but it may not be the solution itself.

###### NOTE

A quote from _Empowered_ by Marty Cagan (Wiley) resonates with this topic:

> In strong product companies, technology is not an expense, it is the business. Technology enables and powers the products and services we provide to our customers. Technology allows us to solve problems for our customers in ways that are just now possible. Whether the product or service is an insurance policy, a bank account, or an overnight parcel delivery, that product now has enabling technology at its core.

There are a few strategies you can use to familiarize yourself with different domains. I’ll go over three of them.

## Learn from People Working Directly in the Domain

The first strategy is learning from domain experts where they are. It’s always great to start with people who already work in the domain every day because they can tell you exactly what they do and how. These are the people who know what is really happening behind the scenes to keep everything moving. For example, if you’re working on software for a manufacturing organization, go to the shop and talk to the machinists and mechanics who actually build the physical products.

You can also join online communities or go to meetups and conferences for that domain. It doesn’t hurt to join some organizations that are focused on people in that specific domain. Professional trade associations and standards bodies will help you connect with the people doing this work every day. That could include local chapters of an engineering community or a casual get-together for people who work on CNC machines.

You’re trying to figure out where these experts are and go to them. You can start internally at your organization by talking to some people in sales and marketing to see who they are targeting. The main thing is that you make an effort to show up where they are having discussions and listen to them. It’s OK if you don’t understand what they are talking about in the beginning or the actual meaning behind the changes you read about. Ask questions when you have them, and you’ll find people who are happy to answer you.

It’s good to learn about some of the history of the domain because that will give you a lot of context for the current state. You might start reading articles about the news in an industry and learn about how it’s changed over time and who the leaders are. This might even help you understand the purpose of your organization’s work. It also helps establish you as a domain expert because you will start to understand and speak the jargon for that domain. Then you can bring in your technical skills and show how they apply to that domain in a meaningful way. It will also give you more empathy when you’re developing features for users because you really get what the use case is.

## Take Courses

The second thing you can do to build domain knowledge is take courses. There are free online courses for just about anything now. With a few searches, you can find a course on logistics or hospitality. If you want a more structured approach, courses will typically give you that. It also gives you a chance to deeply learn about something not related to your technical skills. Some devs see this as a conflict with keeping technical skills up to date because it takes time away from the code and other technical decision making. I personally think it helps broaden your perspective with using the tools because you can focus more on what matters: the product.

Consider taking courses at a local community college or another type of institution and see if your organization will pay for it. These programs give you hands-on experience in what a domain is really about. You can get something from talking to people and reading on your own, but some domains benefit from a more structured introduction. I suggest selecting one domain and trying to find free courses to start with to decide if that’s the domain you really want to do a deep dive into.

## Work in the Domain

The third approach to gaining domain knowledge is to go work in that domain for a while! You might find a way to get the experience by volunteering with different organizations. For a while, I was interested in construction, and I was able to get involved with Habitat for Humanity, a nonprofit that builds or updates houses for people in the community. It was a great way to help people and learn a lot at the same time. You can also look for part-time opportunities so that you can stay in tech while you explore another domain. The best way I’ve found to get this exposure is to ask local business owners if they need help and explain to them what you’re trying to do.

Maybe you’ve been thinking about what you would do if you didn’t write software, and you’ve wanted to explore other options. If you are able to do so, working in a domain for a while will give you all kinds of new skills. This approach isn’t for everyone, and it requires a much larger commitment than the others discussed so far, but it can be fun and give you more perspective on the technology industry as a whole.

This will open a whole new world to you, which might even inspire you to try the entrepreneur route. One way to come up with business ideas is to figure out what an industry is lacking, and the best way to do that is firsthand experience. Even if you don’t take on a job or volunteer opportunity to learn more about a domain, listening to domain experts after you have some general understanding of how the domain operates will provide tidbits on the missing pieces or pain points. You can take those and expand further to figure out how your tech skills and domain knowledge can overlap, and that can give you a target to focus on in the domain.

# Architectural and System Design Decisions

After you’ve gained more understanding about the business domain, you can use it to drive some architecture designs for the software. You can use [clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html), [CRUD (create, read, update, and delete) architecture](https://www.oreilly.com/library/view/migrating-to-microservice/9781492048824/ch04.html), or [layered architecture](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html), and these will work most of the time. But if you’re working in a more complex domain, you might choose an architecture that better fits it. There is some overlap between the domain-based architectures and the others, so be mindful of the complexity that you introduce into the design pattern for the app.

## Domain-Driven Design

_Domain-driven design_ (DDD) is an architecture that involves designing the product to match the business domain. It takes more work to implement up front because you and the team have to work together with domain experts to come up with a shared language that describes the functionality consistently. This involves creating a domain model that reflects the rules, processes, and entities in the business domain. By having all of this in the beginning, you can set the code up to precisely meet the needs of the business.

There are several parts to the architecture that you have to define: bounded contexts, entities, value objects, aggregates, and events. Using this approach typically involves microservices that use [_bounded contexts_](https://www.oreilly.com/library/view/learning-domain-driven-design/9781098100124/ch04.html), parts of the code that are separated by business functionality.

_Entities_ in this architecture are objects with unique identities that aren’t defined by their attributes. These are usually things that may have attributes that change over time and need to be tracked. An example of an entity is an order because it has a consistent identity, such as an order ID, that doesn’t change based on the total price, the customer, or the products. But it will likely have a status attribute that needs to be tracked and updated over time. These are also called _reference objects_ because while the attributes they have may change, what they represent in the business remains the same. [Figure 30-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch30.html#an_order_entity) shows what an order entity might look like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_3001.png)

###### Figure 30-1. An order entity

You could have any values or types in the attributes for this order entity, and it will still represent an order to the business. Entities always have a specific ID, are mutable, and typically have a lifespan. So while an order will always be part of the organization’s data, it’s fine if the attributes change as long as the ID remains the same.

_Value objects,_ on the other hand, are defined by their attributes, are typically immutable, and don’t have unique IDs. This includes things like addresses, prices, and dates. In our example, a value object might be the customer. [Figure 30-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch30.html#an_address_value_object_related_to_the) shows what the address value object looks like.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_3002.png)

###### Figure 30-2. An address value object related to the order entity

You can think of a value object as a type for an attribute in the entity. If you change one of the attributes in the type, it makes a completely new object. For example, if someone updates their shipping address, we don’t want to update the old one because it could be their billing address. So that would lead to a new instance of the address being passed to the order. You can usually think about the relationship between entities and value objects as the difference between a table and a row in a database. An entity could represent a table while a value object may represent some columns in a row of that table.

_Aggregates_ show relationships between entities and values to represent how they are grouped together. For example, if you take an order and add more info to it, such as a whole customer entity, that could then be seen as an aggregate because it’s grouping information.

_Events_ are the triggers in the system that make different parts of the domain react based on changes that happen across the bounded contexts. For example, if a customer has enabled automatic ordering of a product by a certain time, when the date passes, it can trigger events to charge the customer and ship their products.

When the business domain is complex and has a lot of overlapping dependencies between parts of the system, it may be worth the effort to set all this up. This architecture does have a steep learning curve because of all the domain knowledge the team needs and the intentionality you need to have when creating the entities, values, events, and bounded contexts. All of these pieces will grow in complexity as the organization grows to have more offerings.

You have to make sure events are handled correctly so that your system doesn’t end up in a broken state where data isn’t being updated as expected or the events aren’t being triggered at the right times. This takes special effort when you’re working with third-party systems because data can get out of sync between what you have in your database and what they show users in their systems. Keeping bounded contexts clean can also become difficult as you start to need data from one context to trigger functionality in another context.

One of the best resources on DDD is _Domain-Driven Design: Tackling Complexity in the Heart of Software by Eric Evans (Addison-Wesley)_. This was the book that established the DDD architecture in the early 2000s, so it has all the information about how DDD should work and be implemented by your team. You can also find some good information about DDD in [this blog post](https://www.geeksforgeeks.org/domain-driven-design-ddd/). DDD is such a huge topic that numerous books have been written about it with plenty of in-depth examples, including _Implementing Domain-Driven Design_ by Vaughn Vernon (Addison-Wesley), [_What Is Domain-Driven Design?_](https://www.oreilly.com/library/view/what-is-domain-driven/9781492057802/) by Vladik Khononov (O’Reilly), and [_Learning Domain-Driven Design_](https://www.oreilly.com/library/view/learning-domain-driven-design/9781098100124/) by Vladik Khononov (O’Reilly). I highly encourage you to take a look at these resources because they have entire chapters dedicated to some of the concepts I’ve only briefly mentioned here.

## C4 Design

The [_C4 model_](https://c4model.com/) is another type of architectural design that can be used when you understand the business domain of your product. You’ll still need to work with domain experts to create a shared language and understand the relationships among the parts of the system. This breaks down the diagramming process into four parts: the system context, containers, components, and code. The system context is made up of multiple containers. Each container has multiple components, and each component has code in it. These take the high level of the software business domain and zoom into the individual parts at different levels.

The _system context_ is where you typically start because it defines the product you’re building. It’s the most abstract of all the layers in this architecture. This is where you’ll define how your product fits into the business domain and what problem you’re solving. The containers in this architecture are defined as frontend apps, backend apps, databases, serverless functions, and content storage like S3 buckets.

This terminology takes time to adjust to because we call a lot of things _containers_. The same goes for _components_ in this architecture. Containers are similar to what you might think of for a frontend architecture, but it applies to every part of the system. Your backend can be described in terms of components in this architecture when you group functionality together. For example, all the methods and endpoints that handle orders can be called a component in this context. _Code_ refers to the software you’ll write to make up the components.

You can organize the code into folders and files you’ll write in the components. All of these pieces are connected in a diagram with lines that represent their relationships between one another, which lets you visualize how the smallest pieces of your software fit in the business domain. This will enable you to create an architecture that works like a map you can zoom in and out of when you need clarification on something in the domain. Tools that explicitly let you build C4 models include [Miro](https://miro.com/diagramming/c4-model-for-software-architecture/), [Lucidchart](https://www.lucidchart.com/blog/c4-model), [IcePanel](https://icepanel.io/c4-model), and [Mermaid](https://mermaid.js.org/syntax/c4c.html).

A good resource on the C4 model is _The C4 Model for Visualising Software Architecture_ by Simon Brown (Leanpub). [This helpful visual](https://c4model.com/diagrams/system-landscape) breaks down the different layers of the architecture. The main thing to remember is that the C4 model was made to help document how parts of the system work in an easy-to-read way.

# Learning from Other Teams

To understand the business domain of your organization with more focus on what you’re building, reach out to the internal teams that engineering doesn’t commonly talk to, including sales, marketing, finance, and customers. This is where you have to take initiative because opportunities to talk to these other teams don’t come up organically.

## Get on Sales Calls

Engineers don’t usually interact much with the sales team or the sales process. When we do, it’s usually to bring technical validation to a call so that customers understand our product better. That’s the perfect reason to jump on a call with a member of the sales team. You can learn more about what customers need and what drives the revenue for the organization. Getting involved in this process is a great way to get insights to bring back to the Engineering and Product teams.

Something cool that happens on these calls is understanding how the software you build fills a need in another organization. This is especially true when your product is for a nontech industry. Being able to see the real-world application makes a difference in how you think about features and how you implement them. It will also help you understand how the product is demonstrated by people who aren’t involved in the development process. You’ll be talking about technical implementation, and that might bring up some ideas you want to take back to the dev team.

At this phase, you aren’t going to provide technical support or help the customer integrate anything. You could help with the sales demos and highlight functionality that the sales team doesn’t know about or show different ways the product can be used. Be careful about this and make sure you coordinate what you’ll do with the salesperson. This is a great time to see how customers view the product you’re building and get the perspective of stakeholders. It will also shed light on the importance of your role in the organization. You already know you’re valued for your technical skills, but this will give you a new angle for looking at the software you build.

## Join User Research Studies

Participate in or watch user research studies or user testing sessions. These are commonly led by user researchers, product owners, or UX designers, depending on your organization. This is one way the Product team comes up with the features that you work on. They reach out to existing customers to see if there’s anything that could be improved in the app and then get on a call to walk through that with customers. Sometimes the best features come from a user just talking about what they do in the app and what they wish they had.

This is also how the Design team gains an understanding of how users actually interact with things. They might present the user with multiple designs for the same screen and watch how they navigate through different tasks. Then they can take the best from all the designs and make one user-refined experience.

## Talk to Marketing

The sales team might be closing the deal, but marketing helps generate the leads. Marketing’s activities include creating social media campaigns, running booths at conferences, developing and researching customer personas to learn where they are and what they want, and raising awareness of what the organization does through networking and building relationships across the industry. Marketing creates the funnel of customers that leads to sales. So they have to understand the product well enough to talk about it at a high level.

Reach out to someone in marketing to see if you can occasionally sit in on their discussions just to better understand what they talk about in regard to the product. You might learn that parts of the app are unclear to them, so they describe it in an unexpected way. You can also learn about the customer personas they’ve made and figure out more about the target audience for the software you’re building. This can help you think about accessibility, the location of tools, and even naming conventions.

You can review product screenshots, content copy, and videos to make sure they make sense with the capabilities of the app. This is another way you can get stakeholder feedback from the people who are trying to get others interested in the software you build. In all the meetings you go to, take notes and review them later. You could find out that something that was hard to implement is also hard for nontechnical users to understand.

## Listen in on Customer Support Calls

This is something that is crucial to product development. Understanding the issues customers have while using the product will help you figure out where things can be improved. You could also help the Support team learn more about the features of the app. Or you can create features that help the Support team assist customers better. Sometimes people know they’re having difficulties with something, but they can’t explain what they need. When you come in with the knowledge of how the app works under the hood, you can see where improvements can be made more quickly than they can explain the problem.

This might be a small thing, like better documentation or naming conventions. Or it could be something larger, such as a clearer layout for a page, refactored features, or new tools. Working directly with customers will show you edge cases you didn’t know could happen. This comes up a lot when users need to log in with another service like Google or Facebook. Understanding their struggles will give you more empathy for users as you build features, and it will lead you to ask more questions about requirements.

The things you’ll learn on customer calls will help you consider internal stakeholder needs more as well. This is something that can get overlooked in favor of getting more features out to customers. But if the product can’t be supported adequately, those new features will end up causing more customer frustration than they will solve. This will show you how you’re an internal stakeholder, too, because you might find yourself doing hacky things in the code to figure out where a user’s problem stems from.

## Learn About Legal

This is an area you don’t have to get super familiar with, but it helps to have a little knowledge about it. These are the people who make sure the organization doesn’t get into legal trouble because laws, regulations, or compliances aren’t met correctly. A few examples include PCI, HIPAA, GDPR, and international embargos. You might meet with them once or twice or even ask for a list of legal requirements for the software that you can review. This can help you do some technical auditing to make sure your systems can pass a legal audit.

Since you’re likely using third-party services or open source tools in some capacity, you need to make sure they meet compliance rules with the country your product originates from. This can get tricky to pin down since software is developed by thousands of people all over the world, but once you have a list of rules, it’s a little easier to know what you’re looking for. Talking to the legal team can also bring out more of the business domain you didn’t know about. It’s unlikely that they use the software, but they can tell you the legal implications for it.

# Documentation Considerations

While it’s not solely your responsibility to document everything in the business domain for the Engineering and Product teams, it will help if you bring some considerations to them. You want to help everyone keep the documentation organized in a way that’s easy to search through. When you’re writing docs that are centered on the business domain, it can be easy to bloat them with all the info you’ve learned from domain experts, other teams in the organization, and even the notes you’ve taken. Here are some things you can do to maintain the docs.

## Define Jargon

Since this is documentation for the business domain, there might be some acronyms or jargon that others are unfamiliar with. Don’t assume that people know what words mean because this documentation could quickly become part of the onboarding material for new team members, both in engineering and in other departments. This can help the dev team understand why parts of the architecture are divided like they are in the software and why you implement code a certain way.

You can distill some of the more common things you’ve learned about the domain here and work with Product to get more of the details. The Product team will likely have more domain knowledge, so collaborate with them by having them contribute to the docs. When they introduce new features that bring in another part of the domain, have them clearly define it in these shared docs.

## Only Keep Relevant Info

It can be easy to try to cram everything you’ve learned about the business domain into the documentation, but try to highlight the main points and put the details in an appendix, asides, or footnotes. Focus on things that pertain to the architecture and features so that new devs can understand how the code fits together and why. You don’t need to go into detail about the history of the domain or how the marketing department has different customer personas. Find a balance between giving enough context for the features and providing some background on how the domain operates. Keep in mind that if the info was useful to you, it will likely be useful to others.

## Share Your Knowledge

Once you’ve compiled all your knowledge, share it with the team! You can do things like lunch-and-learn meetings to informally teach others about the domain. These can be quick 15- to 30-minute meetings where you talk about behind-the-scenes processes, takeaways you’ve gotten from professional organizations, and how the domain connects to the software. You can also help others start presenting on domain topics that interest them.

A more laid-back approach to sharing knowledge is posting the latest news or interesting tidbits you come across in your research to your dev team chats, such as part of a conversation you had at a meetup or an article you ran across. You could even introduce this as something the team takes turns with. Maybe every other day or at least once a week, someone shares a new thing they learned in the chat.

## Show How the Product Affects the Organization

When you’re talking about the business domain, show how the software you and the team are building fits into the way the organization generates revenue. This is an area where you can use diagrams to clearly illustrate the gap that the product is filling in the domain. Having this included in your documentation can give the entire engineering department a better understanding of what they do and why. It can also show the rest of the organization how important the software work is.

Departments are frequently siloed, so it can be hard to describe what you do in the context of the whole organization. But when you have these kinds of docs to refer to, you can easily show the impact you’re making every day. This is going to overlap some with any organizational charts that are in place, but keep it tailored to engineering and perhaps your team specifically by including the details of your app in the diagram. Having diagrams and definitions like this might also make it easier to update your architecture and figure out which approaches are better for the long-term maintenance of the product along with the changes in the domain.

When you connect your team’s work to the overall goals of the organization, that helps the dev team really feel the impact they make. It’s easy for the team to become disconnected when they’re focused on getting features finished for a sprint. Showing them how their work contributes to the direct growth of the organization can be a huge morale boost and help them see how valuable their skills and inputs are.

# Conclusion

In this chapter, we went over some ways you can learn more about a business domain and why that’s important. It will help you tremendously to have some knowledge about the domain you work in because it can change the tools you use or the approaches you take. It helps you to become more involved in the organization holistically. It’s always good to remember that you aren’t writing code just to build a technically sound product. You’re writing code to create a product that meets the needs of its users and maybe entertains them a bit.

Domain knowledge coupled with relationship-building skills as you talk to other departments in the organization will make you invaluable to any organization you work with. It will also make you a more thoughtful engineer because you’ll be more likely to think about things like regulations and laws the app may fall under. You’ll understand the impact your changes will have on customers and how that drives business as a whole. With this kind of domain knowledge, the relationships you create, and the technical skills you have, you’ll be a more well-rounded senior dev in many aspects.