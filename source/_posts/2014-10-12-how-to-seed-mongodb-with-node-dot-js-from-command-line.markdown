---
layout: post
title: "How to seed MongoDB with Node.js from command line"
date: 2014-10-12 01:08:03 -0700
comments: true
categories: [mongodb, node.js]
---

I'm new to Node.js and this blog post is to help me remember what I did as I learn it. Hopefully, it will also help others who are reading this. Comments and suggestions are welcome.

I have a Rails background so I'm used to having nice terminal commands like Rail's `rake` command. To create a database and populate it, I would use a terminal command like: `rake db:setup`

For Node.js, I wanted a command like `node seed.js` to

1. Connect to MongoDB
2. Drop a specific DB
3. Recreate the DB
4. Insert documents into DB

<!--more-->

There are a few libraries out there for handling migrations, but I didn't use them here and maybe I'll use them in a future post.

* [mongo-migrate - https://github.com/afloyd/mongo-migrate](https://github.com/afloyd/mongo-migrate)
* [node-migrate - https://github.com/visionmedia/node-migrate](https://github.com/visionmedia/node-migrate)



## Node.js = Asynchronous

Node.js uses asynchronous calls in an event loop with a single thread. So if you were to make an I/O call such as reading a file from your filesystem and saving it to a database, Node.js will execute those lines of code and then move on to next line of code. It will not wait for the code to finish reading that file and saving to the DB. Here's an example: 

1. Read a file and insert it into 1000 User documents into a MongoDB database
2. Print to console when the code finishes inserting those 1000 documents

<script src="https://gist.github.com/jonathanxie/70d52f124b6159402b8b.js"></script>

So there's a problem with the asynchronous nature of Node.js when we want to do certain things sequentially. In the case above, `console.log` would be executed before all the `user.save` calls are done. So how do we fix this? We use a wonderful utility library called:

* `async` - [https://github.com/caolan/async](https://github.com/caolan/async) 

## Async.js = Useful Utility Library

The `async` library provides a number of useful utility functions to help you manage the flow of your asynchronous calls. For the example above, we would use the `async.each` utility function to loop through the 1000 users and then call console.log when it's done. Here's the code:

<script src="https://gist.github.com/jonathanxie/9632aa28b39bccfb147f.js"></script>


Now that you a sense of how `async` works, I can move on to the more complicated example I wanted to demonstrate for seeding a database from command line. So I need to the following in this order:

1. Ensure a database connection to MongoDB has been established 
2. Once the connection has been established, drop the database
3. Create database and add 1000 User documents to the database

Below is a gist of my documented code. 

<script src="https://gist.github.com/jonathanxie/42e87ced561c1b8b0d58.js"></script>

You can download the full code at: 
[https://github.com/jonathanxie/seed-mongodb-with-node.git](https://github.com/jonathanxie/seed-mongodb-with-node.git).

To run the code, do following commands:

1. `npm install`
2. `node seed.js`

Refer to the following blogs for further reading as I found these useful in helping me write this blog post:

* [http://www.sebastianseilund.com/nodejs-async-in-practice](http://www.sebastianseilund.com/nodejs-async-in-practice)
* [http://justinklemm.com/node-js-async-tutorial/](http://justinklemm.com/node-js-async-tutorial/)

