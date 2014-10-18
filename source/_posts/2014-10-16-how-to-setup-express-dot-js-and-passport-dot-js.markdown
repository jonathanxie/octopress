---
layout: post
title: "How to setup express.js and passport.js"
date: 2014-10-16 21:46:06 -0700
comments: true
categories: 
---

So I'm building a mobile app and I decided to use express.js on the server side. My needs registration and authentication so this post discuses how I set up express.js and passport.js.

<!--more-->

## Initial Setup

1. Install [Node.js](http://nodejs.org/)
2. Create a directory: `mkdir demo`
3. Go to created directory: `cd demo`
4. Create a package.json file: `npm init`
5. Install Express and save the npm package reference to [package.json](https://docs.nodejitsu.com/articles/getting-started/npm/what-is-the-file-package-json): `npm install express --save`

Note: Node modules installed with the --save option are added to the dependencies list in the package.json file. Then using npm install in the app directory will automatically install modules in the dependecies list.




## References

####Node.js
https://docs.nodejitsu.com/articles/getting-started/npm/what-is-the-file-package-json

<br/>

####Express.js
http://expressjs.com/starter/installing.html