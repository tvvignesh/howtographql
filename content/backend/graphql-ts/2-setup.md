---
title: Setup
pageTitle: 'Setup the GraphQL server with GraphQL EZ'
description:
  'Learn how to setup the GraphQL Server with GraphQL EZ'
---

Before we work on the hackernews app, lets create a simple GraphQL server which gets us started with a simple schema and resolver. We will later evolve this into the Hackernews application by making the necessary changes to the app.

### Setting up the server

The first step of the process is to get your GraphQL server setup. `index.ts` is your entry point for the project where we will start the server and register the app

```ts(path="../hackernews-ts/src/index.ts")
import Fastify from 'fastify';
import { ezApp } from './app';

// Create a fastify instance in the server

const app = Fastify({
    logger: true,
});

// Register the app and the relevant modules within - Our modules directory has an index.ts file which inturn loads all the relevant modules we need for the app

const { fastifyPlugin } = ezApp.buildApp({});

app.register(fastifyPlugin);

// Listen for the ready event and exit if error occured

app.ready(err => {
    if (!err) return;
  
    console.error(err);
    process.exit(1);
});

// Start listening to the GraphQL Server in the port

app.listen(process.env.PORT || 3000);
```

This is what we did above:

- Register an instance of fastify which is what we are using for building the Node.js / GraphQL server with the logger turned on
- Register the app by registering the plugin which make up the app
- Listen for error events and exit the server after logging on failures
- Start listening to request on port 3000 by default


Now, the next step is to setup `src/app.ts` which takes care of creating our app and registering the GraphQL EZ plugins we need to work on the project

```ts(path="../hackernews-ts/src/app.ts")
import { BuildContextArgs, CreateApp, InferContext } from '@graphql-ez/fastify';
import { ezGraphiQLIDE } from '@graphql-ez/plugin-graphiql';
import { ezSchema } from '@graphql-ez/plugin-schema';
import { ezScalars } from '@graphql-ez/plugin-scalars';
import { schema } from './modules';

// Context Factory to build the context object in GraphQL Server

function buildContext({ req }: BuildContextArgs) {
    return {
      req,
      foo: 'bar',
    };
}

// Leverage Typescript augmentation

declare module 'graphql-ez' {
  interface EZContext extends InferContext<typeof buildContext> {}
}

// Create GraphQL APP by bootstrapping it with the plugins we need

export const ezApp = CreateApp({
    buildContext,
    ez: {
      plugins: [
        ezSchema({
          schema: schema
        }),
        ezGraphiQLIDE(),
        ezScalars({
            DateTime: 1,
        }),
        ezWebSockets('adaptive')
      ],
    }
});


```

And this is what we did above:

- We build a context factory object. The context factory object would be accessible in all the resolvers throughout our GraphQL Server. It is typically used for things like initializing user context, add database connections, etc. which would be inturn used in all resolvers.
- We declare the `graphql-ez` module to help us with Typescript Augmentation
- We create the app by bootstrapping it with all the plugins we need for our app to work. In this case, we are using GraphiQL, GraphQL Schema Plugin and GraphQL Scalars to get us started. These plugins were already installed by us during our first step in the project


Let us quickly look at what the plugins do.

- `@graphql-ez/plugin-graphiql` helps us to initialize the GraphiQL explorer and allows us to browse through and operate on the GraphQL schema, run operations, browse through history of operations, etc. You can read more about GraphiQL [here](https://github.com/graphql/graphiql)
- `@graphql-ez/plugin-scalars` helps us to declare and use custom GraphQL scalars which are not part of the GraphQL Spec by default. For instance, the GraphQL spec does not offer us a way to declare `DateTime` fields. So, using a package like this enables us to extend GraphQL to add the types we need. You can read more about GraphQL Scalars [here](graphql-scalars.dev)
- `@graphql-ez/plugin-schema` helps us build the schema for our GraphQL server. It is an abstraction on top of the various packages offered by [GraphQL Tools](https://graphql-tools.com/) and also comes with typings as required for the resolvers we build.


We can add more plugins in the future to help us with realtime subscriptions, file uploads, etc. We will look at one of the cases as we go through the tutorial.



Now that we are setup with the basics, the next step is declare the modules we are going to use in the project and add our schema and logic onto it.


To do this, we create a `modules` folder within `src` and add all our modules there as separate typescript files

In our case, it is `src/modules/first.ts` , `src/modules/second.ts` and we will also have a file `src/modules/index.ts` which will load all the modules we have in the project.

So, let's start with the first file `src/modules/first.ts`

```ts(path="../hackernews-ts/src/modules/first.ts")
import { gql } from '@graphql-ez/plugin-schema';

export const typeDefs = gql`
  type Query {
      hello: String!
    }`;

export const resolvers = {
    Query: {
        hello(_root, _args, _ctx) {
            return 'hello';
        },
    }
};
```

The above code is simple as it should be. All we are doing is that we register a GraphQL module and to it, we pass the GraphQL Schema and the resolvers which is where we would have all the logic to get the values for the relevant types.

In this case, we define a GraphQL Query `hello` which returns a String type which cannot be null and to return the value for the field, we also have a resolver nested within Query which returns the string which we have hardcoded.

If you see the resolvers, we do see 3 parameters - These are parent, arguments and context information of the GraphQL operation that we are currently running.

While we are not using it in the current use case, the same can be used to pass data/arguments hierarchially to children, which can then be used in the resolvers to get us the relevant data we need.

Once we do all of this, we have the first module of the application.


Now, similarly, we work on another file `src/modules/second.ts` which will act as the second module in the application which works the same way as above but has a different schema and different resolver to resolve the values.

```ts(path="../hackernews-ts/src/modules/second.ts")
import { gql } from '@graphql-ez/plugin-schema';

export const typeDefs = gql`
  extend type Query {
      hello2: String!
    }`;

export const resolvers = {
    Query: {
        hello2() {
            return 'asd';
        },
    },
};
```

Once we get this done, the next step is to actually import this in `src/modules/index.ts` like this so that we can easily import it when registering the modules in our application


```ts(path="../hackernews-ts/src/modules/index.ts")
import { EZSchema } from '@graphql-ez/plugin-schema';
import * as first from './first';
import * as second from './second';

export const schema: EZSchema = {
    typeDefs: [first.typeDefs, second.typeDefs],
    resolvers: [first.resolvers, second.resolvers]
};
```

And that's it. We have our first GraphQL Server built using GraphQL EZ, Envelop, Fastify, GraphiQL, Prisma which can further be extended to add all the features we need.

Now, you can run it by using the command `yarn dev` in the CLI from the root of the project and when you do, your GraphQL Server would start by default in `localhost:3000` port and you can visit the GraphiQL explorer by going to http://localhost:3000/graphiql

Once you are in, you can try running the query which we setup as seen in the screenshot (and click the play button or `Ctrl/Cmd+Enter`) and you should see the result as you see in the screenshot. You can also explore the Docs using the Docs panel in the right and play around with the various autocomplete options you get when you press `Ctrl/Cmd+Space`

![GraphiQL Playground](https://imgur.com/i4HWeux.jpg)