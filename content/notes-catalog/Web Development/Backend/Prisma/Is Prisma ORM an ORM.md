---
id: 01J5TVNVHPXP8TBBH2ND69Y58N
title: Is Prisma ORM an ORM?
modified: 2024-08-21T12:18:39-04:00
description: Discussing if Prisma ORM is technically an ORM
tags:
  - prisma
  - schema
  - orm
  - api
  - programming
---
# Is Prisma ORM an ORM?

To answer the question briefly: _Yes, Prisma ORM is a new kind of ORM that fundamentally differs from traditional ORMs and doesn't suffer from many of the problems commonly associated with these_.

Traditional ORMs provide an object-oriented way for working with relational databases by mapping tables to _model classes_ in your programming language. This approach leads to many problems that are caused by the [object-relational impedance mismatch](https://en.wikipedia.org/wiki/Object%E2%80%93relational_impedance_mismatch).

Prisma ORM works fundamentally different compared to that. With Prisma ORM, you define your models in the declarative [Prisma schema](https://www.prisma.io/docs/orm/prisma-schema) which serves as the single source of truth for your database schema and the models in your programming language. In your application code, you can then use Prisma Client to read and write data in your database in a type-safe manner without the overhead of managing complex model instances. This makes the process of querying data a lot more natural as well as more predictable since Prisma Client always returns plain JavaScript objects.

In this article, you will learn in more detail about ORM patterns and workflows, how Prisma ORM implements the Data Mapper pattern, and the benefits of Prisma ORM's approach.

## What are ORMs?[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#what-are-orms "Direct link to What are ORMs?")

If you're already familiar with ORMs, feel free to jump to the [next section](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#prisma-orm) on Prisma ORM.

### ORM Patterns - Active Record and Data Mapper[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#orm-patterns---active-record-and-data-mapper "Direct link to ORM Patterns - Active Record and Data Mapper")

ORMs provide a high-level database abstraction. They expose a programmatic interface through objects to create, read, delete, and manipulate data while hiding some of the complexity of the database.

The idea with ORMs is that you define your models as **classes** that map to tables in a database. The classes and their instances provide you with a programmatic API to read and write data in the database.

There are two common ORM patterns: [_Active Record_](https://en.wikipedia.org/wiki/Active_record_pattern) and [_Data Mapper_](https://en.wikipedia.org/wiki/Data_mapper_pattern) which differ in how they transfer data between objects and the database. While both patterns require you to define classes as the main building block, the most notable difference between the two is that the Data Mapper pattern decouples in-memory objects in the application code from the database and uses the data mapper layer to transfer data between the two. In practice, this means that with Data Mapper the in-memory objects (representing data in the database) don't even know that there’s a database present.

#### Active Record[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#active-record "Direct link to Active Record")

_Active Record_ ORMs map model classes to database tables where the structure of the two representations is closely related, e.g. each field in the model class will have a matching column in the database table. Instances of the model classes wrap database rows and carry both the data and the access logic to handle persisting changes in the database. Additionally, model classes can carry business logic specific to the data in the model.

The model class typically has methods that do the following:

- Construct an instance of the model from an SQL query.
- Construct a new instance for later insertion into the table.
- Wrap commonly used SQL queries and return Active Record objects.
- Update the database and insert into it the data in the Active Record.
- Get and set the fields.
- Implement business logic.

#### Data Mapper[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#data-mapper "Direct link to Data Mapper")

_Data Mapper_ ORMs, in contrast to Active Record, decouple the application's in-memory representation of data from the database's representation. The decoupling is achieved by requiring you to separate the mapping responsibility into two types of classes:

- **Entity classes**: The application's in-memory representation of entities which have no knowledge of the database
- **Mapper classes**: These have two responsibilities:
    - Transforming the data between the two representations.
    - Generating the SQL necessary to fetch data from the database and persist changes in the database.

Data Mapper ORMs allow for greater flexibility between the problem domain as implemented in code and the database. This is because the data mapper pattern allows you to hide the ways in which your database is implemented which isn’t an ideal way to think about your domain behind the whole data-mapping layer.

One of the reasons that traditional data mapper ORMs do this is due to the structure of organizations where the two responsibilities would be handled by separate teams, e.g., [DBAs](https://en.wikipedia.org/wiki/Database_administrator) and backend developers.

In reality, not all Data Mapper ORMs adhere to this pattern strictly. For example, [TypeORM](https://github.com/typeorm/typeorm/blob/master/docs/active-record-data-mapper.md#what-is-the-data-mapper-pattern), a popular ORM in the TypeScript ecosystem which supports both Active Record and Data Mapper, takes the following approach to Data Mapper:

- Entity classes use decorators (`@Column`) to map class properties to table columns and are aware of the database.
- Instead of mapper classes, _repository_ classes are used for querying the database and may contain custom queries. Repositories use the decorators to determine the mapping between entity properties and database columns.

Given the following `User` table in the database:

![user-table](data:image/png;base64,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)

This is what the corresponding entity class would look like:

```
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm'@Entity()export class User {  @PrimaryGeneratedColumn()  id: number  @Column({ name: 'first_name' })  firstName: string  @Column({ name: 'last_name' })  lastName: string  @Column({ unique: true })  email: string}
```

### Schema migration workflows[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#schema-migration-workflows "Direct link to Schema migration workflows")

A central part of developing applications that make use of a database is changing the database schema to accommodate new features and to better fit the problem you're solving. In this section, we'll discuss what [schema migrations](https://www.prisma.io/dataguide/types/relational/what-are-database-migrations) are and how they affect the workflow.

Because the ORM sits between the developer and the database, most ORMs provide a **migration tool** to assist with the creation and modification of the database schema.

A migration is a set of steps to take the database schema from one state to another. The first migration usually creates tables and indices. Subsequent migrations may add or remove columns, introduce new indices, or create new tables. Depending on the migration tool, the migration may be in the form of SQL statements or programmatic code which will get converted to SQL statements (as with [ActiveRecord](https://guides.rubyonrails.org/active_record_migrations.html) and [SQLAlchemy](https://alembic.sqlalchemy.org/en/latest/tutorial.html#create-a-migration-script)).

Because databases usually contain data, migrations assist you with breaking down schema changes into smaller units which helps avoid inadvertent data loss.

Assuming you were starting a project from scratch, this is what a full workflow would look like: you create a migration that will create the `User` table in the database schema and define the `User` entity class as in the example above.

Then, as the project progresses and you decide you want to add a new `salutation` column to the `User` table, you would create another migration which would alter the table and add the `salutation` column.

Let's take a look at how that would look like with a TypeORM migration:

```
import { MigrationInterface, QueryRunner } from 'typeorm'export class UserRefactoring1604448000 implements MigrationInterface {  async up(queryRunner: QueryRunner): Promise<void> {    await queryRunner.query(`ALTER TABLE "User" ADD COLUMN "salutation" TEXT`)  }  async down(queryRunner: QueryRunner): Promise<void> {    await queryRunner.query(`ALTER TABLE "User" DROP COLUMN "salutation"`)  }}
```

Once a migration is carried out and the database schema has been altered, the entity and mapper classes must also be updated to account for the new `salutation` column.

With TypeORM that means adding a `salutation` property to the `User` entity class:

```
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm'@Entity()export class User {  @PrimaryGeneratedColumn()  id: number  @Column({ name: 'first_name' })  firstName: string  @Column({ name: 'last_name' })  lastName: string  @Column({ unique: true })  email: string  @Column()  salutation: string}
```

Synchronizing such changes can be a challenge with ORMs because the changes are applied manually and are not easily verifiable programmatically. Renaming an existing column can be even more cumbersome and involve searching and replacing references to the column.

> **Note:** Django's [makemigrations](https://docs.djangoproject.com/en/3.1/ref/django-admin/#django-admin-makemigrations) CLI generates migrations by inspecting changes in models which, similar to Prisma ORM, does away with the synchronization problem.

In summary, evolving the schema is a key part of building applications. With ORMs, the workflow for updating the schema involves using a migration tool to create a migration followed by updating the corresponding entity and mapper classes (depending on the implementation). As you'll see, Prisma ORM takes a different approach to this.

Now that you've seen what migrations are and how they fit into the development workflows, you will learn more about the benefits and drawbacks of ORMs.

### Benefits of ORMs[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#benefits-of-orms "Direct link to Benefits of ORMs")

There are different reasons why developers choose to use ORMs:

- ORMs facilitate implementing the domain model. The domain model is an object model that incorporates the behavior and data of your business logic. In other words, it allows you to focus on real business concepts rather than the database structure or SQL semantics.
- ORMs help reduce the amount of code. They save you from writing repetitive SQL statements for common CRUD (Create Read Update Delete) operations and escaping user input to prevent vulnerabilities such as SQL injections.
- ORMs require you to write little to no SQL (depending on your complexity you may still need to write the odd raw query). This is beneficial for developers who are not familiar with SQL but still want to work with a database.
- Many ORMs abstract database-specific details. In theory, this means that an ORM can make changing from one database to another easier. It should be noted that in practice applications rarely change the database they use.

As with all abstractions that aim to improve productivity, there are also drawbacks to using ORMs.

### Drawbacks of ORMs[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#drawbacks-of-orms "Direct link to Drawbacks of ORMs")

The drawbacks of ORMs are not always apparent when you start using them. This section covers some of the commonly accepted ones:

- With ORMs, you form an object graph representation of database tables which may lead to the [object-relational impedance mismatch](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch). This happens when the problem you are solving forms a complex object graph which doesn't trivially map to a relational database. Synchronizing between two different representations of data, one in the relational database, and the other in-memory (with objects) is quite difficult. This is because objects are more flexible and varied in the way they can relate to each other compared to relational database records.
- While ORMs handle the complexity associated with the problem, the synchronization problem doesn't go away. Any changes to the database schema or the data model require the changes to be mapped back to the other side. This burden is often on the developer. In the context of a team working on a project, database schema changes require coordination.
- ORMs tend to have a large API surface due to the complexity they encapsulate. The flip side of not having to write SQL is that you spend a lot of time learning how to use the ORM. This applies to most abstractions, however without understanding how the database works, improving slow queries can be difficult.
- Some _complex queries_ aren't supported by ORMs due to the flexibility that SQL offers. This problem is alleviated by raw SQL querying functionality in which you pass the ORM a SQL statement string and the query is run for you.

Now that the costs and benefits of ORMs have been covered, you can better understand what Prisma ORM is and how it fits in.

## Prisma ORM[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#prisma-orm "Direct link to Prisma ORM")

Prisma ORM is a **next-generation ORM** that makes working with databases easy for application developers and features the following tools:

- [**Prisma Client**](https://www.prisma.io/docs/orm/prisma-client): Auto-generated and type-safe database client for use in your application.
- [**Prisma Migrate**](https://www.prisma.io/docs/orm/prisma-migrate): A declarative data modeling and migration tool.
- [**Prisma Studio**](https://www.prisma.io/docs/orm/tools/prisma-studio): A modern GUI for browsing and managing data in your database.

> **Note:** Since Prisma Client is the most prominent tool, we often refer to it as simply Prisma.

These three tools use the [Prisma schema](https://www.prisma.io/docs/orm/prisma-schema) as a single source of truth for the database schema, your application's object schema, and the mapping between the two. It's defined by you and is your main way of configuring Prisma ORM.

Prisma ORM makes you productive and confident in the software you're building with features such as _type safety_, rich auto-completion, and a natural API for fetching relations.

In the next section, you will learn about how Prisma ORM implements the Data Mapper ORM pattern.

### How Prisma ORM implements the Data Mapper pattern[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#how-prisma-orm-implements-the-data-mapper-pattern "Direct link to How Prisma ORM implements the Data Mapper pattern")

As mentioned earlier in the article, the Data Mapper pattern aligns well with organizations where the database and application are owned by different teams.

With the rise of modern cloud environments with managed database services and DevOps practices, more teams embrace a cross-functional approach, whereby teams own both the full development cycle including the database and operational concerns.

Prisma ORM enables the evolution of the DB schema and object schema in tandem, thereby reducing the need for deviation in the first place, while still allowing you to keep your application and database somewhat decoupled using `@map` attributes. While this may seem like a limitation, it prevents the domain model's evolution (through the object schema) from getting imposed on the database as an afterthought.

To understand how Prisma ORM's implementation of the Data Mapper pattern differs conceptually to traditional Data Mapper ORMs, here's a brief comparison of their concepts and building blocks:

|Concept|Description|Building block in traditional ORMs|Building block in Prisma ORM|Source of truth in Prisma ORM|
|---|---|---|---|---|
|Object schema|The in-memory data structures in your applications|Model classes|Generated TypeScript types|Models in the Prisma schema|
|Data Mapper|The code which transforms between the object schema and the database|Mapper classes|Generated functions in Prisma Client|@map attributes in the Prisma schema|
|Database schema|The structure of data in the database, e.g., tables and columns|SQL written by hand or with a programmatic API|SQL generated by Prisma Migrate|Prisma schema|

Prisma ORM aligns with the Data Mapper pattern with the following added benefits:

- Reducing the boilerplate of defining classes and mapping logic by generating a Prisma Client based on the Prisma schema.
- Eliminating the synchronization challenges between application objects and the database schema.
- Database migrations are a first-class citizen as they're derived from the Prisma schema.

Now that we've talked about the concepts behind Prisma ORM's approach to Data Mapper, we can go through how the Prisma schema works in practice.

### Prisma schema[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#prisma-schema "Direct link to Prisma schema")

At the heart of Prisma's implementation of the Data Mapper pattern is the _Prisma schema_ – a single source of truth for the following responsibilities:

- Configuring how Prisma connects to your database.
- Generating Prisma Client – the type-safe ORM for use in your application code.
- Creating and evolving the database schema with Prisma Migrate.
- Defining the mapping between application objects and database columns.

Models in Prisma ORM mean something slightly different to Active Record ORMs. With Prisma ORM, models are defined in the Prisma schema as abstract entities which describe tables, relations, and the mappings between columns to properties in Prisma Client.

As an example, here's a Prisma schema for a blog:

```
datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}model Post {  id        Int     @id @default(autoincrement())  title     String  content   String? @map("post_content")  published Boolean @default(false)  author    User?   @relation(fields: [authorId], references: [id])  authorId  Int?}model User {  id    Int     @id @default(autoincrement())  email String  @unique  name  String?  posts Post[]}
```

Here's a break down of the example above:

- The `datasource` block defines the connection to the database.
- The `generator` block tells Prisma ORM to generate Prisma Client for TypeScript and Node.js.
- The `Post` and `User` models map to database tables.
- The two models have a _1-n_ relation where each `User` can have many related `Post`s.
- Each field in the models has a type, e.g. the `id` has the type `Int`.
- Fields may contain field attributes to define:
    - Primary keys with the `@id` attribute.
    - Unique keys with the `@unique` attribute.
    - Default values with the `@default` attribute.
    - Mapping between table columns and Prisma Client fields with the `@map` attribute, e.g., the `content` field (which will be accessible in Prisma Client) maps to the `post_content` database column.

The `User` / `Post` relation can be visualized with the following diagram:

![1-n relation between User and Post](https://www.prisma.io/docs/assets/images/user-post-relation-1-n-bb9740243c22ae8f3685667ee2cce56f.png)

At a Prisma ORM level, the `User` / `Post` relation is made up of:

- The scalar `authorId` field, which is referenced by the `@relation` attribute. This field exists in the database table – it is the foreign key that connects Post and User.
- The two relation fields: `author` and `posts` **do not exist** in the database table. Relation fields define connections between models at the Prisma ORM level and exist only in the Prisma schema and generated Prisma Client, where they are used to access the relations.

The declarative nature of Prisma schema is concise and allows defining the database schema and corresponding representation in Prisma Client.

In the next section, you will learn about Prisma ORM's supported workflows.

### Prisma ORM workflow[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#prisma-orm-workflow "Direct link to Prisma ORM workflow")

The workflow with Prisma ORM is slightly different to traditional ORMs. You can use Prisma ORM when building new applications from scratch or adopt it incrementally:

- _New application_ (greenfield): Projects that have no database schema yet can use Prisma Migrate to create the database schema.
- _Existing application_ (brownfield): Projects that already have a database schema can be [introspected](https://www.prisma.io/docs/orm/prisma-schema/introspection) by Prisma ORM to generate the Prisma schema and Prisma Client. This use-case works with any existing migration tool and is useful for incremental adoption. It's possible to switch to Prisma Migrate as the migration tool. However, this is optional.

With both workflows, the Prisma schema is the main configuration file.

#### Workflow for incremental adoption in projects with an existing database[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#workflow-for-incremental-adoption-in-projects-with-an-existing-database "Direct link to Workflow for incremental adoption in projects with an existing database")

Brownfield projects typically already have some database abstraction and schema. Prisma ORM can integrate with such projects by introspecting the existing database to obtain a Prisma schema that reflects the existing database schema and to generate Prisma Client. This workflow is compatible with any migration tool and ORM which you may already be using. If you prefer to incrementally evaluate and adopt, this approach can be used as part of a [parallel adoption strategy](https://en.wikipedia.org/wiki/Parallel_adoption).

A non-exhaustive list of setups compatible with this workflow:

- Projects using plain SQL files with `CREATE TABLE` and `ALTER TABLE` to create and alter the database schema.
- Projects using a third party migration library like [db-migrate](https://github.com/db-migrate/node-db-migrate) or [Umzug](https://github.com/sequelize/umzug).
- Projects already using an ORM. In this case, database access through the ORM remains unchanged while the generated Prisma Client can be incrementally adopted.

In practice, these are the steps necessary to introspect an existing DB and generate Prisma Client:

1. Create a `schema.prisma` defining the `datasource` (in this case, your existing DB) and `generator`:

```
datasource db {  provider = "postgresql"  url      = "postgresql://janedoe:janedoe@localhost:5432/hello-prisma"}generator client {  provider = "prisma-client-js"}
```

2. Run `prisma db pull` to populate the Prisma schema with models derived from your database schema.
3. (Optional) Customize [field and model mappings](https://www.prisma.io/docs/orm/prisma-schema/data-model/models#mapping-model-names-to-tables-or-collections) between Prisma Client and the database.
4. Run `prisma generate`.

Prisma ORM will generate Prisma Client inside the `node_modules` folder, from which it can be imported in your application. For more extensive usage documentation, see the [Prisma Client API](https://www.prisma.io/docs/orm/prisma-client) docs.

To summarize, Prisma Client can be integrated into projects with an existing database and tooling as part of a parallel adoption strategy. New projects will use a different workflow detailed next.

#### Workflow for new projects[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#workflow-for-new-projects "Direct link to Workflow for new projects")

Prisma ORM is different from ORMs in terms of the workflows it supports. A closer look at the steps necessary to create and change a new database schema is useful for understanding Prisma Migrate.

Prisma Migrate is a CLI for declarative data modeling & migrations. Unlike most migration tools that come as part of an ORM, you only need to describe the current schema, instead of the operations to move from one state to another. Prisma Migrate infers the operations, generates the SQL and carries out the migration for you.

This example demonstrates using Prisma ORM in a new project with a new database schema similar to the blog example above:

1. Create the Prisma schema:

```
// schema.prismadatasource db {  provider = "postgresql"  url      = "postgresql://janedoe:janedoe@localhost:5432/hello-prisma"}generator client {  provider = "prisma-client-js"}model Post {  id        Int     @id @default(autoincrement())  title     String  content   String? @map("post_content")  published Boolean @default(false)  author    User?   @relation(fields: [authorId], references: [id])  authorId  Int?}model User {  id    Int     @id @default(autoincrement())  email String  @unique  name  String?  posts Post[]}
```

2. Run `prisma migrate` to generate the SQL for the migration, apply it to the database, and generate Prisma Client.

For any further changes to the database schema:

1. Apply changes to the Prisma schema, e.g., add a `registrationDate` field to the `User` model
2. Run `prisma migrate` again.

The last step demonstrates how declarative migrations work by adding a field to the Prisma schema and using Prisma Migrate to transform the database schema to the desired state. After the migration is run, Prisma Client is automatically regenerated so that it reflects the updated schema.

If you don't want to use Prisma Migrate but still want to use the type-safe generated Prisma Client in a new project, see the next section.

##### Alternative for new projects without Prisma Migrate[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#alternative-for-new-projects-without-prisma-migrate "Direct link to Alternative for new projects without Prisma Migrate")

It is possible to use Prisma Client in a new project with a third-party migration tool instead of Prisma Migrate. For example, a new project could choose to use the Node.js migration framework [db-migrate](https://github.com/db-migrate/node-db-migrate) to create the database schema and migrations and Prisma Client for querying. In essence, this is covered by the [workflow for existing databases](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#workflow-for-incremental-adoption-in-projects-with-an-existing-database).

## Accessing data with Prisma Client[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#accessing-data-with-prisma-client "Direct link to Accessing data with Prisma Client")

So far, the article covered the concepts behind Prisma ORM, its implementation of the Data Mapper pattern, and the workflows it supports. In this last section, you will see how to access data in your application using Prisma Client.

Accessing the database with Prisma Client happens through the query methods it exposes. All queries return plain old JavaScript objects. Given the blog schema from above, fetching a user looks as follows:

```
import { PrismaClient } from '@prisma/client'const prisma = new PrismaClient()const user = await prisma.user.findUnique({  where: {    email: 'alice@prisma.io',  },})
```

In this query, the `findUnique()` method is used to fetch a single row from the `User` table. By default, Prisma ORM will return all the scalar fields in the `User` table.

> **Note:** The example uses TypeScript to make full use of the type safety features offered by Prisma Client. However, Prisma ORM also works with [JavaScript in Node.js](https://dev.to/prisma/productive-development-with-prisma-s-zero-cost-type-safety-4od2).

Prisma Client maps queries and results to [structural types](https://en.wikipedia.org/wiki/Structural_type_system) by generating code from the Prisma schema. This means that `user` has an associated type in the generated Prisma Client:

```
export type User = {  id: number  email: string  name: string | null}
```

This ensures that accessing a non-existent field will raise a type error. More broadly, it means that the result's type for every query is known ahead of running the query, which helps catch errors. For example, the following code snippet will raise a type error:

```
console.log(user.lastName) // Property 'lastName' does not exist on type 'User'.
```

### Fetching relations[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#fetching-relations "Direct link to Fetching relations")

Fetch relations with Prisma Client is done with the `include` option. For example, to fetch a user and their posts would be done as follows:

```
const user = await prisma.user.findUnique({  where: {    email: 'alice@prisma.io',  },  include: {    posts: true,  },})
```

With this query, `user`'s type will also include `Post`s which can be accessed with the `posts` array field:

```
console.log(user.posts[0].title)
```

The example only scratches the surface of Prisma Client's API for [CRUD operations](https://www.prisma.io/docs/orm/prisma-client/queries/crud) which you can learn more about in the docs. The main idea is that all queries and results are backed by types and you have full control over how relations are fetched.

## Conclusion[​](https://www.prisma.io/docs/orm/overview/prisma-in-your-stack/is-prisma-an-orm#conclusion "Direct link to Conclusion")

In summary, Prisma ORM is a new kind of Data Mapper ORM that differs from traditional ORMs and doesn't suffer from the problems commonly associated with them.

Unlike traditional ORMs, with Prisma ORM, you define the Prisma schema – a declarative single source of truth for the database schema and application models. All queries in Prisma Client return plain JavaScript objects which makes the process of interacting with the database a lot more natural as well as more predictable.

Prisma ORM supports two main workflows for starting new projects and adopting in an existing project. For both workflows, your main avenue for configuration is via the Prisma schema.

Like all abstractions, both Prisma ORM and other ORMs hide away some of the underlying details of the database with different assumptions.

These differences and your use case all affect the workflow and cost of adoption. Hopefully understanding how they differ can help you make an informed decision.