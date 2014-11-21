---
layout: post
title: "Setting Up LoopBack to use HTTPS and SSL Certificates"
date: 2014-11-12 17:56:26 -0800
comments: true
categories: [LoopBack, SSL Certificates, HTTPS]
published: true
---

This post is about setting up LoopBack to always use HTTPS and SSL certificates because that it's important to have a secure connection when you have registration and login for an app.

<!--more-->

## Software versions I am using:

1. OS X 10.10 on MacBook Pro (17-inch, Mid 2009)
2. [homebrew](http://www.brew.sh): 0.9.5
3. [mongodb](http://www.mongodb.org): 2.6.5
4. [mysql](http://www.mysql.com): 5.6.21
5. [postgres](http://www.postgres.org): 9.3.5
6. [nvm](https://github.com/creationix/nvm): 0.17.3
7. [Node.js](http://nodejs.org/):	0.10.33
8. [npm](https://www.npmjs.org/): 1.4.28
9. [strongloop](https://github.com/strongloop/strongloop): 2.9.2

## Set up SSL Certificates and HTTPS redirect

<b>1.</b>&nbsp;&nbsp;Make sure you have a loopback application setup. If you don't, then run: `slc loopback` 

``` bash 'slc loopback' output
[?] What's the name of your application? demo-loopback
identical .editorconfig
identical .jshintignore
identical .jshintrc
identical .npmignore
   create server/boot/authentication.js
   create server/boot/explorer.js
   create server/boot/rest-api.js
   create server/boot/root.js
   create server/server.js
   create client/README.md

I'm all done. Running npm install for you to install the required dependencies. 
If this fails, try running the command yourself.


...
...
...


Next steps:

  Create a model in your app
    $ slc loopback:model

  Optional: Enable StrongOps monitoring
    $ slc strongops

  Run the app
    $ slc run .
```


This will set up a simple loopback application. For a more complicated app with models, you can also check out what  I did [here](http://www.jonxie.com//blog/2014/11/03/setting-up-a-loopback-application/) or check out StrongLoop's [tutorial](http://docs.strongloop.com/display/public/LB/Getting+started+with+LoopBack).

Your directory structure should look like:

```bash Initial directory structure for LoopBack 
.
├── client
├── node_modules
└── server
```


<b>2.</b>&nbsp;&nbsp;In your StrongLoop application, go to the `server` folder and create a subfolder called `ssl` and `middleware`

``` bash Create subfolders 'ssl' and ''middleware' in 'server' folder
.
├── client
├── node_modules
└── server
    ├── boot
    ├── middleware
    └── ssl		
```

<b>3</b>&nbsp;&nbsp; Add `ssl-keygen.sh` to the **ssl directory** 

``` bash ssl-keygen.sh
#!/bin/bash
 
# https://github.com/strongloop/loopback-gateway/blob/master/server/private/ssl-keygen.sh
 
KEY_LENGTH=2048
 
# Generate CA
#openssl genrsa -des3 -out ca.key $KEY_LENGTH
#openssl req -new -key ca.key -out ca.csr
#openssl x509 -req -in ca.csr -out ca.crt -signkey ca.key
 
# Generate server key
openssl genrsa -passout pass:1234 -des3 -out server.key $KEY_LENGTH
openssl req -passin pass:1234 -new -key server.key -out server.csr
 
# Remove the passphrase
cp server.key server.key.org
openssl rsa -passin pass:1234 -in server.key.org -out server.key
 
# Generate server certificate
openssl x509 -req -days 730 -in server.csr -signkey server.key -out server.pem
cp server.pem certificate.pem 
cp server.key privatekey.pem
rm server.pem server.key server.key.org server.csr
```

<b>4.</b>&nbsp;&nbsp;Add `ssl-cert.js` to the **ssl directory** 

``` javascript ssl-cert.js
/**
 * https://github.com/strongloop/loopback-gateway/blob/master/server/private/ssl_cert.js
 **/
 
var crypto = require('crypto'),
  fs = require("fs"),
  path = require('path');
 
exports.privateKey = fs.readFileSync(path.join(__dirname, 'privatekey.pem')).toString();
exports.certificate = fs.readFileSync(path.join(__dirname, 'certificate.pem')).toString();
 
exports.credentials = crypto.createCredentials({
	key: exports.privateKey, 
	cert: exports.certificate
});
```

<b>5.</b>&nbsp;&nbsp;Your LoopBack directory structure should now look like

``` bash LoopBack directory structure after adding two files to ./middleware/ssl
.
├── client
├── node_modules
└── server
    ├── boot
    ├── middleware
	  └── ssl  
        ├── ssl-keygen.sh
        └── ssl_cert.js
```

<b>6.</b>&nbsp;&nbsp;In the **ssl directory**, run **ssl-keygen.sh**: `bash ssh-keygen.sh`

<b>7.</b>&nbsp;&nbsp;If you input in the right information in the terminal, your directory should look like:

``` bash LoopBack directory structure after adding running 'ssh-keygen.sh' in './server/ssl'
.
├── client
├── node_modules
└── server
    ├── boot
    ├── middleware
	  └── ssl  
        ├── certificate.pem
        ├── privatekey.pem
        ├── ssl-keygen.sh
        └── ssl_cert.js
```

<b>8.</b>&nbsp;&nbsp;In the **middleware directory**, create a subdirectory called **https-redirect**. Next you need code for redirecting all HTTP calls to HTTPS. So create a file called `index.js` with the following code:

``` javascript index.js
/**
 * https://github.com/strongloop/loopback-gateway/blob/master/server/middleware/https-redirect/index.js
 * 
 * Create a middleware to redirect http requests to https
 * @param {Object} options Options
 * @returns {Function} The express middleware handler
 */
module.exports = function(options) {
  options = options || {};
  var httpsPort = options.httpsPort || 443;
  return function(req, res, next) {
    if (!req.secure) {
      var parts = req.get('host').split(':');
      var host = parts[0] || '127.0.0.1';
      return res.redirect('https://' + host + ':' + httpsPort + req.url);
    }
    next();
  };
};
```


<b>9.</b>&nbsp;&nbsp;Again in the **middleware** directory, create a subdirectory called **explorer** which will contain code for bringing up LoopBack's API explorer. By default, LoopBack has `explorer.js` in the **./server/boot** directory. You will remove or comment out that code later since you are putting the API explorer in as middleware. So create a file called `index.js` with the following code:

``` javascript index.js
/**
 * Middleware for LoopBack's API Explorer
 **/
module.exports = function(server) {
  var explorer;
  try {
    explorer = require('loopback-explorer');
  } catch(err) {
    console.log(
      'Run `npm install loopback-explorer` to enable the LoopBack explorer'
    );
    return;
  }
 
  var restApiRoot = server.get('restApiRoot');
 
  var explorerApp = explorer(server, { basePath: restApiRoot, protocol: 'https' });
  server.use('/explorer', explorerApp);
  server.once('started', function() {
    var baseUrl = server.get('url').replace(/\/$/, '');
    // express 4.x (loopback 2.x) uses `mountpath`
    // express 3.x (loopback 1.x) uses `route`
    var explorerPath = explorerApp.mountpath || explorerApp.route;
    console.log('Browse your REST API at %s%s', baseUrl, explorerPath);
  });
};
```



<b>10.</b>&nbsp;&nbsp;Your directory should now look like:

``` bash Loopback directory structure after adding in the 'http-redirect' and 'explorer' middleware 
.
├── client
├── node_modules
└── server
    ├── boot
    ├── middleware
    │   ├── explorer
    │   │   └── index.js
    │   ├── https-redirect
    │   │   └── index.js		    
	  └── ssl  
```

<b>11.</b>&nbsp;&nbsp;Now go to your **./server/boot** folder and it should look like:

``` bash Loopback directory structure for 'boot' folder
.
├── client
├── node_modules
└── server
    ├── boot
    │   ├── authentication.js
    │   ├── explorer.js
    │   ├── rest-api.js
    │   └── root.js
    ├── middleware
	  └── ssl  

```
 
The boot directory contain boot scripts to perform custom initialization, in addition to that performed by the LoopBack bootstrapper. More information can be found [here](http://docs.strongloop.com/display/public/LB/Defining+boot+scripts).

<b>12.</b>&nbsp;&nbsp;Now either comment out all the code in **explorer.js** and **root.js** *OR* delete both files. I opted to comment them out.

<b>13.</b>&nbsp;&nbsp;Modify **./server/config.json** by adding in the following:

		"https-port": 3001,
		"url": "https://localhost:3001/"

<b>14.</b>&nbsp;&nbsp;Your config **./server/config.json** should now look like:

``` json config.json
{
  "restApiRoot": "/_internal",
  "host": "0.0.0.0",
  "port": 3000,
  "https-port": 3001,
  "url": "https://localhost:3001/"
}
```


<b>15.</b>&nbsp;&nbsp;Next open **./server/server.js** and your file should look like:

``` javascript server.js
var loopback = require('loopback');
var boot = require('loopback-boot');
 
var app = module.exports = loopback();
 
// Set up the /favicon.ico
app.use(loopback.favicon());
 
// request pre-processing middleware
app.use(loopback.compress());
 
// -- Add your pre-processing middleware here --
 
// boot scripts mount components like REST API
boot(app, __dirname);
 
// -- Mount static files here--
// All static middleware should be registered at the end, as all requests
// passing the static middleware are hitting the file system
// Example:
//   var path = require('path');
//   app.use(loopback.static(path.resolve(__dirname, '../client')));
 
// Requests that get this far won't be handled
// by any middleware. Convert them into a 404 error
// that will be handled later down the chain.
app.use(loopback.urlNotFound());
 
// The ultimate error handler.
app.use(loopback.errorHandler());
 
app.start = function() {
  // start the web server
  return app.listen(function() {
    app.emit('started');
    console.log('Web server listening at: %s', app.get('url'));
  });
};
 
// start the server if `$ node server.js`
if (require.main === module) {
  app.start();
}
```

<b>16.</b>&nbsp;&nbsp;Now modify **./server/server.js** to look like:

``` javascript server.js
var loopback = require('loopback');
var boot = require('loopback-boot');

// Load the necessary modules and middleware for HTTPS
var http = require('http');
var https = require('https');
var httpsRedirect = require('./middleware/https-redirect');
var sslCert = require('./ssl/ssl-cert');
var httpsOptions = {
  key: sslCert.privateKey,
  cert: sslCert.certificate
};

// Load middleware for API Explorer
var explorer = require('./middleware/explorer');

// Get an app server instance from LoopBack
var app = module.exports = loopback();

// Set up the /favicon.ico
app.use(loopback.favicon());

// request pre-processing middleware
app.use(loopback.compress());

// -- Add your pre-processing middleware here --

// boot scripts mount components like REST API
boot(app, __dirname);

// Define HTTPS redirect first since order matters when defining middleware
var httpsPort = app.get('https-port');
app.use(httpsRedirect({httpsPort: httpsPort}));

// All middleware and routes defined below and on will be redirected to a HTTPS connection

// Set up API explorer
explorer(app);

// Set up route for `/` to show loopback status for now
app.get('/', loopback.status());

// -- Mount static files here--
// All static middleware should be registered at the end, as all requests
// passing the static middleware are hitting the file system
// Example:
//   var path = require('path');
//   app.use(loopback.static(path.resolve(__dirname, '../client')));

// Requests that get this far won't be handled
// by any middleware. Convert them into a 404 error
// that will be handled later down the chain.
app.use(loopback.urlNotFound());

// The ultimate error handler.
app.use(loopback.errorHandler());

app.start = function() {
  
  // start the web server
  var port = app.get('port');

  http.createServer(app).listen(port, function() {
    console.log('Web server listening at: %s', 'http://localhost:3000/');
    https.createServer(httpsOptions, app).listen(httpsPort, function() {
      app.emit('started');
      console.log('Web server listening at: %s', app.get('url'));
    });
  });

};

// start the server if `$ node server.js`
if (require.main === module) {
  app.start();
}
```

<b>17.</b>&nbsp;&nbsp;Now go to the root directory of your LoopBack application and run: `slc run`

``` bash Output from running 'slc run'
INFO strong-agent not profiling, StrongOps configuration not found.
Generate configuration with:
    npm install -g strongloop
    slc strongops
See http://docs.strongloop.com/strong-agent for more information.
supervisor running without clustering (unsupervised)
Web server listening at: http://localhost:3000/
Browse your REST API at https://localhost:3001/explorer
Web server listening at: https://localhost:3001/
```

<b>18.</b>&nbsp;&nbsp;Now that your server is running, go to 

  * [http://localhost:3000](http://localhost:3000) which should redirect you to [https://localhost:3001]
  * [http://localhost:3000/explorer](http://localhost:3000) which should redirect you to [https://localhost:3001/explorer]


## References

### SSL
* https://github.com/strongloop/loopback-example-ssl

### LoopBack Gateway
* http://strongloop.com/strongblog/node-js-loopback-api-gateway-sample-applications/
* https://github.com/strongloop/loopback-gateway

### LoopBack Boot Scripts
* http://docs.strongloop.com/display/public/LB/Defining+boot+scripts