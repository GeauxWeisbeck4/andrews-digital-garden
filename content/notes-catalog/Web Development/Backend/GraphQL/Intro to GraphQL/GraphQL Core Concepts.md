---
id: 01J5HQFWYY7PZ4G46KSSMPCCPV
title: GraphQL Core Concepts
modified: 2024-08-17T23:12:48-04:00
description: Learn the core concepts of GraphQL
tags:
  - graphql
  - api
  - web-development
---
# GraphQL Core Concepts

GraphQL introduces a new set of concepts for someone coming from a REST API background. In this section, we will look at the core concepts of GraphQL from a client/frontend perspective.

## GraphQL document

The content of a GraphQL request string is called the GraphQL document. This is an example of a document:

Copy

{

  author {

    id

    name

  }

}

The string follows a syntax like the above which a GraphQL server or client can parse and validate. The above syntax uses a shorthand notation for a query operation.

## GraphQL operation

A GraphQL operation can be of type

- query (a read-only fetch)
- mutation (a write followed by fetch)
- subscription (a long‐lived request that fetches data in response to source events.)

A GraphQL document can contain one or more of these operations (i.e multiple queries/mutations/subscriptions).

Let's look at an example of a GraphQL document with an operation:

Copy

query {

  author {

    id

    name

  }

}

In this example, a document contains a query operation. GraphQL operation selects the set of information it needs, referred to as the selection set. In the above example, the query operation selects information about the `author` and its `id` and `name`.

We will look into the anatomy of the document in detail now.

## Anatomy of a GraphQL Document

Let's take the following example now:

Copy

query {

  author(limit: 5) {

    id

    name

  }

}

You would have realized that this is another GraphQL document with query operation.

What is the rest of the document composed of? Let's see.

#### Fields

A GraphQL field describes a discrete piece of information. This information could be simple or complex with relationships between data.

In the above document, everything enclosed within the operation is fields. (author, id, and name).

Copy

author {

  id

  name

}

There can be complex information with relationships like the one below:

Copy

query {

  author(limit: 5) {

    id

    name

    articles {

      id

      title

      content

    }

  }

}

Here in addition to author fields, we also have articles fields allowing you to represent relationships between fields.

#### Arguments

Imagine fields as functions that return values. Now let's assume the function also accepts arguments that behave differently.

In the above example,

Copy

author(limit: 5)

The author field accepts an argument `limit` to limit the number of results returned. These arguments can be optional or mandatory and can appear on any field in the document.

#### Variables

GraphQL queries can be parameterized with variables for reuse and easy construction of queries on the client-side.

In the example above, assume the limit parameter is configurable by the user viewing the page, then it would be easier to pass a variable to the field argument.

Copy

query ($limit: Int) {

  author(limit: $limit) {

    id

    name

  }

}

The variable(s) is defined at the top of the operation and the value for the variable can be sent by the client in a format that the server understands. Typically variables are represented in JSON like below

Copy

{

  limit: 5

}

#### Operation Name

When the document contains multiple operations, the server has to know which ones to execute and map the results back in the same order. For example:

Copy

query fetchAuthor {

  author(id: 1) {

    name

    profile_pic

  }

}

query fetchAuthors {

  author(limit: 5, order_by: { name: asc }) {

    id

    name

    profile_pic

  }

}

This has two operations - one for fetching a single author and one for fetching multiple authors.

These are the most common ones typically used in a simple GraphQL request.

Here are some other concepts used by more complex apps.

#### Aliases

Consider the following example:

Copy

query fetchAuthor {

  author(id: 1) {

    name

    profile_pic_large: profile_pic(size: "large")

    profile_pic_small: profile_pic(size: "small")

  }

}

When you are fetching information about an author, let's say you have two images, different sizes and you have a field with an argument to do that. In this case, you cannot use the same field twice under the same selection set and hence an `Alias` would be helpful to distinguish the two fields.

#### Fragments

Fragments make GraphQL even more reusable. If there are some parts of your document that reuses the same set of fields on a given type, then fragment can be powerful.

For example:

Copy

fragment authorFields on author {

  id

  name

  profile_pic

  created_at

}

query fetchAuthor {

  author(id: 1) {

    ...authorFields

  }

}

query fetchAuthors {

  author(limit: 5) {

    ...authorFields

  }

}

Notice the usage of fragment here - `...authorFields`. This type of usage is called a spread fragment. There are also inline fragments where you don't explicitly declare fragment separately but use it inline in a query.

#### Directives

Directives are identifiers which add additional functionality without affecting the value of the response but can affect what response comes back to the client.

The identifier `@` is optionally followed by a list of named arguments.

Some default server directives supported by GraphQL spec are:

- @deprecated(reason: String) - marks the field as deprecated
- @skip (if: Boolean) - Skips GraphQL execution for this field
- @include (if: Boolean) - Calls resolver for an annotated field, if true.

For example:

Copy

query ($showFullname: Boolean!) {

  author {

    id

    name

    fullname @include(if: $showFullname)

  }

}

In the above query, we include the field fullname, only if the condition is true (the condition can have its own logic depending on the app).

You can also use custom directives to handle other use cases.

In your GraphQL adventures, you’re sure to run into these core concepts. Now you’re equipped to work with them!

## GraphQL Comments

With GraphQL, you can add code comments that are also shown in GraphQL tools such as GraphiQL, GraphQL Playground, and others. As a result, the comments are helpful for both the developers working on your GraphQL API and the developers consuming the API.

You can add comments to your code by using quotation marks as follows:

Copy

"""

 Retrieve one or more notes from the database

"""

type Query {

    notes: [Note]

    note(id: Int): Note 

}

The figure below shows an example of GraphQL comments from an existing application.

![GraphQL server comments](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-intro/graphql-server-comments.png)

Using the [public GraphQL API explorer](https://cloud.hasura.io/public/graphiql), we can explore the GraphQL API and see the comments live.

![GraphQL comments showing in the GraphQL Playground](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-intro/note-comment.png)

In the above figure, you can see the comments added for the `Note` type.

![GraphQL comments showing in the GraphQL Playground](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-intro/query-comment.png)

You can also see the comments for the "Query" type. Code comments are a great way to add additional context and help the developers using your GraphQL API.