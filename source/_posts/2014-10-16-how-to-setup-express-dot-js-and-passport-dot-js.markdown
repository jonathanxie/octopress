---
layout: post
title: "How to setup ExpressJS/Passport with a REST API"
date: 2014-10-16 21:46:06 -0700
comments: true
categories: 
published: true
---

I'm building a mobile app using (Express)[http://www.expressjs.com] on the server side. I needed registration

<!--more-->


## Software versions I am using:

1. [homebrew](http://www.brew.sh): 0.9.5
2. [mongodb](http://www.mongodb.org): 2.6.5
3. [nvm](https://github.com/creationix/nvm): 0.17.3
4. [Node.js](http://nodejs.org/):	0.10.33
5. [npm](https://www.npmjs.org/): 1.4.28
6. [express-generator](https://www.npmjs.org/package/express-generator): 4.9.0
7. [passport](http://passportjs.org/): 0.2.1
8. [mongoose](http://mongoosejs.com/): 3.8.18


## Initial Setup

1. Install [Homebrew](http://www.brew.sh): 

	`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

2. Install [mongodb](http://www.mongodb.org): `brew install mongodb`

3. Set launchd start mongodb at login: `ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents`

4. Load mongodb now: `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist`

5. Install [nvm](https://github.com/creationix/nvm): `curl https://raw.githubusercontent.com/creationix/nvm/v0.17.3/install.sh | bash`

6. Install [Node.js](http://nodejs.org/) (this will install [npm](https://www.npmjs.org/) also): `nvm install 0.10`

7. Set nvm's default Node.js version to be used in any shell: `nvm alias default 0.10`

8. Install [express-generator](https://www.npmjs.org/package/express-generator) for your terminal: `npm install express-generator -g`

9. Create a directory: `mkdir demo`

10. Go to created directory: `cd demo`

11. Use express generator: `express`

12. You should see the following output in you terminal:

    <script src="https://gist.github.com/jonathanxie/dfebe2aefe4cb20477bd.js"></script>

13. You should see a package.json file now that looks like:

    <script src="https://gist.github.com/jonathanxie/5844aa5970bf9a136935.js"></script>

14. At the time of this writing, Express is at 4.10.1, so open `package.json` and modify Express' version on line 12 to: `4.10.1`

    <script src="https://gist.github.com/jonathanxie/226f14891533768f4ea0.js"></script>

14. Install [passport](http://passportjs.org/) and save it to package.json: `npm install passport --save`

15. Install [mongoose](http://mongoosejs.com/): `npm install mongoose --save`

16. Your updated package.json should now look like:

    <script src="https://gist.github.com/jonathanxie/a8bfdaeca98edb6ff12e.js"></script>


## Setting up Passport


## References

####Node.js
* https://docs.nodejitsu.com/articles/getting-started/npm/what-is-the-file-package-json

####Express.js
* http://expressjs.com/starter/generator.html
* https://www.digitalocean.com/community/tutorials/how-to-install-express-a-node-js-framework-and-set-up-socket-io-on-a-vps
* http://scottksmith.com/blog/2014/05/02/building-restful-apis-with-node/
* http://scottksmith.com/blog/2014/05/05/beer-locker-building-a-restful-api-with-node-crud/

#### Passport
* http://scottksmith.com/blog/2014/05/29/beer-locker-building-a-restful-api-with-node-passport/
