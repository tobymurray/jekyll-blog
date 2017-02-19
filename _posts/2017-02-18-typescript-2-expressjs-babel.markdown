---
layout: post
title:  "ExpressJS with TypeScript 2 and Babel"
date:   2017-02-18 00:00:00 -0500
---
Find the code [here](https://github.com/tobymurray/express-typescript-babel)

# Contents
{:.no_toc}

*  
{:toc}

## Prerequisites

- [nvm](https://github.com/creationix/nvm) - not specifically necessary, but a great way to manage Node
- [Node.js](https://nodejs.org/en/) - the more recent the better
- [Yarn Package Manager](https://yarnpkg.com/) - the more recent the better

## Premise

My goal is to provide a seed project for TypeScript development of a server. The context I'm comfing from is using one of the ever-multiplying front end frameworks (e.g. Angular) to produce the client and wanting a technology to deliver the client and provide APIs for the client to interact with. The intention is to base the seed project off the output of the [Express Generator](https://github.com/expressjs/generator), but with TypeScript instead of vanilla JavaScript.

The [Express Generator](https://github.com/expressjs/generator) provides a nice scaffold, but it doesn't produce many files. It doesn't have [an option for avoiding any view templating languages](https://github.com/expressjs/generator/issues/152), which is unfortunate, but easy enough to work around. If we ignore the directories and the view templating engines, the generator only produces 6 files.

{% highlight bash %}

# Excluding directories and the `.ejs` files that are generated
$ express -v ejs

   create : ./package.json
   create : ./app.js
   create : ./routes/index.js
   create : ./routes/users.js
   create : ./bin/www
   create : ./public/stylesheets/style.css

{% endhighlight %}

Great place to start then!

## Things that are not impacted by TypeScript

### public/stylesheets/style.css

There's nothing to change here. The `style.css` can stay exactly the same as in the generator.

{% highlight css %}
body {
  padding: 50px;
  font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;
}

a {
  color: #00B7FF;
}
{% endhighlight %}

### views/index.ejs --> public/index.html

Avoiding a template engine means we're going to have to serve up something different. A static file will work fine, so we'll swap out `index.ejs` for the equivalent static HTML file and throw it in the public folder - `public/index.html'.

{% highlight html %}
<html>

<head>
  <title>Express</title>
  <link rel="stylesheet" href="/stylesheets/style.css">
</head>

<body>
  <h1>Express</h1>
  <p>Welcome to Express</p>
</body>

</html>
{% endhighlight %} 

### views/error.ejs --> public/error.html

Same goes for the Error view, but unfortunately we lose some functionality here - as the file is static we can't dynamically add the error. Works for now! `public/error.html`:

{% highlight html %}
<html>

<head></head>

<body>
  <h1>Don't know the error message</h1>
  <h2>Don't know the status code</h2>
  <pre>Error: Being static, we can't dynamically generate this content.</pre>
</body>

</html>

{% endhighlight %} 

## Things we need to add

### /models/http_error.ts

This is unfortunate, but in the generated `app.js` there are these two lines when catching `404` errors:

{% highlight javascript %}
var err = new Error('Not Found');
err.status = 404;
{% endhighlight %}

`Error` doesn't have a status, so this will blow up for us as soon as we add some typing information. We can add a wrapper in its place `models/http_error.ts` that we'll use when we look at converting the `app.js` file. We could also get rid of the `status` entirely, as we're not rendering it anymore.

{% highlight typescript %}
export class HttpError extends Error {

  private status: number;

  constructor(message: string, status: number) {
    super(message);

    this.status = status;
  }
}
{% endhighlight %}

## The fun stuff

### package.json

This is where we get into the mess of TypeScript. First of all, we can use everything that the generator produced (except of course we'll get rid of the `ejs` dependency):

{% highlight json %}
{
  "name": "express-typescript",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.16.0",
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.0",
    "express": "~4.14.1",
    "morgan": "~1.7.0",
    "serve-favicon": "~2.3.2"
  }
}
{% endhighlight %} 

Then we add the TypeScript specific things:

#### 1. The big one - TypeScript itself:

{% highlight yarn %}
$ yarn add typescript -D
yarn add v0.20.3
info No lockfile found.
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 52 new dependencies.
... < snip > ...
Done in 2.03s.
{% endhighlight %} 

#### 2. Add the types for Node:

{% highlight yarn %}
$ yarn add @types/node -D
yarn add v0.20.3
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 1 new dependency.
└─ @types/node@7.0.5
Done in 1.39s.
{% endhighlight %}

#### 3. Create our `tsconfig.json` file 

{% highlight bash %}
$ node ./node_modules/typescript/lib/tsc --init
message TS6071: Successfully created a tsconfig.json file.
{% endhighlight %}

which produces

{% highlight json %}
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "es5",
        "noImplicitAny": false,
        "sourceMap": false
    }
}
{% endhighlight %}

And this pushes us off the edge of the cliff.  
- `"module"` can be: `None`, `CommonJS`, `AMD`, `System`, `UMD`, `ES6`, or `ES2015`
- `"target"` can be: `ES3`, `ES5`, `ES6`/`ES2015`, `ES2016`, `ES2017` or `ESNext`

We're already committing to deal with the compilation step of TypeScript to JavaScript, so it doesn't seem like too much of a stretch to also try out Babel. So we're moving from writing JavaScript then running JavaScript to writing TypeScript that will transpile to an ES6 target, then using Babel to transpile it to vanilla ES5 JavaScript that we'll be able to actually run. A process for sure, but hopefully worth it. We'll modify the `tsconfig.json` to reconcile the TypeScript portion of it:

{% highlight json %}
{
    "compilerOptions": {
        "module": "es6",
        "target": "es6",
        "noImplicitAny": false,
        "sourceMap": true
    }
}
{% endhighlight %}

Now we've committed to set up Babel as well...

#### 4. Add all the remaining relevant typing dependencies in one big shot

{% highlight yarn %}

$ yarn add @types/body-parser @types/cookie-parser @types/debug @types/express @types/morgan @types/serve-favicon -D
yarn add v0.20.3
[1/5] Resolving packages...
[2/5] Fetching packages...
warning fsevents@1.0.17: The platform "linux" is incompatible with this module.
info "fsevents@1.0.17" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/5] Linking dependencies...
[4/5] Building fresh packages...
[5/5] Cleaning modules...
success Saved lockfile.
success Saved 10 new dependencies.
... < snip > ...
Done in 5.68s.

{% endhighlight %}

#### 5. Integrate Babel, since we committeed to that...

Taking the [example from the Babel site](https://github.com/babel/example-node-server)

{% highlight yarn %}

$ yarn add babel-cli -D
yarn add v0.20.3
[1/4] Resolving packages...
[2/4] Fetching packages...
warning fsevents@1.0.17: The platform "linux" is incompatible with this module.
info "fsevents@1.0.17" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 115 new dependencies.
... < snip > ...
Done in 37.38s.

$ yarn add babel-preset-es2016 -D
yarn add v0.20.3
[1/4] Resolving packages...
[2/4] Fetching packages...
warning fsevents@1.0.17: The platform "linux" is incompatible with this module.
info "fsevents@1.0.17" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 7 new dependencies.
... < snip > ...
Done in 3.47s.

$ yarn add babel-preset-stage-2 -D
yarn add v0.20.3
[1/4] Resolving packages...
[2/4] Fetching packages...
warning fsevents@1.0.17: The platform "linux" is incompatible with this module.
info "fsevents@1.0.17" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 20 new dependencies.
... < snip > ...
Done in 4.34s.

{% endhighlight %}

#### 6. Add the Babel transpile step:

We can do this with Grunt or Gulp or whatever other dependency, but we can also do it with plain old JavaScript

{% highlight json %}

"scripts": {
  "clean": "rm -r build && rm -r test",
  "tsc": "node ./node_modules/.bin/tsc",
  "babel": "node ./node_modules/.bin/babel build --out-dir test --source-maps",
  "build": "yarn run clean && yarn run tsc && yarn run babel"
}

{% endhighlight %}

### routes/index.js

This stays pretty much the same as it is in the generator. The difference here is that the view engine has been swapped for a static file. Additionally, to make it easier to serve up the static file this makes use of global variable - the path to the root of the server. 

{% highlight typescript %}

import express from 'express';
var index = express.Router();

index.get('/', function (req, res, next) {
  res.sendFile('index.html', {
    root: global["appRoot"] + '/public/'
  });
});

export { index };

{% endhighlight %}

### routes/users.js

Same story here, pretty much the same as it is in the generator. 

{% highlight typescript %}

import express from 'express';
var users = express.Router();

users.get('/', function (req, res, next) {
  res.send('respond with a resource');
});

export { users };

{% endhighlight %}

### app.js

This is a big one in terms of changing the structure. It quite naturally looks like it should be a class in the general vicinity of `App`. This is different than the structure of `app.js`, but it seems different in a positive way. There are a couple other changes here:

- swapped out the view enging for static files, removing the view engine lines
- added in the `HttpError` in place of the `Error` object

{% highlight typescript %}

"use strict";

import express from 'express';
import path from 'path';
import favicon from 'serve-favicon';
import logger from 'morgan';
import cookieParser from 'cookie-parser';
import bodyParser from 'body-parser';

import { HttpError } from './models/http_error'
import { index } from './routes/index';
import { users } from './routes/users';

export default class App {
  public app: express.Application;

  constructor() {
    this.app = express();

    // uncomment after placing your favicon in /public
    //app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
    this.app.use(logger('dev'));
    this.app.use(bodyParser.json());
    this.app.use(bodyParser.urlencoded({ extended: false }));
    this.app.use(cookieParser());
    this.app.use(express.static(path.join(__dirname, 'public')));

    this.app.use('/', index);
    this.app.use('/users', users);

    // catch 404 and forward to error handler
    this.app.use(function (req, res, next) {
      var err = new HttpError('Not Found', 404);
      next(err);
    });

    // error handler
    this.app.use(function (err, req, res, next) {
      // set locals, only providing error in development
      res.locals.message = err.message;
      res.locals.error = req.app.get('env') === 'development' ? err : {};

      // render the error page
      res.status(err.status || 500);
      res.sendFile('error.html', {
        root: (<any>global).appRoot + '/public/'
      });
    });
  }
}

{% endhighlight %}

### /bin/www

{% highlight typescript %}
#!/usr/bin/env node

/**
 * As this is the entrypoint for the application, set a global variable for 
 * the root path of the server. This makes it a little easier to serve static
 * files as their path is relative to the root instead of the file that is 
 * trying to serve them.
 */

var path = require('path');
(<any>global).appRoot = path.join(__dirname, './..');

/**
 * Module dependencies.
 */

import App from '../app';
import debug from 'debug';
import http from 'http';

/**
 * Instantiate the application as it's a class now
 */

const application = new App();

/**
 * Get port from environment and store in Express.
 */

var port = normalizePort(process.env.PORT || '3000');
application.app.set('port', port);

/**
 * Create HTTP server.
 */

var server = http.createServer(application.app);

/**
 * Listen on provided port, on all network interfaces.
 */

server.listen(port);
server.on('error', onError);
server.on('listening', onListening);

function normalizePort(val) {
  var port = parseInt(val, 10);

  if (isNaN(port)) {
    // named pipe
    return val;
  }

  if (port >= 0) {
    // port number
    return port;
  }

  return false;
}

/**
 * Event listener for HTTP server "error" event.
 */

function onError(error) {
  if (error.syscall !== 'listen') {
    throw error;
  }

  var bind = typeof port === 'string'
    ? 'Pipe ' + port
    : 'Port ' + port;

  // handle specific listen errors with friendly messages
  switch (error.code) {
    case 'EACCES':
      console.error(bind + ' requires elevated privileges');
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error(bind + ' is already in use');
      process.exit(1);
      break;
    default:
      throw error;
  }
}

/**
 * Event listener for HTTP server "listening" event.
 */

function onListening() {
  var addr = server.address();
  var bind = typeof addr === 'string'
    ? 'pipe ' + addr
    : 'port ' + addr.port;
  debug('Listening on ' + bind);
}

{% endhighlight %}

Check out another take on the subject [here](http://brianflove.com/2016/11/08/typescript-2-express-node/) - what I read before starting. Also remember you can find the code [here](https://github.com/tobymurray/express-typescript-babel).