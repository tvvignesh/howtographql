---
title: Running the GraphQL Server
pageTitle: 'Running and testing the GraphQL Server'
description:
  'Let us test what we have built and see how it works'
---

There you go. You have reached the end of the tutorial where we have completed setting up a working Hackernews clone using the latest stack. But let's test it out.

**NOTE:** Remember that you need the right node version to use this project. You can see the nodejs version you need in the `.nvmrc` file in the root of the project (`v16.6.0` at the time of writing) without which you may get errors

As you may already know, you can use the command `yarn dev` from the root of the project to start the GraphQL Server, go to http://localhost:3000/graphiql and you can test if the server is working by running a test query like this:

![Test Server](https://imgur.com/JVP7BYS.jpg)

And you can run a query like this to query the feed for any posts you have made. Ideally, this should return empty for you, but it shows data in the screenshot for me cause I have already made some posts in the app.

![Query Feed](https://imgur.com/y8kDApC.jpg)

Now, let's see how to signup/login as a user, and get the token to be used for subsequent operations because as you may remember, only authenticated users can make posts to the app.

Signing up is simple. All you have to do is, pass the relevant details like your name, email and password to a mutation like you see below and when you execute the mutation, the user should be created for you to use.

![Signup User](https://imgur.com/tksSetN.jpg)

You may use the same token as generated during signup or login to get the token at a later point of time to get the token like this:

![Login User](https://imgur.com/oCy0pIp.jpg)

And if you don't use the token to do certain operations, you may end up getting errors like this:

![User error](https://imgur.com/tJI0F0n.jpg)

The valid way of creating the post would be to add the token which we got to the request headers like this when making the request and you should see your post created

![Auth headers](https://imgur.com/7sQxAo4.jpg)

You can use the same approach to vote for the posts we made as well (Note that we are using the same Link ID we retrievd in the previous step as the parameter when voting)

![Voting for Posts](https://imgur.com/BYHcO5j.jpg)

And if you try voting again using the same user on the same post, you will get an error like this since only one vote is allowed per person on a post as per our logic

![Voting for Posts](https://imgur.com/fM8mzPo.jpg)

So, there you have it. We have a sample working Hackernews application and once you are done with the Backend, you can even start putting together your own user interface to make the GraphQL calls as necessary using the GraphQL Client of your choice (be it Apollo, Relay, URQL, etc.) and represent it in the UI the way you would want to.

We hope you found this useful. If you would like to know more or have any questions, you can always use join us on [Discord](https://discord.com/invite/xud7bH9) or use any of [these links](https://graphql.org/community/) to get in touch with the community.

We are eager to see what you build 😃 If you found this tutorial useful, do share it with your peers and the rest of the community and we hope it can help many more people as well. And if you think there is some room for improvement, let us know and we would love to work on it. Happy Coding 😉