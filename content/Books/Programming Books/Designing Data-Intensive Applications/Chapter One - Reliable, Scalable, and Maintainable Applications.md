---
id: 01J724DEFFC345XM0CAVQBAMVX
modified: 2024-09-05T18:38:48-04:00
title: Chapter One - Reliable, Scalable, and Maintainable Applications
description: Learning about reliable scalable and maintainable applications
---
# Reliability
- Application performs the function user expected
- Tolerates user making mistakes or using software in unexpected ways
- Performance is good enough for required use case, under the expected load and data volume
- System prevents any unauthorized access and abuse
Things that can go wrong are called faults and systems that can handle faults are fault tolerant.
Failure is when the whole system stops providing the required service to a user.
- ## Hardware Faults
	- Hard disks are reported as having a mean time to failure of about 10 to 50 years. One storage cluster with 10,000 disks should expect on average one disk to die a day.
	- What do we do? Typically we add redundancy to the individual hardware components in order to reduce the failure rate of the system.
		- Disks may be set up in a RAID configuration
		- Servers have dual power supplies and hot-swappable CPUS
		- Datacenters may have batteries and diesel generators for backup power
- ## Software Faults
	- Typically thought of as random and independent from each other - one machine's disk failing does not imply that another machine's disk is going to fail
		- A systematic error within the system is typically harder to anticipate because they are correlated across nodes. Examples:
			- Software bug that causes every instance of an application server to crash when given a particular bad input. The leap second on June 30, 2012 caused many applications to hang simultaneously due to a bug in the Linux kernel.
			- Runaway process that uses up some shared resources, CPU time memory, disk space, or network bandwith
			- Service that the system depends on that slows down, becomes unresponsive, or starts returning corrupted responses
			- Cascading failures where a small fault in one component triggers a fault in another component
- ## Human Errors
	- How can we prevent human errors?
		- Design systems in a way that minimizes opportunities for error. Well-designed abstractions, APIs, and admin interfaces make it easy to do "the right thing" and discourage "the wrong thing." Can't be too restrictive or people will work around them, negating benefit - got to have balance.
		- Decouple places where people make most mistakes from places where they can cause failures. Use sandboxes.
		- Test thoroughly at all levels from unit tests to whole-system integration tests and manual tests. Automated testing is also widely used.
		- Allow quick and easy recovery from human erros, to miimize the impact in case of failure. Make it easy to roll back configuration changes, roll out new code gradually, and provide tools to recompute data.
		- Set up detailed and clear monitoring such as performan emetrics and error rates. This is referred to as telemetry
		- Implement good management practices and training.
# Scalability 
- System's ability to cope with increased load.
- ## Describing Load
	- Load parameters
	- Depends on th earchitecture of your system - could be requests per second to a web server, the ratio of reads to writes in a database, number of simultaneously active users in a chat room, the hit rate on a cache, or something else.
	- ### Twitter Example
		- Post Tweet
			- User can publish a new message to their followers (4.6k requests/sec on average, over 12k requests/sec at peak)
		- Home timeline 
			- User can view tweets posted by people they follow (300k requests/sec
		- Handling 12,000 writes per second would be fairly easy. Twitter's scaling challenge is not due to tweet volume but due to fan-out: each user follows many people and each user is followed by many people. 
		- Two methods for implementing these tow operations
			- 1. POsting a tweet simply inserts the new tweet into a global collection of tweets. When a user requests their ome tiemline, look up all the people they follow, find all the tweets for each of those users, and merge them (sorted by time.) Here is how it would look in SQL
```sql
SELECT tweets.*, users.* FROM tweets
  JOIN users   ON tweets.sender_id    = users.id
  JOIN follows ON follows.followee_id = users.id
  WHERE follows.follower_id = current_user
```