---
id: 01JGFP01AF8M5BSSHXN0TM9BVG
modified: 2024-12-31T19:59:44-05:00
---
# Chapter 3. Building the Data Schema

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 3rd chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

The data layer covers a lot, so it’s awesome that we have so many good tools to work with. Regardless of the tools you choose, you do need to understand what’s going on beneath the surface. When you start building your data schema, take your time to really write it out. The data schema drives everything for the app and any of its dependencies. Over time, it can become hard to update the schema without breaking everything.

Since you’ve already decided what kind of backend architecture to go with, you probably have an idea of how data will be related and what that data looks like. Making a diagram of how the data will be connected is going to help you and the team a lot because you’ll see the relationships.

That’s why this chapter is going to cover:

- Initial considerations for setting up the data schema
    
- Setting up a database
    
- Using object relational mapping (ORM) tools
    
- Writing data migrations in your database
    
- Seeding the database with initial data
    

You want to be as detailed as you can and document as much of the business logic as possible. You’ll also want to get feedback from the team in small intervals because the schema can get complex as the app grows. Make sure everyone understands how and where data is being stored and why. By the time you finish this chapter, you’ll have a good foundation for how to build out your data schema.

# Initial Considerations

To build out your data layer, you’ll go through these basic steps:

1. Make a diagram for a data schema. This should include the entities, their columns and data types, and their relationships.
    
2. Set up the database connection in the app. This example will use [Prisma](https://www.prisma.io/docs/orm/overview/introduction/what-is-prisma) to connect to a [PostgreSQL](https://www.postgresql.org/) database, but there are other popular tools you can use, such as [Knex.js](https://knexjs.org/) or [Drizzle](https://github.com/drizzle-team/drizzle-orm). Some reasons to use Postgres are because it is open source and has a long history of reliability. You’ll find it behind huge, complex apps that have been in production for decades as well as newer apps that have just been released.
    
3. Write the data schema in the app by translating your diagram into code.
    
4. Add seed data. This is to ensure that your database has the essential data it needs from the beginning. This is also a good way to add data to test in different environments.
    
5. Run migrations. After the connection is established and the schema and seed data is ready, you need to run a migration to get these changes to the database.
    
6. Test the database with simple SQL queries. Check that the tables are creating, updating, and storing data as expected. Double-check the relationships between tables by looking at primary and foreign keys.
    

Some apps will have a more complex scenario than this, but you’ll see a process similar to this across all projects. You already know what data you’re expecting based on your conversations with Product, so now it’s time to make a good diagram for the dev team.

###### NOTE

We’re not going to discuss nonrelational databases in this book because we’re going to work with relational databases. Relational databases enforce strict rules between data whereas nonrelational databases don’t. Choosing between the two depends on the type of project you’re working on. If you want somewhere to store any format of data that comes in, such as with events that may have constantly changing information, nonrelational databases can give you more flexibility. Nonrelational databases have specific use cases, but many apps are fine with a normal relational database.

# Diagramming the Data Schema

A diagram is a great visual reference for developers and QA to understand what values to expect and why. It’s a good tool to spark discussions between the frontend and backend developers and to document relations between data in a noncode way. Again, these diagrams don’t have to be anything fancy. Developers tend to get hung up on little details in places that don’t matter, such as diagramming tools and image formats. You have to be self-aware enough to notice when you start diving too deep on a task that doesn’t need that much attention.

Your diagram can be simple as long as it has the tables, columns, data types, and relationships between tables. When possible, you want to use a tool that connects directly to the database to create the visualization, such as [DBeaver](https://github.com/dbeaver/dbeaver). That way, you can see the exact relationships you’ve defined. Or you can continue using Miro to keep all your architectural documentation in one place. The important thing is that it’s in a format that everyone can understand.

[Figure 3-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#user_table_columns), [Figure 3-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#order_table_columns), and [Figure 3-3](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#product_table_columns) show what some of the documentation for different tables can look like and how they can relate to one another.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0301.png)

###### Figure 3-1. User table columns

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0302.png)

###### Figure 3-2. Order table columns

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0303.png)

###### Figure 3-3. Product table columns

In [Figure 3-4](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#relationships_between_userscomma_orders), you can see how all the tables are related to one another.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0304.png)

###### Figure 3-4. Relationships between users, orders, and products

Now that you have documented the data schema, you’ll take this to the team, walk through what your thoughts are, and ask for feedback. The frontend devs may have specific data-format requests based on how they have to render elements. When it comes to the frontend, you typically shouldn’t plan the schema around it. It’s important to make sure that the queries the frontend makes, like searches and sorts, are thought of in the endpoint response, though. Other devs may bring up security considerations.

When you open this up for everyone to think about, you end up with a stronger schema than if one person handled it alone. You’ll learn how to be more confident with sharing your ideas and getting others to share theirs. Don’t be afraid to be wrong during these discussions because that might spark an idea for someone else.

This can be an uncomfortable place for a while because it feels like everyone is heavily scrutinizing your code and your technical skills when they aren’t. You have to get used to receiving constructive feedback; that’s what will help you and the team build better code. The more you present your code and thoughts to the team, the quicker you’ll be able to improve things for everyone.

With the data schema diagrammed and agreed on by the frontend and backend devs, you can start work on connecting the backend to the database to create the tables. You’ll be working with a Postgres instance locally, but this can also be hosted on a server in the cloud.

# Setting Up Postgres

You need to set up Postgres so that you can get the connection information for your app. This will normally be handled by the team that manages the infrastructure for your production and nonproduction environments. You’ll still need your own local instance to make sure the changes that you and the team are making work as expected. This is something you’d include in [a Docker container](https://hub.docker.com/_/postgres) if that’s how you want to keep the local environments consistent. Something else you’re responsible for thinking about is how to make onboarding smooth for new devs. Getting the local environment set up is one of the biggest hurdles for anyone joining a new team, so doing this will pay off as the team and the product grow.

You can [download Postgres for free](https://www.postgresql.org/download/). Follow the documentation to get everything up and running and set a master password. Doing some basic database security locally can help reveal potential issues early because you’ll already be thinking about a production environment. Once pgAdmin has finished installing, open the app to create a new database. Then click the PostgreSQL 14 drop-down menu and right-click Databases. This will give you the option to create a new database that you’ll name _dashboard_. It will look similar to [Figure 3-5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#dashboard_database_in_local_postgres) when you’re done.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0305.png)

###### Figure 3-5. Dashboard database in local Postgres

After you have the new dashboard database, you’ll need to remember the database name, database password, port number, and database username to set up the connection to the backend. Unless you choose something other than the default values, some of your credentials will be the following:

- Database username: postgres
    
- Database port number: 5432
    

Now you’re ready to start creating the tables you diagrammed earlier. You can move on to using an ORM tool to connect the backend to the database and make sure everything’s set up correctly.

# BASIC SQL QUERIES

It’s good to know some basic SQL commands. You don’t have to get super in-depth with things like views and indexing, but knowing enough to do create, read, update, and delete (CRUD) operations goes a long way. One way to get started is by writing a statement to insert a new row into a table. You can do that with the following SQL:

```
INSERT
```

Change `_table_name_` to the table you want to insert the data into. The `_column1_`, `_column2_`, and `_column3_` fields represent the column names you want to put the data into. Finally, `_value1_`, `_value2_`, and `_value3_` are the values you want to add to the respective columns. For testing purposes, you can add new row entries like this:

```
INSERT
```

To check that your data is stored like you expect it to be, you can query the data like this:

```
SELECT
```

To round things out, you may need to delete some data to clean up an example you’ve been working on. You can do that with something like this:

```
DELETE
```

If you can confidently use commands like these, you have enough SQL knowledge to double-check values directly in the database. Of course, you can dive deeper into SQL with resources like [SQLBolt](https://sqlbolt.com/) or [LearnSQL.com](https://learnsql.com/), but you don’t need to.

# Deciding What ORM to Use

Aside from choosing the framework for your backend, choosing the ORM tool is one of the big decisions you’ll make for the future of your project. It’s not an easy task to switch to a different ORM tool once you’ve started building. NestJS comes with [TypeORM](https://github.com/typeorm/typeorm), [Sequelize](https://sequelize.org/), and [Mongoose](https://mongoosejs.com/docs/) built in if you don’t have a preference. Other common ORM tools that you’ll be using include [Knex.js](https://knexjs.org/) and [Prisma](https://www.prisma.io/).

All these tools essentially do the same thing but with a different flavor. Choosing the one you and the team use will come down to everyone’s experience and comfort level as well as any limitations the tools may have. For this project, you’ve decided to go with Prisma because it’s the tool everyone on the team has used before, the documentation is well maintained, and tech teams across different projects in the industry use it. There’s a built-in ORM for NestJS, but Prisma has more support and a bigger community, and it works really well with Postgres. These reasons make it a strong candidate, which is why you and the team have selected it.

Now you’ll need to install Prisma and TSX as dev dependencies in your project with the following command:

npm install prisma @prisma/client tsx --save-dev

###### NOTE

There are some finer details you need to be aware of, especially concerning dependencies. When you’re adding a dependency to your project, make sure you understand what type of dependency it needs to be. There are three types of dependencies: dependencies (regular), development (dev) dependencies, and peer dependencies. _Dependencies_ are the packages your app needs to run after it’s been built for production. _Dev dependencies_ are the packages you need to do development work, like testing and linting, but they aren’t required for the app to work. You’ll learn more about _peer dependencies_ if you work on a project that needs to be published as its own package, but they are the packages that your package expects to be installed in a container app.

With Prisma installed in your project, you need to set up some configs to connect the app to your Postgres database. As with any ORM, you have to initialize it in your project. With Prisma, you can run the following command to do that. Remember, this output is an example of what you might see, and it could have changed with the latest version:

npx prisma init --datasource-provider postgresql

✔ Your Prisma schema was created at prisma/schema.prisma
  You can now open it `in` your favorite editor.

warn You already have a .gitignore file. Don’t forget to add `` ` ``.env`` ` `` `in` it to not commit any private information.

Next steps:
`1`. Set the DATABASE_URL `in` the .env file to point to your existing database. If your database has no tables yet, `read` https://pris.ly/d/getting-started
`2`. Run prisma db pull to turn your database schema into a Prisma schema.
`3`. Run prisma generate to generate the Prisma Client. You can `then` start querying your database.

More information `in` our documentation:
https://pris.ly/d/getting-started

###### TIP

Always read the console output after you run commands. It usually gives you useful advice!

This will create the _prisma_ directory and a _.env_ file in your project. Make sure the _.env_ file is in your _.gitignore_ first. Then, inside the _prisma_ directory, you’ll find _schema.prisma,_ which is where you’ll set up your database connection and the models for the app. The _.env_ file has a URL to your database, which is the connection string that will contain the database credentials. Update the value for `DATABASE_URL` in the _.env_ file with your local credentials. It might look something like this:

```
DATABASE_URL
```

Any values in your _.env_ should be handled in your CI/CD pipeline. Work with the DevOps team to get this in place depending on the infrastructure setup. Here’s a simple example of what that might look like with [GitHub Actions](https://docs.github.com/en/actions):

```
name
```

Now you can move over to the _schema.prisma_ file and start writing your model. You’ve already done the hard part of thinking out how everything relates, so now you can confidently start coding. You’ll notice this is where you can see one of the biggest differences between the ORM tools. Prisma has its own flavor that you’ll have to get used to, and you should refer to the docs often.

In your _schema.prisma,_ you can start adding pieces of your model. Add the following code to the end of the file:

```
// schema.prisma
```

These models represent the three tables you diagrammed earlier and their relationships. Building models with Prisma is similar to writing objects or type definitions with TypeScript. These models use special Prisma syntax for the types, but they match closely to the common types you work with. The most important thing to note is how relationships are defined between tables. On the Product table, there’s an associated `userId`. On the User table, we have an array of orders. This is how Prisma defines relationships between tables, and I highly encourage you to look through the documentation to learn about building more complex relationships.

You might also want to consider using the [Prisma VS Code plug-in](https://marketplace.visualstudio.com/items?itemName=Prisma.prisma) to help make development smoother. For this app, though, these few models will get you moving.

This completes the data schema for your app so far. Now it’s time to get this schema onto the database with a migration.

# Writing Migrations

_Migrations_ are the SQL queries made by the ORM based on your schema definition. When you connect to the database to run a migration, you’re essentially executing SQL statements. That’s what makes ORM tools so useful. Instead of having to manually write the SQL for multiple tables, you can write the query in TypeScript, and the ORM will translate it to SQL. JavaScript developers use these tools so that they don’t have to learn all the details of SQL and database quirks. Running a migration is also how you will check that your database is connected to your backend correctly. To run a migration with Prisma, you’ll open your console, navigate to the root of the project, and run this command:

prisma migrate dev --name initialize_dashboard_db

Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource “db”: PostgreSQL database “dashboard”, schema “public” at “localhost:5432”

Applying migration `` ` ``20230318132006_initialize_dashboard_db`` ` ``

The following migration`(`s`)` have been created and applied from new schema changes:

migrations/
  └─ 20230318132006_initialize_dashboard_db/
    └─ migration.sql

Your database is now `in` sync with your schema.

After the migration is successful, check your Postgres instance. You should see these tables and their columns populated in your dashboard database. Check that the tables have the columns you defined in your models. You should also see a new table called _OrderToProduct that defines the relationship you created between the Order and Product tables in your model. This is the fastest way to determine if your database connection is established. The migration would have failed if the connection didn’t exist, but looking directly at the database lets you know everything is fine.

Anytime you update your schema, you’ll need to make a new migration to update the database. Make sure to give it a descriptive name so that you can quickly understand what happened in the database history. This is super helpful if you run into data issues on the backend because it’s easier to see what changes have been made and when. Every migration will automatically generate the timestamp at the beginning of the folder name in Prisma. The timestamp is important for the database to know in what order to run the migrations when someone is trying to set up the database initially.

Other tools will handle migrations a little differently. Knex.js, for example, lets you write migrations in pure SQL if you want, and it generates migration files instead of folders. If you look in your project’s _prisma_ directory, you should see a _migrations_ folder with a subfolder. This is where the generated SQL for your migrations are. That’s the beauty of an ORM. You can write in syntax you’re familiar with, and it will generate the SQL for you. Take a look at the _migration.sql_ file in the migration folder. You’ll see something like this:

```
-- CreateTable
```

This is just the initial setup migration for the database. You’ll be adding more tables and columns and changing names as the app grows. When you run into issues with a migration, you can roll it back; turn to the [Prisma docs](https://www.prisma.io/docs/orm/prisma-migrate/workflows/generating-down-migrations) to learn how to do that. For now, you have what you need to add some seed data to the database so that you can start working on the rest of the app.

# Seeding the Database

Since this is your local database, you’ll eventually need some test data to play with. You can add that as part of the dev _.env_ setup and have it as part of the setup for the real database. To start, create a new _seed.ts_ file in the _prisma_ directory. This is where you’ll use the Prisma Client to create some records. In your _seed.ts_ file, add the following code:

```
// seed.ts
```

The complete _seed.ts_ file can be found on [GitHub](https://github.com/flippedcoder/dashboard-server/blob/main/prisma/seed.ts).

This code allows you to connect to the database and insert the new rows into their respective tables. Then, it disconnects from the database. To run this with Prisma, you need to add the `seed` config to your _package.json_ like this:

```
...
```

This tells Prisma where to look for your seed file and how to execute it. With the configs and some data ready, you can run this command to actually insert the data into your Postgres instance:

npx prisma db seed

Environment variables loaded from .env
Running seed `command` `` ` ``ts-node prisma/seed.ts`` ` `` ..

ߌᠠThe seed `command` has been executed.

If you look in your Postgres tables (see [Figure 3-6](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch03.html#dashboard_database_with_seeded_data_in)), you should see the values you wrote in _seed.ts._ This should be everything you need for your data concerns right now. Just to double-check, here’s a quick checklist you can go through when you think you’re finished setting everything up:

- Does the schema match the designs and functionality explained in the behavioral doc?
    
- Have you had at least one other dev look at it?
    
- Have you checked to make sure the schema works for all the apps consuming data from this database?
    

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0306.png)

###### Figure 3-6. Dashboard database with seeded data in local Postgres

There are more advanced things to consider as well that are out of scope for this book. Here’s a list of some of them and links to more resources:

- Does your schema give room for the app to grow? Resource: [_Software Architecture: The Hard Parts_](https://www.oreilly.com/library/view/software-architecture-the/9781492086888/) by Neal Ford, Mark Richards, Pramod Sadalage, and Zhamak Dehghani (O’Reilly)
    
- Is there a way to audit actions and the users who triggered them? Resource: [“What Is an Audit Trail? Everything You Need to Know”](https://www.auditboard.com/blog/what-is-an-audit-trail/) (Auditboard)
    
- Have you considered different user-role levels for table operation access? Resource: [“What Is Role-Based Access Control?”](https://www.imperva.com/learn/data-security/role-based-access-control-rbac/) (Imperva)
    

These advanced topics could have their own chapters or even entire books dedicated to them. There are a wide range of questions you can ask here, but to keep things moving forward at this stage, don’t get too deep into the details. If you can answer these questions, explain the schema decisions to another dev, and demo it to the Product team, you’re good to go for now.

Take notes and make tickets for any optimizations you see along the way. That way, you can come back and actually work on the details because it’s documented in a way that Product considers during sprint planning. You’ve decided that the schema is in a good place and the tickets regarding different endpoints can be unblocked. This is when you’ll start working on the API that different apps will interact with to get data from the database you just set up.

# Conclusion

In this chapter, you went through the process of drawing diagrams for your data, setting up a Postgres instance, and handling some initial setup with your ORM. You should feel pretty good about expanding your data schema from here because you’ve already answered a lot of questions about the future of the app.