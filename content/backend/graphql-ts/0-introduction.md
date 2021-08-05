---
title: Introduction
pageTitle: 'Building a GraphQL Server with graphql-ez using Node.js, Fastify, Typescript, GraphQL Helix, Envelop & Prisma'
description:
  'Learn how to build a GraphQL server with graphql-ez, Fastify, graphql-helix, Node.js, Typescript, Envelop & Prisma'
question: What is GraphiQL?
answers:
  [
    'A GraphQL IDE to work with a GraphQL API',
    'A tool to generate GraphQL operations',
    'A REST client',
    'The successor of Postman'
  ]
correctAnswer: 0
---

### Overview

**This tutorial is heavily inspired from the [graphql-node tutorial by Robin MacPherson](https://www.howtographql.com/graphql-js/0-introduction/). We have modified the same tutorial to work with Fastify, Typescript, GraphQL Helix, Envelop & Prisma.**

The GraphQL specification was open sourced in 2015 by Facebook along with some basic implementations with a completely unique approach on how to structure, consume, transmit and process data and data graphs.

Today, the GraphQL spec and its implementations have been donated by Facebook to the GraphQL Foundation with open license for development and governance from the community and it has been great so far. And today, the GraphQL foundation comprises not just of companies like Facebook but other organizational members as well.

GraphQL does not make REST or any other channel of communication obsolete. It all boils down to your usecase. For small projects, the simplicity of REST might overweigh the advantages provided by GraphQL but as you have more teams, an evolving product, complex lifecycles and a data schema which gets bigger and bigger by the day, that’s when you will truly realize the value that GraphQL has to offer.

In this tutorial, we'll learn how to build a GraphQL server entirely from scratch. This is the stack we will be working with:

- [Typescript](https://www.typescriptlang.org/): Typed JavaScript at Any Scale.
- [GraphQL EZ](https://www.graphql-ez.com/): Build GraphQL servers with ease by leveraging plugins based on Envelop
- [`Fastify`](https://github.com/fastify/fastify/) An efficient server for Node.js
- [GraphQL Helix](https://github.com/contrawork/graphql-helix/): A highly evolved GraphQL HTTP Server with zero dependencies except for graphql-js itself
- [`graphql-js`](https://github.com/graphql/graphql-js) The JavaScript reference implementation for GraphQL
- [`Envelop`](https://github.com/dotansimha/envelop) Envelop is a lightweight JavaScript library for wrapping GraphQL execution layer and flow, allowing developers to develop, share and collaborate on GraphQL-related plugins, while filling the missing pieces in GraphQL implementations
- [Prisma](https://www.prisma.io/): Replaces traditional ORMs. Use Prisma Client to access your database inside of
  GraphQL resolvers.
- [GraphiQL](https://github.com/graphql/graphiql): A "GraphQL IDE" that allows you to interactively
  explore the functionality of a GraphQL API by sending queries and mutations to it. It's somewhat similar to
  [Postman](https://www.getpostman.com/) which offers comparable functionality for REST APIs.

### Pre-requisites

We assume that you have a basic knowledge of Node.js, Typescript and GraphQL. If you don't you can learn about these here:
- [Node.js](https://nodejs.dev/learn)
- [Typescript](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html)
- [GraphQL Basics](https://graphql.org/learn/)

### What to expect

The goal of this tutorial is to build an API for a [Hacker News](https://news.ycombinator.com/) clone. Here is a quick
rundown of what to expect.

We'll start by learning the basics of how a GraphQL server works, simply by defining a
[_GraphQL schema_](https://graphql.org/learn/schema/) for the server and writing
corresponding _resolver functions_.

Nobody wants a server that's not able to store and persist data, right? Not to worry! Next, we're going to add a
[SQLite](http://sqlite.org/) database to the project which will be managed with [Prisma](https://www.prisma.io/) as the ORM.

Once you have the database connected, you are going to add more advanced features to the API.

We'll start by implementing signup/login functionality that enables users to authenticate against the API. This will
also allow you to check the permissions of your users for certain API operations.

Next, we'll allow the consumers of the API to constrain the list of items they retrieve from the API by adding
filtering and pagination capabalities to it.

Let's get started 🚀
