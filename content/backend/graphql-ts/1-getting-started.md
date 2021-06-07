---
title: Getting Started
pageTitle: 'Building a GraphQL Server with Node.js, Fastify, Typescript, GraphQL Helix, Envelop & Prisma'
description:
  'Learn how to build a GraphQL server with Fastify, graphql-helix, Node.js, Typescript, Envelop & Prisma'
question: 'What role do the root fields play for a GraphQL API?'
answers:
  [
    'The three root fields are: Query, Mutation and Subscription',
    'Root fields implement the available API operations',
    'Root fields define the available API operations',
    'Root field is another term for resolver'
  ]
correctAnswer: 2
---

In this section, you will set up the project for your GraphQL server and implement your first GraphQL query. At the end,
we'll talk theory for a bit and learn about the [GraphQL schema](https://graphql.org/learn/schema/).

### Creating the project

This tutorial teaches you how to build a GraphQL server from scratch, so the first thing you need to do is create the
directory that'll hold the files for your GraphQL server!

<Instruction>

Open your terminal, navigate to a location of your choice, and run the following commands:

```bash
mkdir hackernews-ts
cd hackernews-ts
npm init -y
```

</Instruction>

This creates a new directory called `hackernews-ts` and initializes it with a `package.json` file. `package.json` is
the configuration file for the Node.js app you're building. It lists all dependencies and other configuration options
(such as _scripts_) needed for the app.

### Configuring your Typescript Project

Typescript acts as a superset for Javascript and brings a lot of goodness along with it including things like providing strong typing, allows us to use the latest JS syntax without bothering about the compatibility, allows for amazing tooling around with the editors providing powerful intellisense features, helps us avoid a lot of bugs by throwing it all during compilation and moreover also helps us enforce standardization and sanity to our codebase.

Since we are going to use Typescript for our project, the first thing we should look at is to configure the initial boilerplate we would need to get setup with TS projects

### Setting up your boilerplate

<Instruction>

In your terminal, first create the `tsconfig.json` file `.gitignore` for excluding unwanted directories from the version control `schema.graphql` for holding the GraphQL schema and README.md for holding some documentation, the `src` directory which is where your typescript code would live:

```bash(path=".../hackernews-ts/")
mkdir src
touch tsconfig.json .gitignore schema.graphql README.md src/index.ts
```

</Instruction>

Now that you have created the folders and files as necessary, the next step is to configure typescript using the tsconfig file. To know more about what each of the configuration does, we advise that you look into the [TSConfig Reference](https://www.typescriptlang.org/tsconfig) which provides more details about the same.

<Instruction>

Open `tsconfig.json` and type the following:

```json(path="../hackernews-ts/tsconfig.json")
{
  "compilerOptions": {
    "target": "ES2019",
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "module": "CommonJS",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noImplicitAny": false,
    "outDir": "dist",
    "rootDir": "src",
    "typeRoots": ["../node_modules/@types", "./@types"]
  },
  "include": ["./src/**/*"],
  "exclude": ["node_modules"]
}
```
</Instruction>

### Installing the packages you would need

The next step you will want to do is to install all the packages/dependencies you would need to work on this project. We will be using yarn `1.22.10` which comes with node `v14.5.0` by default.

<Instruction>

In your terminal, cd to the root of your project and run these commands - these packages will be added to your main dependencies:

```bash(path=".../hackernews-ts/")
yarn add fastify graphql graphql-helix @envelop/core @graphql-tools/schema @prisma/client bcryptjs jsonwebtoken graphql-ws ws
```

Now, run these commands in your terminal to install the devdependencies you will need for this project

```bash(path=".../hackernews-ts/")
yarn add --dev @types/node @types/ws concurrently nodemon prisma rimraf typescript
```

Now, all these packages would get installed in your project for you to work with and you will also see a `yarn.lock` file generated in your root directory. We will see the reason behind installing these packages as we go through this course.

</Instruction>


### Adding scripts to your project

To make things easy for you to work with, we will add some scripts to your `package.json` file which can be invoked with shorthand commands. In the `package.json` file available in the root of the project, add these scripts to the JSON.

<Instruction>

```json(path=".../hackernews-ts/package.json")
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "rimraf dist && concurrently \"tsc -w\" \"nodemon dist/index.js\""
  }
}
```

Now this will allow you transpile your typescript code to javascript and start the server with just 2 commands `yarn start` and `yarn dev`

</Instruction>

### Setting up a gitignore

To make sure that we don't checkin unnecessary files to the version control, we will also add a `.gitignore` file to the root of the project which looks like this:

<Instruction>
```(path=".../hackernews-ts/.gitignore")
.env*
dist
node_modules
.idea
.vscode
*.log
.DS_Store
temp/
```
</Instruction>

### Creating a raw GraphQL server

With the project directory in place, you can go ahead and create the entry point for your GraphQL server. This will be a
file called `index.ts`, located inside a directory called `src`.

To start the app, you can now execute `yarn dev` inside the `hackernews-ts` directory. At the moment, this
won't do anything because `index.ts` is still empty ¯\\\_(ツ )\_/¯

Let's go and start building the GraphQL server and write some code 🙌

<Instruction>

Open `schema.graphql` and type the following:

```graphql(path="../hackernews-ts/schema.graphql")
type Query {
    info: String!
}
```

This is the file where you would be defining the GraphQL Schema for the project

</Instruction>


<Instruction>

Next, open `src/index.ts` and type the following:

```ts(path="../hackernews-ts/src/index.ts")
// 1

import { envelop, useLogger, useSchema, useTiming } from '@envelop/core';
import { makeExecutableSchema } from '@graphql-tools/schema';
import { PrismaClient } from '@prisma/client';
import fastify from 'fastify';
import { GraphQLError } from 'graphql';
import { getGraphQLParameters, processRequest, renderGraphiQL, shouldRenderGraphiQL } from 'graphql-helix';
import * as http from "http";

// 2
const typeDefs = `
  type Query {
    info: String!
  }
`

// 3
const resolvers = {
  Query: {
    info: () => `This is the API of a Hackernews Clone`
  }
}

// 4

const schema = makeExecutableSchema({
  typeDefs: typeDefs,
  resolvers: resolvers,
});

// 5

const getEnveloped = envelop({
  plugins: [
    useSchema(schema), 
    useLogger(), 
    useTiming()
  ],
});

const { execute, parse, validate } = getEnveloped();

// 6

const port = process.env.PORT || 4000;

const httpServer = http.createServer();

const app = fastify(httpServer);

app.listen(port, function(){
  console.log(`GraphQL server is running on port ${port}.`);
});


// 7

app.route({
  method: ['GET', 'POST'],
  url: '/graphql',
  async handler(req:any, res:any) {
    
    const request = {
      body: req.body,
      headers: req.headers,
      method: req.method,
      query: req.query,
    };

  // 8

    if (shouldRenderGraphiQL(request)) {
      res.type('text/html');
      res.send(renderGraphiQL({
        // subscriptionsEndpoint: "ws://localhost:4000/graphql",
      }));
    } else {
      const request = {
        body: req.body,
        headers: req.headers,
        method: req.method,
        query: req.query,
      };
      const { operationName, query, variables } = getGraphQLParameters(request);
      const result = await processRequest({
        operationName,
        query,
        variables,
        request,
        schema,
        parse,
        validate,
        execute,
        contextFactory: () => ({
          ...req,
          prisma,
          // pubsub,
          userId:
            req && req.headers.authorization
              ? getUserId(req)
              : null
        }),
        
      });

      // 9

      if (result.type === 'RESPONSE') {
        result.headers.forEach(({ name, value }) => res.setHeader(name, value));
        res.status(result.status);
        res.send(result.payload);
      } else if (result.type === "MULTIPART_RESPONSE") {
        res.writeHead(200, {
          Connection: "keep-alive",
          "Content-Type": 'multipart/mixed; boundary="-"',
          "Transfer-Encoding": "chunked",
        });
    
        req.on("close", () => {
          result.unsubscribe();
        });
    
        res.write("---");
    
        await result.subscribe((result) => {
          const chunk = Buffer.from(JSON.stringify(result), "utf8");
          const data = [
            "",
            "Content-Type: application/json; charset=utf-8",
            "Content-Length: " + String(chunk.length),
            "",
            chunk,
          ];
    
          if (result.hasNext) {
            data.push("---");
          }
    
          res.write(data.join("\r\n"));
        });
    
        res.write("\r\n-----\r\n");
        res.end();
      } else {
        res.status(422);
        res.send({
          errors: [
            new GraphQLError("Subscriptions should be sent over WebSocket."),
          ],
        });
      }
    }
  },
});

// 10

process.once("SIGINT", () => {
  console.log("Received SIGINT. Shutting down HTTP and Websocket server.");
  httpServer.close();
});
```

</Instruction>

> **Note**: This code block is annotated with a file name. It indicates into which file you need to put the code that's
> shown. The annotation also links to the corresponding file on GitHub to help you figure out _where_ in the file you
> need to put it in case you are not sure about that.

All right, let's understand what's going on here by walking through the numbered comments:

1. First we import all the libraries we need for building the server. 

- `envelop` is used for controlling the GraphQL execution pipeline since it exposes all the GraphQL.js functions as hooks to the user to work with via a plugin system

- `graphql-tools` exposes utility functions to the users to operate on the schema and do things like parsing, schema stitching, creating directives, etc. In this case, we are using it to build an executable schema with type definitions and resolvers.

- `prisma` is used as the ORM to operate against any of the popular databases you may have by defining the schema and building operations on top of it. In this case, we are connecting to SQLite by default

- `fastify` is used to build the server on top of Node.js since the functions exposed by default in Node.js are primitive and difficult to work with.

- `graphql-helix` is used to build the GraphQL server in this case. It offers us the ability to work with GraphQL in a scalable and modular way without introducing any bloat or performance bottleneck to the code since the only dependency it uses internally is GraphQL.js


2. The `typeDefs` constant defines your _GraphQL schema_ (more about this in a bit). Here, it defines a simple `Query`
   type with one _field_ called `info`. This field has the type `String!`. The exclamation mark in the type definition
   means that this field is required and can never be `null`.

3. The `resolvers` object is the actual _implementation_ of the GraphQL schema. Notice how its structure is identical to
   the structure of the type definition inside `typeDefs`: `Query.info`.

4. We use `makeExecutableSchema` function available as part of `graphql-tools` to combine the typeDefs and resolvers we have into an executable GraphQL schema which we can run queries and mutations on top of.

5. The envelop function here introduce plugins to the execution cycle. You can look at the available plugins and their functions [here](https://github.com/dotansimha/envelop/#available-plugins). In our case, we use the `useSchema` plugin to provide the GraphQL schema, `useLogger` plugin to log the GraphQL execution phases, `useTiming` to time/trace the GraphQL execution. Since envelop builds on top of GraphQL.js it exposes its own execute, parse and validate function which we then extract to be used in the further steps

6. Here, we setup the Node.js http server, pass the reference to fastify and listen on port 4000 (or you can pass in a runtime env variable for `PORT` to change the ports if you want).

7. Here, we setup the route for GraphQL execution and GraphiQL as well. We accept both GET and POST requests via HTTP and extract the body, headers, method and query data from the fastify request object

8. We will do subscriptions later, so I have commented that out for now. This is the step where we take the request object from fastify, extract data using `getGraphQLParameters` process it, pass the other contextual information like request headers and authorization info which will discuss about later.

9. This is where we respond back depending on the type of response that is expected by the client (plain response or multipart) with appropriate headers and status codes. In case of a multipart response, we will need to manage the response data in chunks and write it to the stream. For more information regarding this, I would recommend going through [this](https://nodejs.org/api/stream.html). 

10. And when we receive the SIGNINT (which is an interrupt signal sent when the user presses Ctrl+C or tries to terminate the proccess), we log the same and close the http server connection 

Go ahead and test your GraphQL server!

### Testing the GraphQL server

<Instruction>

In the root directory of your project, run the following command:

```bash(path=".../hackernews-ts/")
yarn dev
```

</Instruction>

As indicated by the terminal output, the server is now running on `http://localhost:4000`. To test the API of your
server, open a browser and navigate to that URL.

What you'll then see is a [GraphiQL](https://github.com/graphql/graphiql), a powerful "GraphQL
IDE" that lets you explore the capabilities of your API in an interactive manner.


By clicking the **DOCS**-button on the right, you can open the API documentation. This documentation is auto-generated
based on your schema definition and displays all API operations and data types of your schema.

![open the API documentation](https://raw.githubusercontent.com/graphql/graphiql/main/packages/graphiql/resources/graphiql.jpg)

Let's go ahead and send your very first GraphQL query. Type the following into the editor pane on the left side:

```graphql
query {
  info
}
```

Now send the query to the server by clicking the **Play**-button in the center (or use the keyboard shortcut
**CMD+ENTER** for Mac and **CTRL+ENTER** on Windows and Linux).

Congratulations, you just implemented and successfully tested your first GraphQL query 🎉

Now, remember when we talked about the definition of the `info: String!` field and said the exclamation mark means this
field could never be `null`. Well, since you're implementing the resolver, you are in control of what the value for that
field is, right?

So, what happens if you return `null` instead of the actual informative string in the resolver implementation? Feel free
to try that out!

In `index.ts`, update the the definition of `resolvers` as follows:

```js{3}(path=".../hackernews-ts/src/index.ts")
const resolvers = {
  Query: {
    info: () => null,
  }
}
```

To test the results of this, you need to restart the server: First, stop it using **CTRL+C** on your keyboard, then
restart it by running `node src/index.ts` again.

Now, send the query from before again. This time, it returns an error:
`Error: Cannot return null for non-nullable field Query.info.`

What happens here is that the underlying [`graphql-js`](https://github.com/graphql/graphql-js/) reference implementation
ensures that the return types of your resolvers adhere to the type definitions in your GraphQL schema. Put differently,
it protects you from making stupid mistakes!

This is in fact one of the core benefits of GraphQL in general: it enforces that the API actually behaves in the way
that is promised by the schema definition! This way, everyone who has access to the GraphQL schema can always be 100%
sure about the API operations and data structures that are returned by the API.

### A word on the GraphQL schema

At the core of every GraphQL API, there is a GraphQL schema. So, let's quickly talk about it.

> **Note**: In this tutorial, we'll only scratch the surface of this topic. If you want to go a bit more in-depth and
> learn more about the GraphQL schema as well as its role in a GraphQL API, be sure to check out
> [this](https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e) excellent article.

GraphQL schemas are usually written in the GraphQL
[Schema Definition Language](https://www.prisma.io/blog/graphql-sdl-schema-definition-language-6755bcb9ce51) (SDL). SDL
has a type system that allows you to define data structures (just like other strongly typed programming languages such
as Java, TypeScript, Swift, Go, etc.).

How does that help in defining the API for a GraphQL server, though? Every GraphQL schema has three special _root
types_: `Query`, `Mutation`, and `Subscription`. The root types correspond to the three operation types offered by
GraphQL: queries, mutations, and subscriptions. The fields on these root types are called _root fields_ and define the
available API operations.

As an example, consider the simple GraphQL schema we used above:

```graphql(nocopy)
type Query {
  info: String!
}
```

This schema only has a single root field, called `info`. When sending queries, mutations or subscriptions to a GraphQL
API, these always need to start with a root field! In this case, we only have one root field, so there's really only one
possible query that's accepted by the API.

Let's now consider a slightly more advanced example:

```(nocopy)
type Query {
  users: [User!]!
  user(id: ID!): User
}

type Mutation {
  createUser(name: String!): User!
}

type User {
  id: ID!
  name: String!
}
```

In this case, we have three root fields: `users` and `user` on `Query` as well as `createUser` on `Mutation`. The
additional definition of the `User` type is required because otherwise the schema definition would be incomplete.

What are the API operations that can be derived from this schema definition? Well, we know that each API operation
always needs to start with a root field. However, we haven't learned yet what it looks like when the _type_ of a root
field is itself another [object type](http://graphql.org/learn/schema/#object-types-and-fields). This is the case here,
where the types of the root fields are `[User!]!`, `User` and `User!`. In the `info` example from before, the type of
the root field was a `String`, which is a [scalar type](http://graphql.org/learn/schema/#scalar-types).

When the type of a root field is an object type, you can further expand the query (or mutation/subscription) with fields
of that object type. The expanded part is called _selection set_.

Here are the operations that are accepted by a GraphQL API that implements the above schema:

```graphql(nocopy)
# Query for all users
query {
  users {
    id
    name
  }
}

# Query a single user by their id
query {
  user(id: "user-1") {
    id
    name
  }
}

# Create a new user
mutation {
  createUser(name: "Bob") {
    id
    name
  }
}
```

There are a few things to note:

- In these examples, we always query `id` and `name` of the returned `User` objects. We could potentially omit either of
  them. Note, however, when querying an object type, it is required that you query at least one of its fields in a
  selection set.
- For the fields in the selection set, it doesn't matter whether the type of the root field is _required_ or a _list_.
  In the example schema above, the three root fields all have different
  [type modifiers](http://graphql.org/learn/schema/#lists-and-non-null) (i.e. different combinations of being a list
  and/or required) for the `User` type:
  - For the `users` field, the return type `[User!]!` means it returns a _list_ (which itself cannot be `null`) of
    `User` elements. The list can also not contain elements that are `null`. So, you're always guaranteed to either
    receive an empty list or a list that only contains non-null `User` objects.
  - For the `user(id: ID!)` field, the return type `User` means the returned value could be `null` _or_ a `User` object.
  - For the `createUser(name: String!)` field, the return type `User!` means this operation always returns a `User`
    object.

Phew, enough theory 😅 Let's go and write some more code!
