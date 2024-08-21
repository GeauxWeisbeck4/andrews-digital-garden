---
id: 01J5TS2AN1YB6Y20AWT0185B02
title: What is Prisma?
modified: 2024-08-21T12:38:19-04:00
tags:
  - prisma
  - backend
  - api
  - schema
---

# What is Prisma?
- Prisma ORM is an [open-source](https://github.com/prisma/prisma) next-generation ORM. It consists of the following parts:
	- **Prisma Client**: Auto-generated and type-safe query builder for Node.js & TypeScript
	- **Prisma Migrate**: Migration system
	- **Prisma Studio**: GUI to view and edit data in your database.
- Prisma Client can be used in _any_ Node.js (supported versions) or TypeScript backend application (including serverless applications and microservices). This can be a [REST API](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/rest), a [GraphQL API](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/graphql), a gRPC API, or anything else that needs a database.
- ## How does Prisma work?
	- ### The Prisma schema[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#the-prisma-schema "Direct link to The Prisma schema")
		- Every project that uses a tool from the Prisma ORM toolkit starts with a [Prisma schema](https://www.prisma.io/docs/orm/prisma-schema). The Prisma schema allows developers to define their _application models_ in an intuitive data modeling language. It also contains the connection to a database and defines a _generator_
		- #### [[PostgreSQL]]
```postgresql
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields: [authorId], references: [id])
  authorId  Int?
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```
- #### [[MongoDB]]
```mongodb
datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Post {
  id        String  @id @default(auto()) @map("_id") @db.ObjectId
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields: [authorId], references: [id])
  authorId  String  @db.ObjectId
}

model User {
  id    String  @id @default(auto()) @map("_id") @db.ObjectId
  email String  @unique
  name  String?
  posts Post[]
}
```

> [!NOTE] Note
> The Prisma schema has powerful data modeling features. For example, it allows you to define "Prisma-level" [relation fields](https://www.prisma.io/docs/orm/prisma-schema/data-model/relations) which will make it easier to work with [relations in the Prisma Client API](https://www.prisma.io/docs/orm/prisma-client/queries/relation-queries). In the case above, the `posts` field on `User` is defined only on "Prisma-level", meaning it does not manifest as a foreign key in the underlying database.

**In this schema, you configure three things:**
- **Data source**: Specifies your database connection (via an environment variable)
- **Generator**: Indicates that you want to generate Prisma Client
- **Data model**: Defines your application models

### The Prisma schema data model[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#the-prisma-schema-data-model "Direct link to The Prisma schema data model")

#### Functions of Prisma schema data models[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#functions-of-prisma-schema-data-models "Direct link to Functions of Prisma schema data models")

The data model is a collection of [models](https://www.prisma.io/docs/orm/prisma-schema/data-model/models#defining-models). A model has two major functions:

- Represent a table in relational databases or a collection in MongoDB
- Provide the foundation for the queries in the Prisma Client API

#### Getting a data model[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#getting-a-data-model "Direct link to Getting a data model")

There are two major workflows for "getting" a data model into your Prisma schema:

- Manually writing the data model and mapping it to the database with [Prisma Migrate](https://www.prisma.io/docs/orm/prisma-migrate)
- Generating the data model by [introspecting](https://www.prisma.io/docs/orm/prisma-schema/introspection) a database

Once the data model is defined, you can [generate Prisma Client](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/generating-prisma-client) which will expose CRUD and more queries for the defined models. If you're using TypeScript, you'll get full type-safety for all queries (even when only retrieving the subsets of a model's fields).

### Accessing your database with Prisma Client[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#accessing-your-database-with-prisma-client "Direct link to Accessing your database with Prisma Client")

- #### Generating Prisma Client[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#generating-prisma-client "Direct link to Generating Prisma Client")
	- The first step when using Prisma Client is installing the `@prisma/client` npm package:
```bash
npm install @prisma/client
```
- Installing the `@prisma/client` package invokes the `prisma generate` command, which reads your Prisma schema and _generates_ Prisma Client code. The code is [generated into the `node_modules/.prisma/client` folder by default](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/generating-prisma-client#the-prismaclient-npm-package).
	- After you change your data model, you'll need to manually re-generate Prisma Client to ensure the code inside `node_modules/.prisma/client` gets updated:
```bash
$ prisma generate
```
- #### Using Prisma Client to send queries to your database[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#using-prisma-client-to-send-queries-to-your-database "Direct link to Using Prisma Client to send queries to your database")
	- Once Prisma Client has been generated, you can import it in your code and send queries to your database. This is what the setup code looks like.
- ##### Import and instantiate Prisma Client
```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()
```
- Now you can start sending queries via the generated Prisma Client API, here are a few sample queries. Note that all Prisma Client queries return _plain old JavaScript objects_.
	- Learn more about the available operations in the [Prisma Client API reference](https://www.prisma.io/docs/orm/prisma-client).
	- #### Retrieve all `User` records from the database
```typescript
// Run inside `async` function
const allUsers = await prisma.user.findMany()
```
- #### Include the `posts` relation on each returned `User` object[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#include-the-posts-relation-on-each-returned-user-object "Direct link to include-the-posts-relation-on-each-returned-user-object")
```typescript
// Run inside `async` function
const allUsers = await prisma.user.findMany({
  include: { posts: true },
})
```
- ##### Filter all `Post` records that contain `"prisma"`[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#filter-all-post-records-that-contain-prisma "Direct link to filter-all-post-records-that-contain-prisma")
```typescript
// Run inside `async` function
const filteredPosts = await prisma.post.findMany({
  where: {
    OR: [
      { title: { contains: 'prisma' } },
      { content: { contains: 'prisma' } },
    ],
  },
})
```
- ##### Create a new `User` and a new `Post` record in the same query
```typescript
// Run inside `async` function
const user = await prisma.user.create({
  data: {
    name: 'Alice',
    email: 'alice@prisma.io',
    posts: {
      create: { title: 'Join us for Prisma Day 2020' },
    },
  },
})
```
- ##### Update an existing `Post` record[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#update-an-existing-post-record "Direct link to update-an-existing-post-record")
```typescript
// Run inside `async` function
const post = await prisma.post.update({
  where: { id: 42 },
  data: { published: true },
})
```
- #### Usage with TypeScript[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#usage-with-typescript "Direct link to Usage with TypeScript")
	- Note that when using TypeScript, the result of this query will be _statically typed_ so that you can't accidentally access a property that doesn't exist (and any typos are caught at compile-time). Learn more about leveraging Prisma Client's generated types on the [Advanced usage of generated types](https://www.prisma.io/docs/orm/prisma-client/type-safety/operating-against-partial-structures-of-model-types) page in the docs.
- ## Typical Prisma ORM workflows[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#typical-prisma-orm-workflows "Direct link to Typical Prisma ORM workflows")
	- As mentioned above, there are two ways for "getting" your data model into the Prisma schema. Depending on which approach you choose, your main Prisma ORM workflow might look different.
	- ### Prisma Migrate[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#prisma-migrate "Direct link to Prisma Migrate")
		- With **Prisma Migrate**, Prisma ORM's integrated database migration tool, the workflow looks as follows:
			1. Manually adjust your [Prisma schema data model](https://www.prisma.io/docs/orm/prisma-schema/data-model/models)
			2. Migrate your development database using the `prisma migrate dev` CLI command
			3. Use Prisma Client in your application code to access your database
![[Pasted image 20240821113059.png]]
- To learn more about the Prisma Migrate workflow, see:
	- [Deploying database changes with Prisma Migrate](https://www.prisma.io/docs/orm/prisma-client/deployment/deploy-database-changes-with-prisma-migrate)
	- [Developing with Prisma Migrate](https://www.prisma.io/docs/orm/prisma-migrate)
- ### SQL migrations and introspection[​](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma#sql-migrations-and-introspection "Direct link to SQL migrations and introspection")
	- If for some reason, you can not or do not want to use Prisma Migrate, you can still use introspection to update your Prisma schema from your database schema. The typical workflow when using **SQL migrations and introspection** is slightly different:
		1. Manually adjust your database schema using SQL or a third-party migration tool
		2. (Re-)introspect your database
		3. Optionally [(re-)configure your Prisma Client API](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/custom-model-and-field-names))
		4. (Re-)generate Prisma Client
		5. Use Prisma Client in your application code to access your database
![[Pasted image 20240821113119.png]]
- To learn more about the introspection workflow, please refer the [introspection section](https://www.prisma.io/docs/orm/prisma-schema/introspection).

> [!NOTE] Navigation
> [[Prisma]]
> **Introduction**
> -> [[What is Prisma|What is Prisma?]]
> [[Why Prisma ORM|Why Prisma ORM?]]
> [[Should you use Prisma ORM|Should you use Prisma ORM?]]
> [[Data Modeling and Prisma]]
> **Prisma ORM in Your Stack**
> [[REST and Prisma]]
> [[GraphQL and Prisma]]
> [[Personal Blog]]
> [[Is Prisma ORM an ORM|Is Prisma ORM an ORM?]]
> **Databases**
> [[Databases and Prisma]]
> **Beyond Prisma ORM**
> [[Beyond Prisma ORM]]

