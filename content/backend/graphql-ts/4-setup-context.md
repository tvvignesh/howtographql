---
title: Setup Context
pageTitle: 'Build the context for the GraphQL Server'
description:
  'Learn how to build and pass the context to the GraphQL Server'
---

Now that we have the schema and resolvers ready for the different portions of the application, the next step to getting our app to work is to build the context by getting the right User ID by extracting it from the [JWT token](https://jwt.io/) passed via authorization headers. We will be using the [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) package for the same.

To this effect, let us create a `utils.ts` file within src and have all the related code there.

```ts(path="../hackernews-ts/src/utils.ts")
import jwt from 'jsonwebtoken';
const APP_SECRET = 'GraphQL-is-aw3some';

function getTokenPayload(token) {
  return jwt.verify(token, APP_SECRET);
}

function getUserId(req, authToken?:string) {
  if (req) {
    const authHeader = req.headers.authorization;
    if (authHeader) {
      const token = authHeader.replace('Bearer ', '');
      if (!token) {
        throw new Error('No token found');
      }
      const { userId } = getTokenPayload(token);
      return userId;
    }
  } else if (authToken) {
    const { userId } = getTokenPayload(authToken);
    return userId;
  }

  throw new Error('Not authenticated');
}

export { APP_SECRET, getUserId };

```

Here, we create a function to extract the authorization headers, decode it and get the userID from the JWT and also verify its authenticity using the signature using the secret which we have signed off with. Do remember that this secret is meant to be kept confidential and not to be shared with anyone outside. You can also sign your tokens with certificates but that's outside the scope of this tutorial.

Now that we have the necessary code in `utils.ts`, let us include it while building the context for the GraphQL server.

If you remember, we had a function `buildContext` in the hello world GraphQL server we built in step 2. We will modify the same function to add the context as required by the GraphQL Server.

This is how our file would look like after the modifications. If you see here, we pass `prisma`, `pubsub`, user context extracted from the authorization headers using `getUserId` function and the `req` object to our context.

Now, this can be accessed in any of our resolver functions thus making it easy for us to work with things like a pooled/shared database connection (setup by prisma), a shared websocket connection which we can use to publish to and subscribe for events and also verify if the relevant user is authenticated or not by verifying the auth headers. We have added comments inline to the code just in case you find something difficult to understand.

```ts(path="../hackernews-ts/src/app.ts")
import { BuildContextArgs, CreateApp, InferContext } from '@graphql-ez/fastify';
import { ezGraphiQLIDE } from '@graphql-ez/plugin-graphiql';
import { ezScalars } from '@graphql-ez/plugin-scalars';
import { ezSchema } from '@graphql-ez/plugin-schema';
import { ezWebSockets } from '@graphql-ez/plugin-websockets';
import { PrismaClient } from '@prisma/client';
import { PubSub } from 'graphql-subscriptions';
import { schema } from './modules';
import { getUserId } from './utils.js';

// Initialize a pubsub instance to emit events to be used for GraphQL Subscriptions

export const pubsub = new PubSub();

// Initialize the prisma client object - Prisma will be used as the ORM to access the underlying SQLITE database

const prisma = new PrismaClient({
  errorFormat: 'minimal'
});

// Context Factory to build the context object in GraphQL Server

function buildContext({ req }: BuildContextArgs) {
    return {
      req,
      prisma,
      pubsub,
      userId:
        req && req.headers.authorization
          ? getUserId(req)
          : null
    };
}

// Leverage Typescript augmentation
declare module 'graphql-ez' {
  interface EZContext extends InferContext<typeof buildContext> {}
}

// Create GraphQL APP by bootstrapping it with the plugins we need (GraphiQL, GraphQL Scalars, GraphQL WS)

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