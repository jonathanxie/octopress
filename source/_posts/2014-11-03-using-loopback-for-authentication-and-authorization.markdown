---
layout: post
title: "Using LoopBack for Authentication and Authorization"
date: 2014-11-03 14:44:30 -0800
comments: true
categories: [LoopBack, StrongLoop, NodeJS]
---

I'm building a mobile app that calls a REST api from a NodeJS server. There are a number of frameworks to use:

1. [Express](www.expressjs.com) and you build your REST API manually
2. [Restify](http://mcavage.me/node-restify/)
3. [hapi](http://hapijs.com/)
4. [LoopBack](http://strongloop.com/node-js/loopback/)

I decided to use LoopBack based on what I read [here](http://strongloop.com/strongblog/compare-express-restify-hapi-loopback/). Setup is quite fast and it has a built in User model with authentication and authorization. This post describes the steps I took to do it. 

<!-- more -->

## Software versions I am using:

1. OS X 10.10 on MacBook Pro (17-inch, Mid 2009)
2. [homebrew](http://www.brew.sh): 0.9.5
3. [mongodb](http://www.mongodb.org): 2.6.5
4. [mysql](http://www.mysql.com): 5.6.21
5. [nvm](https://github.com/creationix/nvm): 0.17.3
6. [Node.js](http://nodejs.org/):	0.10.33
7. [npm](https://www.npmjs.org/): 1.4.28
8. [strongloop](https://github.com/strongloop/strongloop): 2.9.2


## Initial Setup to create a LoopBack REST API project

1. Install [Homebrew](http://www.brew.sh):

    `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

2. Install [mongodb](http://www.mongodb.org): `brew install mongodb`

3. Open a new terminal window and start mongodb: `mongod --config /usr/local/etc/mongod.conf`

4. To exit mongodb, just push `CTRL-C` in the terminal window where you are running mongod.

5. **Optional** - You can set launchd to start mongodb at login: 

    `ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents`

6. **Optional** - If you want to Load mongodb after setting launchd: 

    `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist`

7. Install [mysql](http://www.mysql.com/): `brew install mysql`

8. To start mysql server, run: `mysql.server start`

9. To stop mysql server, run: `mysql.server stop`

10. To log into mysql, make sure mysql server is already running and then run: `mysql -uroot`

11. **Optional** - To have launchd start mysql at login:
  
    `ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents`

12. **Optional** - If you set launchd for mysql, then to load mysql now:

    `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist`

13. Install ossp-uuid for postgres: `brew instal ossp-uuid`

14. Install postgres: `brew install postgres`

15. To start postgres, open a new terminal window and run: `postgres -D /usr/local/var/postgres`

16. To stop postgres, in the terminal window running postgres, push `CTRL-C`

17. **Optional** - To have launchd start postgresql at login:

    `ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents`

18. **Optional** - If you have set launchd for postgres, then to load postgresql now:
    
    `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist`

19. For my own convenience, I've also placed shortcuts in my .zshrc profile for starting and stopping mongodb, mysql, and postgres:

    ```
    alias startmongo="mongod --config /usr/local/etc/mongod.conf"
    alias startmysql="mysql.server start"
    alias stopmysql="mysql.server stop"
    alias startpostgres="postgres -D /usr/local/var/postgres"
    ```

20. Install [nvm](https://github.com/creationix/nvm): `curl https://raw.githubusercontent.com/creationix/nvm/v0.17.3/install.sh | bash`

21. Install [Node.js](http://nodejs.org/) (this will install [npm](https://www.npmjs.org/) also): `nvm install 0.10`

22. Set nvm's default Node.js version to be used in any shell: `nvm alias default 0.10`

23. Install StrongLoop command line client: `npm install -g strongloop`

24. Create and go to a directory for your LoopBack app: `mkdir loopback-demo && cd loopback-demo`

25. Create a loopback app: `slc loopback`

26. Push the `Enter` key since LoopBack already knows you're in the `loopback-demo` directory. Here's what you should see:

    `[?] What's the name of your application? (loopback-demo) `

27. Wait a few minutes and `slc loopback` to finish and you should see the following output:
    
    <script src="https://gist.github.com/jonathanxie/11c9dfb713d59cd2cdaa.js"></script>

28. Your loopback-demo directory should look like:

    ```
    drwxr-xr-x  10 jonxie staff 340 Nov  3 23:44 .
    drwx------+ 22 jonxie staff 748 Nov  4 12:34 ..
    -rw-r--r--   1 jonxie staff 288 Oct  2 02:12 .editorconfig
    -rw-r--r--   1 jonxie staff  24 Oct  2 02:12 .jshintignore
    -rw-r--r--   1 jonxie staff 349 Oct  2 02:12 .jshintrc
    -rw-r--r--   1 jonxie staff 120 Oct  2 02:12 .npmignore
    drwxr-xr-x   3 jonxie staff 102 Nov  3 23:44 client
    drwxr-xr-x  11 jonxie staff 374 Nov  3 23:44 node_modules
    -rw-r--r--   1 jonxie staff 462 Nov  3 23:44 package.json
    drwxr-xr-x   7 jonxie staff 238 Nov  3 23:44 server
    ```


## Create Models

**NOTE:** I followed this [post](http://docs.strongloop.com/display/LB/Create+a+simple+API) by StrongLoop to create a model. 

Now that you have a LoopBack project called `loopback-demo`, it's time to create a model called `Person`.

1. Run this command: `slc loopback:model`

    ```
    [?] Enter the model name: 
    ```

2. Enter `Person` for the model name and then choose db(memory) as your 

    ```
    [?] Enter the model name: Person
    [?] Select the data-source to attach Person to: (Use arrow keys)
      (no data-source) 
    ❯ db (memory) 
    ```
3. Next LoopBack will ask you if you want to expose `Person` model as a REST API. Choose `Yes`

    ```
    [?] Enter the model name: Person
    [?] Select the data-source to attach Person to: db (memory)
    [?] Expose Person via the REST API? (Y/n) Y
    ```

4. Next use **`People`** a custom plural form for the Person model for the REST API url

    ```
    [?] Enter the model name: Person
    [?] Select the data-source to attach Person to: db (memory)
    [?] Expose Person via the REST API? Yes
    [?] Custom plural form (used to build REST URL): People
    ```

5. Next LoopBack will ask you to name some properties for the Person model. Enter `name` for the first property.

    ```
    Let's add some Person properties now.

    Enter an empty property name when done.
    [?] Property name: name
    ```

6. Select `string` as the type for the `name` property and then press the **return** key.

    ```
    Enter an empty property name when done.
    [?] Property name: name
       invoke   loopback:property
    [?] Property type: (Use arrow keys)
    ❯ string 
      number 
      boolean 
      object 
      array 
      date 
      buffer 
    (Move up and down to reveal more choices)
    ```

7. Next make the `name` property a **required** type since every person needs a name by typing `y` and hit the **return** key

    ```
    Enter an empty property name when done.
    [?] Property name: name
       invoke   loopback:property
    [?] Property type: string
    [?] Required? (y/N) y
    ```

8. Now you should see the following output:

    ```
    Enter an empty property name when done.
    [?] Property name: name
       invoke   loopback:property
    [?] Property type: string
    [?] Required? Yes

    Let's add another Person property.
    Enter an empty property name when done.
    [?] Property name: 
    ```

9. Repeat steps 1-8 for the following properties
    * `age` as a required property of type **string**
    * `dob` as a required propetty of type **date**<p/>


10. To exit out of creating a model, just hit the **return** key when you see the following text:

    ```
    Let's add another Person property.
    Enter an empty property name when done.
    [?] Property name: 
    ```

11. Run `ls -la` and you should see a directory structure similar as:

    ```
    drwxr-xr-x  11 jonxie staff 374 Nov  4 15:40 .
    drwx------+ 22 jonxie staff 748 Nov  4 12:34 ..
    -rw-r--r--   1 jonxie staff 288 Oct  2 02:12 .editorconfig
    -rw-r--r--   1 jonxie staff  24 Oct  2 02:12 .jshintignore
    -rw-r--r--   1 jonxie staff 349 Oct  2 02:12 .jshintrc
    -rw-r--r--   1 jonxie staff 120 Oct  2 02:12 .npmignore
    drwxr-xr-x   3 jonxie staff 102 Nov  3 23:44 client
    drwxr-xr-x   3 jonxie staff 102 Nov  4 15:40 common
    drwxr-xr-x  11 jonxie staff 374 Nov  3 23:44 node_modules
    -rw-r--r--   1 jonxie staff 462 Nov  4 15:55 package.json
    drwxr-xr-x   7 jonxie staff 238 Nov  3 23:44 server
    ```

    * **server** - Node application scripts and config files are here
    * **client** - Javascript, HTML, CSS files for client
    * **common** - Files common to both client and server
    * **models** - Subdirectory of `common` folder, contains all model JSON and Javascript files<p/>

    For more information about the project structure and files, click [here](http://docs.strongloop.com/display/LB/Create+a+simple+API#CreateasimpleAPI-Checkouttheprojectstructure).

12. Run the application: `slc run`

13. Go to [http://localhost:3000/explorer](http://localhost:3000/explorer) and follow StrongLoop's instructions [here](http://docs.strongloop.com/display/public/LB/Use+API+Explorer) to test the REST API for `POST` and `GET` calls.


## Connect REST API to a datasource

**NOTE:** I followed Strongloop's Documentation [here](http://docs.strongloop.com/display/public/LB/Connect+an+API+to+a+datasource) to add a datasource

Before you were using StrongLoop's in-memory datasource to save data. Now it's time to add a datasource such as MySQL or MongoDB

1. Run `slc loopback:datasource` to setup a datasource and enter in `accountDB`

    ```
    [?] Enter the data-source name: accountDB
    ```

2. Next you will be asked to choose a connector, choose `MySQL` and then hit the **return** key.

    ```
    [?] Enter the data-source name: accountDB
    [?] Select the connector for accountDB: 
      other 
      In-memory db (supported by StrongLoop) 
      Email (supported by StrongLoop) 
    ❯ MySQL (supported by StrongLoop) 
      PostgreSQL (supported by StrongLoop) 
      Oracle (supported by StrongLoop) 
      Microsoft SQL (supported by StrongLoop) 
    (Move up and down to reveal more choices)
    ```

3. After selecting `MySQL` you should see the following output on your terminal screen:

    ```
    [?] Enter the data-source name: accountDB
    [?] Select the connector for accountDB: MySQL (supported by StrongLoop)
    ```

4. Adding the mysql connector updates `server/datasource.json` to include the `accountDB` datasource you just added. The json file should look like:
    
    <script src="https://gist.github.com/jonathanxie/af76a64e5ae12ed4e02f.js"></script>

5. Create another model called `account`: `slc loopback:model`

    ```
    [?] Enter the model name: account
    ```

6. Now choose `accountDB (mysql)` as the datasource for the `account` model 

    ```
    [?] Enter the model name: account
    [?] Select the data-source to attach account to: 
      (no data-source) 
    ❯ accountDB (mysql) 
      db (memory) 
    ```

7. Say `Yes` to exposing the REST API and type in `accounts` for the custom plural form

    ```
    [?] Expose account via the REST API? Yes
    [?] Custom plural form (used to build REST URL): accounts
    Let's add some account properties now.
    ```

8. Add the following properties which are all **required**

    * `email` with type: **string**
    * `level` with type: **number**
    * `created` with type: **date**
    * `modified` with type: **date**

9. Open `common/models/account.json` and it should look like:

    <script src="https://gist.github.com/jonathanxie/397fbba1bbc90a640fde.js"></script>

10. Install strongloops's `loopback-connector-mysql` and add it to package.json: `npm install loopback-connector-mysql --save`

11. Your `package.json` should look like:

    <script src="https://gist.github.com/jonathanxie/b459d52502a94936d0f1.js"></script>

12. Next you need to configure the datasource to use mysql running on your system. Edit `server/datasources.json` and after the line
    `"connector": "mysql"` add the following:

    ```
    "host": "localhost",
    "port": 3306,
    "database": "loopbackDemo",
    "username": "loopback",
    "password": "loopback"
    ```

    Your datasources.json file should now look like:

    <script src="https://gist.github.com/jonathanxie/a8c10adb1a83a9da940d.js"></script>


13. Now you need to setup your localhost's mysql with a new user called `loopback` with proper privileges. 

    If you haven't done so yet, turn on mysql by running: `mysql.server start`

    ```
    Starting MySQL
    . SUCCESS! 
    ```

14. Once you see that mysql server successful started, then log into mysql as the root user: `mysql --user=root mysql`

15. While in the mysql prompt run the following commands:

    <script src="https://gist.github.com/jonathanxie/f4a4800645001cfa58d1.js"></script>

    The paragraphs below are taken and modified from: [http://dev.mysql.com/doc/refman/5.6/en/adding-users.html](http://dev.mysql.com/doc/refman/5.6/en/adding-users.html) 

    * There are two user accounts with username `loopback` and a password of `loopback`. These are superuser accounts with full privileges. The `'loopback'@'localhost'` account can be used only when connecting from localhost. The `'loopback'@'%'` account uses the `'%'` wildcard for the host part, so it can be used to connect from any host. It is necessary to have both accounts for `loopback` to be able to connect from anywhere as monty. Without the localhost account, the anonymous-user account for localhost that is created by mysql_install_db would take precedence when `loopback` connects from localhost. As a result, `loopback` would be treated as an anonymous user. The reason for this is that the anonymous-user account has a more specific Host column value than the `'loopback'@'%'` account and thus comes earlier in the user table sort order. 

    * The 'admin'@'localhost' account has no password. This account can be used only by admin to connect from the local host. It is granted the RELOAD and PROCESS administrative privileges. These privileges enable the admin user to execute the mysqladmin reload, mysqladmin refresh, and mysqladmin flush-xxx commands, as well as mysqladmin processlist. No privileges are granted for accessing any databases. You could add such privileges later by issuing other GRANT statements.

    * The 'dummy'@'localhost' account has no password. This account can be used only to connect from the local host. No privileges are granted. It is assumed that you will grant specific privileges to the account later.

16. You should see the following output in your mysql prompt:

    ```
    mysql> CREATE USER 'loopback'@'localhost' IDENTIFIED BY 'loopback';
    Query OK, 0 rows affected (0.01 sec)

    mysql> GRANT ALL PRIVILEGES ON *.* TO 'loopback'@'localhost' WITH GRANT OPTION;
    Query OK, 0 rows affected (0.00 sec)

    mysql> CREATE USER 'loopback'@'%' IDENTIFIED BY 'loopback';
    Query OK, 0 rows affected (0.00 sec)

    mysql> GRANT ALL PRIVILEGES ON *.* TO 'loopback'@'%' WITH GRANT OPTION;
    Query OK, 0 rows affected (0.00 sec)

    mysql> CREATE USER 'admin'@'localhost';
    Query OK, 0 rows affected (0.00 sec)

    mysql> GRANT RELOAD,PROCESS ON *.* TO 'admin'@'localhost';
    Query OK, 0 rows affected (0.00 sec)

    mysql> CREATE USER 'dummy'@'localhost';
    Query OK, 0 rows affected (0.00 sec)
    ```

17. While you are still in the mysql prompt, create a database called **loopbackDemo**: `create database loopbackDemo`

18. Now that you have your mysql database setup, it's time to seed your database with LoopBack's Node API. LoopBack's sample file is at: 

    [https://github.com/strongloop/loopback-example-mySimpleAPI/blob/master/server/create-test-data.js](https://github.com/strongloop/loopback-example-mySimpleAPI/blob/master/server/create-test-data.js)

    The code is below:

    <script src="https://gist.github.com/jonathanxie/f4726e1c7a084fbcf718.js"></script>

    Make sure to save this file to the `server` directory.

18. From the root directory of your LoopBack project run: `node server/create-test-data.js`

    If all goes well you should see the following output:

    ```
    Record created: { email: 'foo@bar.com',
      level: 10,
      created: Wed Nov 05 2014 23:41:07 GMT-0800 (PST),
      modified: Wed Nov 05 2014 23:41:07 GMT-0800 (PST),
      id: 1 }
    Record created: { email: 'bar@bar.com',
      level: 20,
      created: Wed Nov 05 2014 23:41:07 GMT-0800 (PST),
      modified: Wed Nov 05 2014 23:41:07 GMT-0800 (PST),
      id: 2 }
    done
    ```

## References

#### LoopBack

* http://strongloop.com/strongblog/compare-express-restify-hapi-loopback/
* http://docs.strongloop.com/display/SL/StrongLoop
* http://docs.strongloop.com/display/LB/Installing+StrongLoop
* http://docs.strongloop.com/display/LB/Create+a+simple+API

#### MySQL

* http://dev.mysql.com/doc/refman/5.6/en/adding-users.html