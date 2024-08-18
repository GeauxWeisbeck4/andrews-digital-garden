---
id: 01J5HQP472C0NQY617NHWE2R4V
title: GraphQL Mutations
modified: 2024-08-17T23:16:44-04:00
description: How to write data in GraphQL with mutations and what subscriptions are
tags:
  - graphql
  - mutations
  - api
  - web-development
  - subscriptions
---
# GraphQL Mutations - Writing data

Before learning about GraphQL Mutations, you should be familiar with the following concepts:

- [GraphiQL](https://hasura.io/learn/graphql/intro-graphql/graphql-queries/#graphiql)
- [Query Variables](https://hasura.io/learn/graphql/intro-graphql/graphql-queries/#graphqlvariables:passingargumentstoyourqueriesdynamically)

Like GraphQL Queries, Mutations use variables too, so it's important to be familiar with them.

## What is a GraphQL Mutation?

Up to this point, you only learned how to fetch data in GraphQL. But the real-world applications are complex and you also need to insert, update or delete data. The question is - how do you do it?

In GraphQL, you insert, update or delete data with **mutations**. A Mutation is a GraphQL Operation that allows you to insert new data or modify the existing data on the server-side. You can think of GraphQL Mutations as the equivalent of `POST`, `PUT`, `PATCH` and `DELETE` requests in REST.

To understand better, let’s look at a GraphQL Mutation example:

Copy

mutation {

  insert_todos(objects: [{ title: "Learn GraphQL" }]) {

    returning {

      id

      created_at

    }

  }

}

The above mutation inserts a new todo note into the database. If successful, it returns the `id` and the `creation date` of the newly inserted todo.

What can we observe by looking at the above mutation?

1. It uses the keyword `mutation` rather than `query`.
2. The mutation takes an input. We provide the title of the new todo - `Learn GraphQL`.
3. The mutation returns data. It returns the specified fields for the newly inserted todo.

You will see more GraphQL Mutations later and you will also have the chance to try them in the GraphiQL IDE.

## Types of GraphQL Mutations

Mutations in GraphQL can be categorised under the following types:

- Insert Mutations
- Update Mutations
- Delete Mutations

When you run an `update` mutation, you need to specify the record you want to update. Additionally, you need to specify the fields you wish to update and the new data.

When you run a `delete` mutation, you only need to specify the record you want to delete.

In the following section, you can see examples of GraphQL mutations for inserts, updates and deletes.

## GraphQL Mutation Examples

When you use Hasura to build a GraphQL API, you automatically get the mutations for inserts, updates, and deletes.

Note: The mutations you get from another GraphQL service might be different.

### Create a todo

The first mutation is an `insert` mutation that adds a new todo to the database. The mutation that creates todos is called `insert_todos`.

> **Protip**: If you don’t know the mutation name, GraphiQL comes to the rescue! Head over to GraphiQL and click on the "docs" tab (on the right side). Type "todo" there and you’ll see a list of GraphQL queries and types that use todo. Read through their descriptions and you’ll soon find that `insert_todos` is what you need.

In GraphiQL, write the following mutation:

Copy

mutation {

  insert_todos(objects: [{ title: "Learn about GraphQL Mutations" }]) {

    returning {

      id

    }

  }

}

The todo data we want to insert is sent as an argument to the `insert_todos` mutation.

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

### Returning data after the mutation

Since the mutation also returns data, we must specify the fields we want to get back after the mutation finishes. The "fields" of the mutation specify the shape of the _response_ that we want the server to send back.

Let's say we would like to get the entire todo object once it's been created. It can be done as follows:

Copy

mutation {

  insert_todos(objects: [{ title: "New Todo" }]) {

    returning {

      id

      title

      is_completed

      is_public

      created_at

    }

  }

}

Let's analyze the response returned by this mutation!

### Mutation Response

Running the above mutation returns the following response:

Copy

{

  "data": {

    "insert_todos": {

      "returning": [

        {

          "id": 55386,

          "title": "New Todo",

          "is_completed": false,

          "is_public": false,

          "created_at": "2022-02-24T10:26:30.718681+00:00"

        }

      ]

    }

  }

}

The mutation returns all the todo data we requested; nothing more, nothing less.

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

### GraphQL Variables: Pass Arguments to Mutations Dynamically

In real-world applications, you will most likely parameterize the arguments of the mutations. There are not many scenarios where you would hardcode the mutations in the application.

The reason for using variables is that the arguments for mutations come from different parts of the application. For instance, the arguments might come from a drop-down menu after selecting an option. If you would hardcode the mutation, you would have duplicate code in your application. By parameterizing the mutation, you can re-use it with different arguments.

Let's refactor the `insert_todos` mutation to use Query Variables:

Copy

# The parameterised GraphQL mutation

mutation($todo: todos_insert_input!){

  insert_todos(objects: [$todo]) {

    returning {

      id

    }

  }

}

Looking at the above mutation, you can observe that we are not passing the todo title directly anymore. Instead, we'll pass the todo title as a variable.

Copy

# As a query variable

{

  "todo": {

    "title": "A new dynamic todo"

  }

}

Here the `todos_insert_input` is the type of the variable `$todo` and `!` is used to denote that it is a mandatory input.

### Update a todo

We can also use Query Variables when using an "update" mutation. The mutation used to update todos is called `update_todos`.

Copy

mutation ($id: Int, $is_completed: Boolean, $title: String) {

  update_todos(where: {id: {_eq: $id}}, _set: {is_completed: $is_completed, title: $title}) {

    affected_rows

    returning {

      id

      title

      is_completed

    }

  }

}

Instead of hardcoding the _id_, _is_completed_ and _title_ directly into the mutation, we pass them as variables.

This way, you can re-use the mutation and update other todos without modifying it. All you need to do is to update the Query Variables.

Copy

{

  "id": 55414,

  "is_completed": true,

  "title": "Updated - New Todo"

}

Running the mutation will return the following response:

Copy

{

  "data": {

    "update_todos": {

      "affected_rows": 1,

      "returning": [

        {

          "id": 55414,

          "title": "Updated - New Todo",

          "is_completed": true

        }

      ]

    }

  }

}

The response illustrates the updated todo. Of course, you can tweak the mutation to return something else. For example, you can only leave `affected_rows` and remove the rest.

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

### Delete a todo

Let's try to delete a note from the database. You can do that with the help of the `delete_todos` mutation.

Copy

mutation($id: Int) {

  delete_todos(where: {id: {_eq: $id}}) {

    affected_rows

    returning {

      id

      title

    }

  }

}

We pass the id as a variable.

Copy

{

  "id": 55386

}

After the mutation finishes, you will get the following response:

Copy

{

  "data": {

    "delete_todos": {

      "affected_rows": 1,

      "returning": [

        {

          "id": 55386,

          "title": "New Todo"

        }

      ]

    }

  }

}

The mutation returns the number of affected rows and the todo title.

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

## Nested Mutation

In GraphQL, you can make multiple mutations in one request with the help of nested mutations. When two objects have a relationship between them, you can perform nested mutations.

Let's say we want to add a new todo and its author in the same request. If these objects have a relationship, you can insert both objects - the todo and user - in the same request.

Copy

mutation insertTodos {

  insert_todos(objects: [{title: "Nested Mutations", user: {data: {name: "Hasura"}}}]) {

    affected_rows

    returning {

      id

      title

      user {

        id

        name

      }

    }

  }

}

If any of the mutations fails, no data is inserted into the database. Both mutations need to be successful for the data to be added to the database.

Note: The way the todo application is set up does not allow us to run the above mutation. The user exists before a new todo is added, so the above mutation is just for illustrative purposes.

## Bulk Mutation

If you want to add or delete multiple todos in one go, you can do that with bulk mutations. The GraphQL Mutation below illustrates an example of a bulk insert mutation.

Copy

mutation {

  insert_todos(

    objects: [

      { title: "Learn about GraphQL Mutations" }, 

      { title: "Learn about Bulk Mutations" }

    ]

  ) {

    affected_rows

    returning {

      id

      title

      is_completed

    }

  }

}

Instead of adding the todos with two separate operations, you added them simultaneously.

How do you achieve the same thing with Query Variables? You can accomplish the same thing as follows:

Copy

mutation($todos: [todos_insert_input!]!) {

  insert_todos(objects: $todos) {

    affected_rows

    returning {

      id

      title

      is_completed

    }

  }

}

Now you need to pass the todos as an array.

Copy

{

  "todos": [

    { "title": "Bulk Mutation" },

    { "title": "GraphQL Variables" }

  ]

}

The mutation response illustrates the number of rows affected and the newly inserted notes.

Copy

{

  "data": {

    "insert_todos": {

      "affected_rows": 2,

      "returning": [

        {

          "id": 59491,

          "title": "Bulk Mutation",

          "is_completed": false

        },

        {

          "id": 59492,

          "title": "GraphQL Variables",

          "is_completed": false

        }

      ]

    }

  }

}

**[Try it out in GraphiQL](https://hasura.io/learn/graphql/graphiql)**

That's how you can perform bulk mutations in GraphQL!

## GraphQL Mutation Best Practices

1. Start with a verb when you name your mutations. Looking at the todo application, you can see that each mutation starts with a verb - `insert_todos`, `update_todos` and `delete_todos`. The verb describes what the mutation does.

2. Keep your mutations specific. Each mutation should serve only one purpose. For instance, a mutation should not allow you to insert and update a todo note.

3. Mutations should use a single input object as an argument. As you might have observed earlier, the `insert_todos` mutation takes the `todos_insert_input!` input object as an argument.

## Summary

In this section, you learned:

- What a GraphQL Mutation is
- About the available types of mutations
- How to write mutations for inserting, updating and deleting data
- How to pass dynamic arguments/data to mutations with query variables
- How to perform nested and bulk mutations
- About GraphQL Mutation best practices

Next, let’s look at GraphQL Subscriptions

# GraphQL Subscriptions - Realtime updates

As we learned earlier, you can fetch data with [GraphQL Queries](https://hasura.io/learn/graphql/intro-graphql/graphql-queries/), which return the requested information in one read. If the data updates on the server, you have to perform another query to get the updates.

However, there are use cases when you might want real-time updates from the server. In this section, you will learn how to do that with the help of GraphQL Subscriptions!

## What is a GraphQL Subscription?

A Subscription is a GraphQL operation that enables you to subscribe to events on the server. You will get real-time updates from the server each time the event you subscribed to occurs.

An event can represent a record insertion, modification or deletion. Using the todo application as an example, we can use GraphQL Subscriptions to notify us each time a new todo is added to the database.

GraphQL Subscriptions are a critical component of adding real-time or reactive features to your applications. GraphQL clients and servers that support subscriptions allow you to build great experiences without dealing with WebSocket code!

## GraphQL Subscription Example

Let's write the first Subscription that notifies us when a user becomes online.

`Step 1`: Head to [https://hasura.io/learn/graphql/graphiql](https://hasura.io/learn/graphql/graphiql)

`Step 2`: Write this GraphQL Query in the textarea:

Copy

subscription {

  online_users {

    id

    last_seen

    user {

      name

    }

  }

}

`Step 3`: Click on the play button.

Every time the set of online users changes, you’ll see the latest set on the right side of the GraphiQL IDE.

## GraphQL Subscriptions with WebSockets

GraphQL Subscriptions are implemented using the WebSocket protocol, enabling us to create a persistent connection between the server and client. The connection stays open until either party terminates it.

The WebSockets make it possible to implement features such as Subscriptions and get real-time updates from the server.

## Replace Query with Subscription

With Hasura, you can convert a Query to a Subscription to get real-time data. Let's consider the following query:

Copy

query {

  todos {

    id

    created_at

    is_completed

    is_public

    title

  }

}

The above query returns all the todos and their:

- id
- creation date
- completion status (completed or not completed)
- privacy (public or private)
- title

How do you convert the above Query to a Subscription? You can do so by replacing the keyword `query` with `subscription`:

Copy

subscription {

  todos {

    id

    created_at

    is_completed

    is_public

    title

  }

}

Now the connection stays open and you get updates every time a new todo is added to the database.

## GraphQL Subscriptions vs Live Queries

Another way to get real-time data updates from the server is using GraphQL Live Queries. A Live Query is like a regular query except that it contains the `@live` directive.

Here’s an example of a Live Query:

Copy

query @live {

  todos {

    id

    created_at

    is_completed

    is_public

    title

  }

}

The above Live Query watches the query result and whenever it changes, the server returns the new results to the client.

One significant difference is that Subscriptions are defined in the GraphQL Specification, whereas Live Queries are not. That means there is no official definition of a Live Query.

Another difference is that Subscriptions respond to events. For example, you might have a Subscription that reacts to an insertion. When the insertion occurs, the server sends back the new data to the client.

On the other hand, Live Queries watch the latest result of a query and whenever it changes, the server returns the latest results to the client. Rather than responding to an event, they monitor for changes in the query result.

**Note**: Live Queries do not work by default. They only work if the server supports them.

## Summary

By this point, you:

- Learnt what a GraphQL Subscription is
- Wrote your first Subscription
- Converted a Query to a Subscription
- Learnt the difference between GraphQL Subscriptions and Live Queries

Now that you’re comfortable with the basics of using GraphQL let's see how the servers and clients are structured.