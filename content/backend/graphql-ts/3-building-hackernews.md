---
title: Building Hackernews
pageTitle: 'Build Hackernews application on top of GraphQL EZ with Typescript and Node.js'
description:
  'Learn how to setup the Hackernews with GraphQL EZ'
---

Now that we are set with the basics, let us now build the hackernews application with GraphQL EZ by extending the same project

### Identifying the scope

We aim to build our hackernews app with basic features like:

- Ability to Signup/Login as an end-user
- Ability to Post links
- Ability to Vote on the links
- Subscribe to links which have been posted
- Browse through the feedb of posts

And we will do all of this by using Prisma and SQLite to help us with the database operations.

### Adding the relevant plugins

To satisfy our use case, we would be needing access to plugins which can help us allow things like realtime subscriptions. So, that's the only plugin we would be adding to our existing app.

We can add support for subscriptions by doing `yarn add graphql-subscriptions @graphql-ez/plugin-websockets`

While `graphql-subscriptions` is not currently under maintenance, we are using it to enable basic `pubsub` in our application (aka publishing events and subscribing to events)

We will also be using `prisma` as the ORM to interact with an SQLITE database. You can read more about prisma [here](https://prisma.io/). You will need to initialize the prisma project with a `schema.prisma` file to help us with the same. This is currently out of scope of this tutorial



### Declaring the modules

To help us with this, let us start by declaring the relevant modules in the `src/modules` directory. These would be:

- `src/modules/feed.ts` to manage the Hackernews Feeds
- `src/modules/post.ts` to manage the Posts made
- `src/modules/user.ts` to help us with User Management, Login/Signup
- `src/modules/vote.ts` to help us vote up/down on posts


And as usual we will have `src/modules/index.ts` which will import all these modules.

This is how the modules will look like:

`feed.ts` - We declare queries to get the list of posts, links, count, etc.

```ts(path="../hackernews-ts/src/modules/feed.ts")
import { gql } from '@graphql-ez/plugin-schema';

export const typeDefs = gql`
  type Feed {
    id: ID!
    links: [Link!]!
    count: Int!
  }

  enum Sort {
    asc
    desc
  }

  input LinkOrderByInput {
    description: Sort
    url: Sort
    createdAt: Sort
  }

  extend type Query {
    feed(
      filter: String
      skip: Int
      take: Int
      orderBy: LinkOrderByInput
    ): Feed!
  }`;

export const resolvers = {
    Query: {
      async feed (parent, args, context, info) {
        const where = args.filter
          ? {
              OR: [
                { description: { contains: args.filter } },
                { url: { contains: args.filter } }
              ]
            }
          : {};
      
        const links = await context.prisma.link.findMany({
          where,
          skip: args.skip,
          take: args.take,
          orderBy: args.orderBy
        });
      
        const count = await context.prisma.link.count({ where });
      
        return {
          id: 'main-feed',
          links,
          count
        };
      },
    },
};
```

`src/modules/post.ts` - We will manage new posts in this module and also handle subscriptions for posts that have been made. Note that we publish using `context.pubsub.publish` every time a post is made which is inturn subscribed to using `context.pubsub.asyncIterator`

```ts(path="../hackernews-ts/src/modules/post.ts")
const sleep = (ms: number) => new Promise<void>(resolve => setTimeout(resolve, ms));

import { gql } from '@graphql-ez/plugin-schema';

export const typeDefs = gql`
    type Link {
      id: ID!
      description: String!
      url: String!
      postedBy: User
      createdAt: DateTime!
    }

    type Subscription {
      newLink: Link
      hello: String!
    }

    extend type Mutation {
      post(url: String!, description: String!): Link!
    }
`;

export const resolvers = {
  Mutation: {
    async post(parent, args, context, info) {
      const { userId } = context;
    
      const newLink = await context.prisma.link.create({
        data: {
          url: args.url,
          description: args.description,
          postedBy: { connect: { id: userId } }
        }
      });

      console.log('Publishing new link:::', newLink);

      await context.pubsub.publish('NEW_LINK', {
        newLink: newLink
      });
    
      return newLink;
    }
    
  },
  Subscription: {
    hello: {
      async *subscribe(_root, _args, _ctx) {
        for (let i = 1; i <= 5; ++i) {
          await sleep(500);

          yield {
            hello: 'Hello World ' + i,
          };
        }
        yield {
          hello: 'Done!',
        };
      },
    },
    newLink: {
      async subscribe(_root, _args, context) {
        // console.log('Val::', await context.pubsub.asyncIterator("NEW_LINK").next());
        const newVal = await (await context.pubsub.asyncIterator("NEW_LINK").next()).value;
        console.log('newVal::', newVal);
        return newVal;
      },
    }
    // newLink: {
    //   subscribe: (parent, args, context, info) => {
    //     // console.log('iterator::', context.pubsub.asyncIterator("NEW_LINK"));
    //     return context.pubsub.asyncIterator("NEW_LINK");
    //   }
    // }
  }
};
```

`src/modules/user.ts` - We will manage new user signups and logins in this file. We are using `jsonwebtoken` to help us with the authentication and `bcrypt` to help us hash and validate passwords to be stored in the database. You can read more about jsonwebtoken [here](https://www.npmjs.com/package/jsonwebtoken) and bcrypt [here](https://www.npmjs.com/package/bcrypt)

```ts(path="../hackernews-ts/src/modules/user.ts")
import { gql } from '@graphql-ez/plugin-schema';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { APP_SECRET } from '../utils.js';

export const typeDefs = gql`
    type Query {
      info: String!
    }

    type Mutation {
      signup(
        email: String!
        password: String!
        name: String!
      ): AuthPayload
      login(email: String!, password: String!): AuthPayload
    }

    type AuthPayload {
      token: String
      user: User
    }

    type User {
      id: ID!
      name: String!
      email: String!
      links: [Link!]!
    }
`;

export const resolvers = {
  Query: {
    info(_root, _args, _ctx) {
      return 'hello world';
    }
  },
  Mutation: {
    async signup(parent, args, context, info) {
      const password = await bcrypt.hash(args.password, 10);
      const user = await context.prisma.user.create({
        data: { ...args, password }
      });
    
      const token = jwt.sign({ userId: user.id }, APP_SECRET);
    
      return {
        token,
        user
      };
    },
    async login(parent, args, context, info) {
      const user = await context.prisma.user.findUnique({
        where: { email: args.email }
      });

      if (!user) {
        throw new Error('No such user found');
      }
    
      const valid = await bcrypt.compare(
        args.password,
        user.password
      );
      if (!valid) {
        throw new Error('Invalid password');
      }
    
      const token = jwt.sign({ userId: user.id }, APP_SECRET);
    
      return {
        token,
        user
      };
    }
  },
  User: {
    async links(parent, args, context) {
      return context.prisma.user
        .findUnique({ where: { id: parent.id } })
        .links();
    }
  }
};
```

`src/modules/vote.ts` - Here we will manage upvotes and downvotes of the posts that have been made and also subscribe to the votes similar to what we did for posts

```ts(path="../hackernews-ts/src/modules/vote.ts")
function newVoteSubscribe(parent, args, context, info) {
  return context.pubsub.asyncIterator("NEW_VOTE")
}

import { gql } from '@graphql-ez/plugin-schema';

export const typeDefs = gql`
    type Vote {
      id: ID!
      link: Link!
      user: User!
    }

    extend type Subscription {
      newVote: Vote
    }

    extend type Link {
      votes: [Vote!]!
    }

    extend type Mutation {
      vote(linkId: ID!): Vote
    }
`;

export const resolvers = {
  Mutation: {
    async vote(parent, args, context, info) {
      const { userId } = context;
      const vote = await context.prisma.vote.findUnique({
        where: {
          linkId_userId: {
            linkId: Number(args.linkId),
            userId: userId
          }
        }
      });
    
      if (Boolean(vote)) {
        throw new Error(
          `Already voted for link: ${args.linkId}`
        );
      }
    
      const newVote = context.prisma.vote.create({
        data: {
          user: { connect: { id: userId } },
          link: { connect: { id: Number(args.linkId) } }
        }
      });
      context.pubsub.publish('NEW_VOTE', newVote);
    
      return newVote;
    }
  },
  Subscription: {
    newVote: {
      subscribe: newVoteSubscribe,
      resolve: payload => {
        return payload
      },
    }
  },
  Vote: {
    async link(parent, args, context) {
      return context.prisma.vote
        .findUnique({ where: { id: parent.id } })
        .link();
    },
    async user(parent, args, context) {
      return context.prisma.vote
        .findUnique({ where: { id: parent.id } })
        .user();
    }
  }
};
```

Now that we have the schema and resolvers ready for the different portions of the application, the next step to getting our app to work is to build the context. Let's have a look at it in the next step.