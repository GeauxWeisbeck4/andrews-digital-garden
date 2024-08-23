---
id: 01J5V8056SCXB74Y042HDDVAPN
title: Nest.js Blog API Tutorial
modified: 2024-08-21T17:13:50-04:00
description: Creating a blog API with Nest.js, Prisma, and PostgreSQL
tags:
  - nestjs
  - prisma
  - postgresql
  - api
  - rest-api
  - tutorial
  - project
---
# Nest.js Blog API  Tutorial
...
## Seed the database
```typescript
// import prisma client
// I
```
- initialize prisma client
- create main async function that creates two dummy articles
	- const post1 = await prisma.article.upsert({ where: { title: }, update: {}, create: { title: , body: , description: , published: , }, });
	- ...post2
	- console.log({ post1, post2 })
- Execute main function with catch error and finally async await prisma.#disconnet();
![[Pasted image 20240821170905.png]]

Get drafts shot
![[Pasted image 20240821171339.png]]
