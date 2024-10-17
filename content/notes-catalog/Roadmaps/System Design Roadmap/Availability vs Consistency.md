---
id: 01JAC2C5PVKQY9SZ85VRZKP3AK
title: Availability vs Consistency
modified: 2024-10-16T21:47:44-04:00
tags:
  - system-design
  - programming
  - roadmaps
  - availability
  - consistency
---
# Availability vs Consistency

Availability refers to the ability of a system to provide its services to clients even in the presence of failures. This is often measured in terms of the percentage of time that the system is up and running, also known as its uptime.

Consistency, on the other hand, refers to the property that all clients see the same data at the same time. This is important for maintaining the integrity of the data stored in the system.

In distributed systems, it is often a trade-off between availability and consistency. Systems that prioritize high availability may sacrifice consistency, while systems that prioritize consistency may sacrifice availability. Different distributed systems use different approaches to balance the trade-off between availability and consistency, such as using replication or consensus algorithms.

Have a look at the following resources to learn more:

Free Resources

---

- [OpenSourceCAP FAQ](https://github.com/henryr/cap-faq)
- [articleCAP Theorem Revisited](https://robertgreiner.com/cap-theorem-revisited/)
- [articleA plain english introduction to CAP Theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem)
- [videoCAP Theorem](https://www.youtube.com/watch?v=_RbsFXWRZ10&t=1s)