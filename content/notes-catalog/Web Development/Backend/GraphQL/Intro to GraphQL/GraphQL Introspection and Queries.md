---
id: 01J5HQJPR48EKBPC33YYGJWC5Q
modified: 2024-08-17T23:14:42-04:00
title: GraphQL Introspection and Queries
description: Learn what GraphQL Introspection and Queries are
tags:
  - graphql
  - introspection
  - query
  - schema
  - api
  - web-development
---
# GraphQL Introspection

A key feature and superpower of GraphQL is the Schema Introspection.

## What is GraphQL Introspection?

The GraphQL query language is strongly typed. Due to its strong type system, GraphQL gives you the ability to query and understand the underlying schema.

Thus, the Introspection feature allows you to query the schema and discover the available queries, mutations, subscriptions, types and fields in a specific GraphQL API.

The schema acts as a contract between the frontend and backend, improving the communication between them. But how does the frontend client know what the schema looks like? How do they prevent over-fetching or under-fetching? How do they know what operations are available? That's where the Introspection query helps.

## GraphQL Introspection Query

A server exposes the following introspection queries on the `Query` operation type.

- `__schema`
- `__type`
- `__typename`

Note that introspection queries start with `__`.

## Schema Introspection

Let's see some examples of introspective queries. We will query the field `__schema` field to find out the available queries, mutations and types.

### Fetch Available Queries

There are scenarios where you might want to see all the available queries in a GraphQL API. You can query the schema as follows:

Copy

{

  __schema {

    queryType {

      fields {

        name

        description

      }

    }

  }

}

The above query returns the name and description of all the available queries.

### Fetch Available Mutations

You can also fetch all the available mutations with the following query:

Copy

{

  __schema {

    mutationType {

      fields {

        name

        description

      }

    }

  }

}

Similar to the first query, it returns the name and description of all available mutations.

### Fetch Existing Types

Lastly, you can fetch all the types as follows:

Copy

{

  __schema {

    types {

      name

      description

    }

  }

}

This way, you can see all the types available in the GraphQL API. Similarly, you can retrieve all the available directives and subscriptions.

## GraphQL Introspection Security & Exploits

The Schema Introspection is a great feature and it can be really helpful, but it can cause problems too.

As you might remember, the Schema Introspection allows developers to query the API schema and see all the available resources. That means potential attackers can get a good understanding of your API and they can even get access to resources that are not meant to be publicly available. All this information available to potential attackers makes it easier to exploit your GraphQL API.

Should you disable the introspection query? Although it can be helpful, especially in the dev environment, turning off the introspection in production is recommended.

## Community Tooling

The ability to introspect is what allows the community to build great tooling around GraphQL. There's [GraphiQL](https://github.com/graphql/graphiql) and [GraphQL Playground](https://github.com/prisma-labs/graphql-playground) which leverages the Introspection feature to provide self-documentation to developers and try out APIs quickly.

The above tools use the `__schema` introspection query to give the schema documentation. You can explore more by trying out the `__schema` query to see the different selection sets, fields, and directives.

# GraphQL Queries - Fetching data

## What is a GraphQL Query?

In GraphQL, you fetch data with the help of queries. A query is a GraphQL Operation that allows you to retrieve specific data from the server.

Let’s look at the following GraphQL query:

Copy

{

  todos {

    title

  }

}

We ask the server for all the todos and their titles in the above query. The “`todos`" represents an object and "`title`" a **field**. All queries consist of an object and one or more fields.

The fields tell the server what information to return for the specified object.

## GraphQL Query Example

Executing the above query would return the following response:

Copy

{

  "data": {

    "todos": [

      {

        "title": "Learn GraphQL"

      },

      {

        “title": “Learn about queries"

      }

    ]

  }

}

The query and the result have the same format, demonstrating that you always get what you ask for in GraphQL. You only asked for the title, which means you will only get the title — nothing more, nothing less.

There are two other types of queries - anonymous and named.

1. Anonymous queries

Copy

query {

  todos {

    title

  }

}

This is similar to the first query.

2. Named queries

Copy

query getTodos {

  todos {

    title

  }

}

It’s considered best practice to name all your GraphQL operations because it helps people understand their purpose and helps when debugging.

In the next section, you will run queries and practice what you learnt.

## Try Out GraphQL Queries

For this tutorial we've set up a GraphQL API for you. The most common way to browse a GraphQL API is to use GraphiQL. GraphiQL is a tool built by Facebook, (pronounced "graphical") that makes it easy to explore any GraphQL API.

When you connect GraphiQL to a GraphQL endpoint, it queries the server for its GraphQL schema and gives you a UI to browse and test queries, and that powers its amazing autocomplete!

![GraphiQL demo](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-react/graphiql.gif)

Tools like GraphiQL make GraphQL APIs really easy to use and integrate APIs in your app without requiring external documentation tools.

You can access the GraphiQL for this realtime todo app tutorial here: [hasura.io/learn/graphql/graphiql](https://hasura.io/learn/graphql/graphiql)

When you work with a GraphQL API in a project you will almost always use a tool like GraphiQL to explore and test your GraphQL queries.

## Run the GraphQL Query

1. Open GraphiQL at: [hasura.io/learn/graphql/graphiql](https://hasura.io/learn/graphql/graphiql). You'll have to login to get an auth token to query the API. In a real-world scenario your GraphQL APIs will be protected.
2. You'll see a URL, and headers that contain the auth token that will be sent along with your GraphQL query.
3. Now, paste this GraphQL query in the GraphiQL window

Copy

 query {

   users {

     name

   }

 }

4. Hit `ctrl + enter` or `cmd + enter` (mac) or click on the ▶️ icon to run the GraphQL query
5. On the right, you should see a list of users by their names that are in the system!

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

Recall that there is no magic here! The hosted GraphiQL app is sending a GraphQL query string to the server at the given endpoint with the HTTP headers. The server then sends the response that you see on the right hand side.

## GraphQL Nested Query

The todo application has:

- users
- todos
- information about users that are currently online

This is what the API "schema" looks like:

![Schema](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-react/schema.png)

The schema is a graph-like schema where all the 3 models are linked to each other. Since all 3 models are linked, we can use nested queries. GraphQL nested queries allow you to fetch relational data in one request.

In the context of the todo application, you can fetch the users and their todos in one request with nested queries.

**How is that possible?**

Nested queries in GraphQL are possible due to the relationships between objects. When you build a GraphQL API, you define relationships in the schema, if there are any.

For example, there is a relationship between `users` and `todos` in the todo application. Each user can have multiple todos, but a todo can only belong to a user. As a result, you can fetch the users and their todos in one request.

### Fetch Users and Their Todos

This GraphQL query will fetch all the users and their publicly visible todos:

Copy

 query {

   users {

     name

     todos {

       title

     }

   }

 }

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

### Fetch Online Users and Their Profile Information

This GraphQL query will fetch all the currently online users and their profile information (which is just their name for now):

Copy

 query {

   online_users {

     last_seen

     user {

       name

     }

   }

 }

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

## Adding Parameters (Arguments) To GraphQL Queries

In most API calls, you usually use parameters. e.g. to specify what data you're fetching. If you're familiar with making `GET` calls, you would have used a query parameter. For example, to fetch only 10 todos you might have made this API call: `GET /api/todos?limit=10`.

The GraphQL query analog of this is _arguments_, which are key-value pairs that you can attach to a "field" or "nested object". GraphQL servers come with a default list of arguments, but you can also define custom arguments.

### GraphQL Query With an Argument: Fetch 10 Todos

This GraphQL query will fetch only 10 todos rather than all of them.

Copy

query {

  todos(limit: 10) {

    id

    title

  }

}

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

The most important bit to check here is `limit: 10`. GraphQL servers will provide a list of arguments that can be used in `()` next to specific fields.

In our case, we are using Hasura for creating the GraphQL backend which provides filter, sort and pagination arguments. The GraphQL server or API that you use, might provide a different set of arguments that can be used.

### GraphQL Query With Multiple Arguments

GraphQL allows you to use multiple arguments in the same query. You can use one or more arguments on each field or nested object in the query.

Let's fetch 1 user and the 5 most recent todos for that user to showcase that.

Copy

query {

  users (limit: 1) {

    id

    name

    todos(order_by: {created_at: desc}, limit: 5) {

      id

      title

    }

  }

}

Notice that we are passing arguments to different fields. The above GraphQL query reads as:

> Fetch users (with limit 1), and their todos (ordered by descending creation time, and limited to 5).

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

## GraphQL Variables: Passing Arguments to Your Queries Dynamically

Until now, you hardcoded the arguments in the queries. In real-life applications, though, the arguments might come from different parts of your application, such as filters for example. So you will pass them dynamically to your queries.

In GraphQL, you can pass arguments dynamically with the help of variables.

## GraphQL Query With Variables

Let's fetch a limited number of todos. That can be done with the `limit` argument as follows:

Copy

query ($limit: Int!) {

  todos(limit: $limit) {

    id

    title

  }

}

If you look at the previous GraphQL queries with arguments, you might spot two differences:

- You define the type of the variable accepted by the query - an integer (number), in this case
- The hardcoded value is replaced by the variable `$limit`

But before you can run the query, there is an additional step. You also need to send a variables object:

Copy

{

   "limit": 10

}

The GraphQL server will automatically use the variable in the right place in the query! That means, `$limit` is replaced by the number "10".

Instead of sending just the query to the GraphQL server from our client, we'll send both the query and the variables.

Let's try this out in GraphiQL:

1. Head to GraphiQL
2. Write out this query
3. Scroll to the bottom of the page, where you see a smaller panel "Query Variables"
4. Add the query variable as a JSON object

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

## GraphQL Limit and Offset

In GraphQL, you can limit the number of rows returned by the query with the `limit` argument. `limit` takes an integer, representing the number of rows to return.

Copy

{

  todos(limit: 5) {

    title

    is_completed

    is_public

  }

}

In this example, you fetch only 5 todos.

But why would you want to do that? One of the most common scenarios is pagination, where you would use the `limit` and `offset` arguments. The `offset` argument specifies how many records to skip.

For example, if we have 50 todos, we could split them into 5 pages of 10 todos. You would fetch the first page as follows:

Copy

{

  todos(limit: 5, offset: 0) {

    title

    is_completed

    is_public

  }

}

You would want to skip the first 5 todos on the second page, so the `offset` would be "5".

Copy

{

  todos(limit: 5, offset: 5) {

    title

    is_completed

    is_public

  }

}

You would continue like that until the last page. So, `limit` specifies the number of rows to return, whereas `offset` specifies where to start counting.

## GraphQL Query Filter - Where Clause

You can filter the data returned by your queries with the `where` argument. The `where` argument is applied on a specific field and it filters the results based on that field's value.

For example, you can use the `where` argument to fetch all the private todos.

The query is as follows:

Copy

{

  todos(where: {is_public: {_eq: false}}) {

    title

    is_public

    is_completed

  }

}

You can also use the `where` argument multiple times in one query. Let's say you want to see all the public notes from a specific user:

Copy

{

  users(where: {id: {_eq: "61dd5e7dc4b05c0069a39att"}}) {

    name

    todos(where: {is_public: {_eq: true}}) {

      title

      is_public

    }

  }

}

The `where` argument uses operators to filter the results accordingly. The above queries use the `_eq` operator, which stands for "equal to". They read:

> Fetch all the todos where the value of the field `is_public` equals "false".

And

> Fetch all the todos where the value of the field `is_public` equals "true" for the user whose id equals to "61dd5e7dc4b05c0069a39att".

There are other operators that you can see in the [API Reference](https://hasura.io/docs/latest/graphql/core/api-reference/graphql-api/query.html#whereexp) documentation.

## Summary

- You can now write simple and nested GraphQL queries
- You know how to pass arguments to your GraphQL queries
- You know how to make your arguments dynamic by using query variables
- You know how to use the `limit` and `offset` arguments
- You know how to filter your queries with the `where` clause

Next, we'll look at writing data and not just fetching data!