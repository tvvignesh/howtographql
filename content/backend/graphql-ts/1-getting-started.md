---
title: Getting Started
pageTitle: 'Building a GraphQL Server with Node.js, Fastify, Typescript, GraphQL EZ, GraphQL Helix, Envelop & Prisma'
description:
  'Learn how to build a GraphQL server with Fastify, graphql-helix, Node.js, Typescript, GraphQL EZ, Envelop & Prisma'
question: What does GraphQL EZ use under the hood for enabling plugins?
answers:
  [
    'Prisma',
    'Envelop',
    'Fastify',
    'Postman'
  ]
correctAnswer: 1
---

In this section, we will look at an overview about the project, create the relevant folders and learn about the packages we would be using to build the GraphQL server.

### GraphQL EZ

We use plugins from GraphQL EZ to make our job of configuring the server easy. Think of it like a plugin based boilerplate for GraphQL server.

In this tutorial, we will use:

- [GraphQL EZ Fastify Integration](https://www.graphql-ez.com/docs/integrations/fastify) for building our server on top of Node.js and make sure its performant at the same time as well.
- [GraphiQL Plugin](https://www.graphql-ez.com/plugins/graphiql) for exploring the GraphQL schema using the IDE
- [GraphQL Schema Plugin](https://www.graphql-ez.com/plugins/schema) for working with the GraphQL Schema
- [GraphQL Scalars Plugin](https://www.graphql-ez.com/plugins/graphql-scalars) for leveraging custom scalars like DateTime which are not part of the GraphQL Specification
- [GraphQL Websockets Plugin](https://www.graphql-ez.com/plugins/websockets) for enabling realtime communication and updates using GraphQL Subscriptions



You can find all the other plugins which you can use with GraphQL EZ has [here](https://www.graphql-ez.com/plugins) and if you want to explore envelop plugins, you can find them [here](https://github.com/dotansimha/envelop#available-plugins)

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
touch tsconfig.json .gitignore README.md src/index.ts
```

</Instruction>

This is how the project structure looks like:

![project directory structure](https://imgur.com/NM2BdEV.jpg)

Now, this is how the src directory looks like:

![src directory structure](https://i.imgur.com/Tm5djKV.jpg)

You can have a look at the [Github Repository](https://github.com/tvvignesh/graphql-ez-example/tree/hackernews-ts) for reference on how the completed project looks like.

Now that you have created the folders and files as necessary, the next step is to configure typescript using the tsconfig file. To know more about what each of the configuration does, we advise that you look into the [TSConfig Reference](https://www.typescriptlang.org/tsconfig) which provides more details about the same.

<Instruction>

Open `tsconfig.json` and type the following:

```json(path="../hackernews-ts/tsconfig.json")
{
    "compilerOptions": {
      "target": "es2015",
      "module": "commonjs",
      "noEmit": false,
      "strict": true,
      "moduleResolution": "node",
      "esModuleInterop": true,
      "skipLibCheck": true,
      "forceConsistentCasingInFileNames": true,
      "noImplicitAny": false,
      "outDir": "dist",
      "rootDir": "./src"
    },
    "include": ["src/**/*.ts"],
    "exclude": ["**/node_modules", "**/dist"]
  }
```
</Instruction>

If you want to understand what these options do, you can look at the file linked with comments [here](https://github.com/tvvignesh/graphql-ez-example/blob/hackernews-ts/tsconfig.json)

### Installing the packages you would need

The next step you will want to do is to install all the packages/dependencies you would need to work on this project. We will be using yarn `1.22.10` which comes with node `v16.0.0` by default or you can install it manually as well by doing `npm i -g yarn`

<Instruction>

In your terminal, cd to the root of your project and run these commands - these packages will be added to your main dependencies:

```bash(path=".../hackernews-ts/")
yarn add fastify graphql @graphql-ez/fastify @graphql-ez/plugin-graphiql @graphql-ez/plugin-scalars @graphql-ez/plugin-websockets @graphql-ez/plugin-schema @prisma/client bcryptjs fastify graphql graphql-ez graphql-subscriptions jsonwebtoken
```

Now, run these commands in your terminal to install the devdependencies you will need for this project

```bash(path=".../hackernews-ts/")
yarn add --dev @types/node prisma typescript concurrently nodemon rimraf
```

Now, all these packages would get installed in your project for you to work with and you will also see a `yarn.lock` file generated in your root directory. We will see the reason behind installing these packages as we go through this course.

</Instruction>


### Adding scripts to your project

To make things easy for you to work with, we will add some scripts to your `package.json` file which can be invoked with shorthand commands. In the `package.json` file available in the root of the project, add these scripts to the JSON.

<Instruction>

```json(path=".../hackernews-ts/package.json")
{
  "scripts": {
    "start": "tsc && node ./dist/index.js"
    "dev": "rimraf dist && concurrently \"tsc -w\" \"nodemon dist/index.js\""
  }
}
```

Now this will allow you transpile your typescript code to javascript and start the server with just 2 commands `yarn start` for production without live reload and `yarn dev` during development with live reload using `nodemon`

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