---
id: 01JAC2QBX0BS5YECD2NXNAKTNF
title: Consistency Patterns
description: learning patterns about fuck knows what
modified: 2024-10-16T21:53:24-04:00
tags:
  - consistency
  - patterns
  - system-design
  - roadmaps
  - programming
---
# Consistency Patterns

Consistency patterns refer to the ways in which data is stored and managed in a distributed system, and how that data is made available to users and applications. There are three main types of consistency patterns:

- Strong consistency
- Weak consistency
- Eventual Consistency

Each of these patterns has its own advantages and disadvantages, and the choice of which pattern to use will depend on the specific requirements of the application or system.

# Weak Consistency

After an update is made to the data, it is not guaranteed that any subsequent read operation will immediately reflect the changes made. The read may or may not see the recent write.

# Eventual Consistency

Eventual consistency is a form of Weak Consistency. After an update is made to the data, it will be eventually visible to any subsequent read operations. The data is replicated in an asynchronous manner, ensuring that all copies of the data are eventually updated.

# Strong Consistency

After an update is made to the data, it will be immediately visible to any subsequent read operations. The data is replicated in a synchronous manner, ensuring that all copies of the data are updated at the same time.

Have a look at the following resources to learn more:

Free Resources



---

- [articleConsistency Patterns in Distributed Systems](https://cs.fyi/guide/consistency-patterns-week-strong-eventual/)