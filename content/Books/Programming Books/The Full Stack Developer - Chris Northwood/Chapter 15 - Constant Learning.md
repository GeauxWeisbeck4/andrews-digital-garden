---
id: 01JB58FGP72PH2RK6TD4R84QTM
title: Chapter 15 - Constant Learning
modified: 2024-10-26T16:41:09-04:00
tags:
  - full-stack
  - books
---
# 15. Constant Learning

Chris Northwood[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_15_Chapter.xhtml#Aff2) 

(1)

Manchester, UK

One area where the modern full stack developer working in a digital organization will differ from a traditional enterprise is that the products they are working on will constantly evolve in response to the real world at a rapid pace, rather than simply in response to requirements being pushed upon them by the organization. It is no longer enough to build something to meet some acceptance criteria and then push it out into the world; you and your team will have to check that any change you've made is actually having the impact you want it to have.

Getting into a position to do this can be hard unless your team is truly working as a core part of your organization. Any new feature you build or change you make also requires you to understand _why_ that change is being made—is it to increase sign-ups or sales, or to satisfy some other business need? Once you understand this underlying motivation, then shipping a feature is no longer enough, if that feature cannot satisfy this underlying cause.

## Collecting Analytics

Many organizations are now relying on data to make informed decisions about strategies, but before the data can be interrogated, it must be collected and held centrally. The buzz term "big data" is sometimes used to talk about this, but this usually means collecting large amounts of unstructured data and analyzing it for insights. For many organizations, getting a handle on individual bits of data can be very powerful. These could be simple numbers, like value of sales, and can be recorded by standard business processes and then aggregated and displayed to stakeholders.

Another type of analytics is derived from data on how a user interacts with your web site: the path they took to a particular webpage, how long they spent on it when they got there, whether there were any errors made while completing a form, etc. The Web is very powerful in allowing you to collect this data, but there have been many abuses of it (for example, Uber's "God View,") that have led many to be wary of analytics and tracking scripts. In many countries, privacy law no longer allows you the ability to collect this data unless the user opts in, and when using analytics, it can be tempting to simply collect everything and then decide how to use it later. This approach may seem technically simpler and give you much more flexibility, but will often fall foul of these laws, as the data then becomes "personally identifying" and may contain very sensitive information. This means you must protect and manage this data in a much more careful way. To avoid this, you must consider what data you want to collect in advance.

Ultimately, the way you want to collect most data is by aggregating it together into counters, such as "how many people clicked this button?" Having a single counter may be useful, but often you will want to ask questions like "how many people clicked this button yesterday?", "how many people clicked this button on a mobile phone?", or "how many people clicked this button and then ended up buying something?". Implementing a simple counter isn't enough, so you often have to think about “segmentation.” In this case, a high-level metric is a bucket that contains a number of other metrics that allows you to slice it into other meaningful pieces. For example, instead of incrementing a counter, you add a record to a bucket that contains a number of key/value pairs. Let's say you have a bucket called "product_button_clicked." Then, whenever someone clicks that button, you might add an entry like:

product=12345;

browser=Safari;

device=Windows_PC;

time=2017-12-19T09:00:17Z

location=ManchesterUK

You must be careful not to collect information that can ultimately lead back to the user, as this becomes personally identifiable, but these labels allow you to segment the total count of items in the bucket in different ways to ask interesting questions. Most analytics toolkits will collect some of these things for you (especially time, location, and browser info), and others require you to add the segments yourself. However, this doesn't help answer the final question, which is often something like, “how many people clicked that button and then went on to buy something?”.

To achieve this, most analytics software will also add a "session ID" to the bucket, which then allows you to see which actions happened in the same session. This can be dangerous, as correlating actions across a single session can allow you to identify an individual user, hence losing the anonymization achieved by placing actions into buckets. Many people accept this as a risk and store all activities, but others only store the session data for a period of time until after the last bit of activity in that session, and then look at the session data to generate answers to those questions before discarding it. A final approach is to set a flag such as "clicked_button=true" in a session and then store that as an additional segment later on in appropriate actions, which avoids capturing any session data at all.

The final thing to consider is exactly what interaction data to collect. Simple things like "opened a page" and "clicked a button" (or had some other sort of interaction) are useful and happen in response to direct user actions. Others may happen more implicitly (such as recording time spent on a page, whether the user reached the end of an article, or if the page was left open with no interactions for a long period of time). To determine what's useful, it's important to talk to your stakeholders and any user experience practitioners on your team to find out what they need.

The same mechanisms used for collecting analytics can be used beyond this use case—for example, to develop personalization and recommendation systems—but those use cases are not covered here, and have their own set of ethical and legal implications. You should always make it clear to your users what information you're collecting and why—otherwise, they may assume the worst.

## Experiments

In addition to reflecting on the performance of your site using analytics, it's also possible to experiment directly with your users by giving them different versions of the same page and seeing how they respond. Experimenting on people is can be morally fraught, but it happens constantly, so how does one do so ethically? The core question is to ask whether or not either variant could result in harm to a user. For example, testing the size and placement of a button probably will not, but applying a "dark pattern" to entice a user into spending more, or A/B testing pricing structures where some users may end up paying more than others based on which segment they are placed into, can cause harm and should be avoided. Other scenarios are less clear cut, and informed consent can be useful, perhaps by allowing a user to opt in to a "beta trial" and making it clear that they will be participating in experiments, but then leaving most users out of it.

Experiments can be useful when you’re trialing new site features and you are unsure how to achieve a specific goal. An A/B test is a fairly simple experiment: you start by giving 90% of your users the existing version of a page (the "control"), and then 5% a variant “A”, which might have a new feature, and 5% variant “B”, which could have a different design or workflow for that feature. You run the experiment for a period of time and add appropriate analytics (remembering to segment whether or not the user is in the control, A, or B group) and then compare the results to see which of variant A or B is better (or if the new feature is actually worthless or helps less than it not being there at all, by comparing it to the control).

With A/B testing, you can only test one change at a time, which can be slow. Multivariate testing has grown out of it which allows you to run several A/B tests in parallel, but the analysis of the results is more complex in order to separate any effects one test may have on another.

## Analyzing Results

It is not enough to simply frame your updates in terms of the underlying change you hope to make, but you must also have a way of measuring that change in a meaningful way. Although often misused, key performance indicators (KPIs) can be a useful way to measure these. KPIs fall down when they measure things that are easy to measure but are not actually what the underlying goal is. For example, a KPI might be number of visitors to a page, as this is easy to measure, but if many of those visitors simply leave a page without completing a meaningful action, then by making page views a KPI, this can make a team focus on getting people to a page, but forget about what they do once they are actually there, which is the thing that actually matters.

As always, the right KPIs are context dependent, and can require some deep thinking to make sure you get it right. In some cases, KPIs can be hard to directly measure quantitatively, but some qualitative measures can be applied instead.

Quantitative measures are often desirable though, as they are easy to collect and seem easy to understand. A headline number can be easy to understand, but a simple number can have many depths that can lead to naive, but wrong, interpretations. The process of deriving a figure can be complex, and it's important to understand those trade-offs.

### Bayes Theorem

Bayes theorem is a fundamental theory in probability, and you may have come across it before. Expressed as an equation, it reads as ![$$ P\left(A|B\right)=\frac{P\left(B|A\right)\ P(A)}{P(B)} $$](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_15_Chapter/471976_1_En_15_Chapter_TeX_IEq1.png). It expresses the probability of event _A_ occurring if some other event _B_ is true (the "conditional probability"), based on the probability of the inverse and the probabilities of each of those events occurring independently of each other. It highlights how some headline figures can give misleading results.

If a test for cancer is 99% accurate, and that this cancer occurs in 0.1% of the population, then through Bayes theorem, if this test detects you have cancer, it means you only have a 9% chance of actually having the cancer. This can seem counter intuitive due to the 99% accuracy, but because the cancer is relatively rare, this rareness cancels out the accuracy to give you the "conditional probability" of 9%.

This book isn't the place to go into statistics theory, and if you're lucky, you'll have access to people who have a good understanding of analytics and data (sometimes called data scientists), or good tools, to help you understand them. An important concept to understand is called the confidence interval, which expresses the percentage confidence that a value is within a specified range. The important thing to remember is not to trust a headline figure. For example, in an A/B test, if result A is 52% conversion and result B is 48% conversion, then actually the two results could be the same, or result B could actually be better performing, depending on the margin of error, which causes the confidence intervals to overlap. The more data you have, the smaller your margin of error becomes, but if, say, the margin of error was ±3%, then the result for A is (with 99% certainty) between 45-51%, and for B, it's between 49-55%. So there's a chance A could be 51% and B could be 49%.

When looking at analytics, although this numeric data can help give you an idea of high-level trends, if will not often tell you why those trends are occurring. For that, you need to use qualitative data. This qualitative data comes through understanding your customers, rather than just watching aggregated metrics. User testing of designs is one way of achieving that (you can user test a built design), as well as having panels and focus groups that allow you to ask questions directly. Another alternative is to capture analytics for every possible metric and store them in a way where you can look at all of the analytics of an individual session and dive into each session individually, but done in bulk this violates the privacy of an individual user, and is a level of observation that a user should explicitly consent to (this is why user testing is a good place to do it).

## Hypothesis-Driven Development

Hypothesis-driven development has been proposed as an alternative way of expressing work to be done in user stories, by phrasing it in a way familiar to a high-school science student. Unlike user stories, which express a change from the perspective of a user, hypotheses go one level deeper and express a change in terms of a question. Barry O'Reilly proposed the following form:

- We believe <_this change_>
    
- Will result in <_some outcome_>
    
- And we will have confidence to proceed when <_a measurable impact_>
    

For example, for a marketing landing page:

- We believe that adding a mailing list sign-up to a product landing page
    
- Will result in an increased number of mailing list signups
    
- And we will have confidence to proceed when the number of mailing list signups in a week is 15% higher than the typical signup amount after seven days
    

This means that your feature does not finish once it ships, but instead you must revisit it seven days afterwards to ensure that it has had the desired impact, and then either roll back the change, revisit the design, or hypothesize a new change to test to have the same advantage.

One challenge with hypothesis-driven design is identifying the meaningful change that you want to impose, rather than simply naming something that might be easy to measure. For example, placing some content behind a sign-in might increase your sign-in metric, but is that actually the ultimate measure you care about, or is it something more core to the fundamentals of your organization?

## Summary

The work of a full stack development team is not done once a feature is delivered. Rather, you should be monitoring the feature to make sure it does what you expected, as well as constantly learning from how your site interacts with users to identify improvements that can be applied to continuously improve your product.

Collecting analytics, and taking care to be conscientious about how you do so, can provide invaluable insight into how your product is really used by your end users, and how that use may change over time. When you are less sure about a change, you can also run experiments on the running site to see how different variants behave in practice. Care must be taken when analyzing this data to not fall into statistical traps that falsely draw conclusions.

Hypothesis-driven development is a way of formalizing this approach to review, by stating changes in terms of hypotheses to be tested, highlighting that the work isn’t done until this hypothesis is tested.