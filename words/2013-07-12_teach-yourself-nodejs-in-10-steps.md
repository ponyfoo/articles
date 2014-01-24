# Teach Yourself Node.JS in 10 Steps

I'm not sure anyone needs convincing that **Node.JS is freaking awesome**, little has been said otherwise. Many of the people reading this blog are _already experienced Node developers_, but maybe we could help newcomers ramp up as fast as the technology itself is ramping up.

> If you are on the verge of [trying something new](/2012/12/29/single-page-design-madness "Single Page Design Madness"), though, then this article has high hopes of you. I can _personally guarantee_ you won't regret giving **Node.JS** a test drive.

I'll try to cover as much as possible, but this is meant to be just _an overview_. You have to **explore by yourself**; I'll give you the tools to do just that, forking off of what I've learned _over the last year_. JavaScript [might be awesome](/2013/02/15/javascript-is-awesome "JavaScript is Awesome"), but **Node is even more so**. That rhymes.

To kick things off, here are _a couple of infographics_. Because who doesn't **love infographics**, right? I generally just embed pictures here, but these are way too large, so I'll just link to them.

The [first infographic](http://i.imgur.com/KCkIkcY.jpg "Infographic - Getting to know Node") I want to show you is about the earlier days of Node. It depicts its strengths, such as its growing community and _the reasons it appeals to developers_. A second, [more recent infographic](http://i.imgur.com/qrhh5Xk.jpg "Infographic - Node Growth Trends"), reveals the steady rise in popularity [Node.JS](http://nodejs.org/ "Node.JS by Joyent") is trending towards.

Those are pretty. You get it, _Node is popular, and awesome_. What is it? Where does it come from?

## History Lesson

Node.JS was released _four years ago_, in 2009. Just a year after Google open-sourced [V8](https://code.google.com/p/v8/ "Google V8 Engine"), the JavaScript engine that powers _Chrome and Node.JS_ alike. You might want to learn more about V8 if the deep technical internals are your thing, you can watch this video in that case. It is _entirely optional_ to the purpose of this article, though, so you keep the link around for later.

[![V8 Talk][1]](http://www.youtube.com/watch?v=FrufJFBSoQY "V8 Talk - High Performance JavaScript Engine - Google I/O")

> Here's something _a bit more in the wheelhouse_ of what we're going to be talking about, an **introduction to Node.JS with Ryan Dahl**, the _huge nerd_ who invented Node. This is a _very entertaining, fast-paced talk_, where you'll learn the basics, _spoken by the original author_ himself. What's not to like? Go ahead, **watch the full thing!**

> [![Intro to Node.JS][2]](http://www.youtube.com/watch?v=jo_B4LTHi3I "Introduction to Node.JS with Ryan Dahl")

Good stuff. Please note the talk is **two years old**, and _some of the statements Ryan makes are now outdated_.

Lets start talking about modularity, to grasp the differences between Node and code in the browser. I prepared a special something you can use as you read this article. You can find every example in this article [on GitHub](https://github.com/ponyfoo/learn-nodejs "Learn NodeJS on GitHub") nicely packed for you to _start playing right away_.

## Modularity in Node

Node implements [CommonJS Modules/1.1](http://wiki.commonjs.org/wiki/Modules/1.1 "Modules/1.1 - CommonJS Spec"), which allow you to keep files self-contained. You can learn all about [Node Modules](http://nodejs.org/api/modules.html "Modules API") from their _increasingly useful documentation_.

Modules can expose an API through the `module.exports` convention.

```js
// modules/math.js

var api = {
    sum: function(a, b){
        return a + b;
    }
};

module.exports = api;
```

Note the variable `api` won't make it's way to the `global` object. You can [learn why](http://nodejs.org/api/globals.html "Global Objects in Node.JS") from the docs. Globals work differently in Node. The top-level scope of a module is local to that module, but you can still access a few globals on your own, such as `process`, and `console`. Setting up your own globals on the `global` object is discouraged.

Consequently, `module` isn't a global, but rather a _local variable_, private to the module we are currently working on.

Modules can be referenced using the `require` function. You can provide a package name _(more on that later)_, or a _physical path_ relative to the file you are invoking `require` from.

```js
// modules/app.js

var math = require('./math.js');
var result = math.sum(1, 2);

console.log('I can\'t believe the result is:', result, '!');
```

Both files being on the same directory is assumed. The `math` variable will now equal the value of `module.exports` in `math.js`.

> What does `console.log` do? It's actually **just syntactic sugar** for `process.stdout.write`, and it will append a new line `\n` at the end. This is **deliberately done all over Node** to _help ease your on-boarding_ onto the platform by leveraging the conventions and objects you are already used to from your experience in writing client-side JavaScript.

> Sidebar. You might want to read the actual [console API](http://nodejs.org/api/stdio.html "console API documentation") documentation.

Note that requiring a file multiple times in the same process will only execute the code in the module once. So if you were to `require` the `app.js` module several times, it would still be executed a single time. As a result, the output would only be buffered once.

```js
// modules/several.js

require('./app.js');
require('./app.js');
require('./app.js');
```

Those are modules all right, but where's the asynchronicity Node is supposedly so popular for?

## Asynchronous Convention

Node is an **event-based language**, and most of the code written for Node follows a really simple convention that helps modules look inspiringly similar to each other, as far as coding conventions go.

Our math module would probably look more like this if we wanted to play nice with the Node community at large.

```js
// async/math.js

var api = {
    sum: function(a, b, done){
        process.nextTick(function(){
            done(null, a + b);
        });
    }
};

module.exports = api;
```

`process.nextTick` is kind of hard to wrap our head around at first, but lets just imagine it's `setTimeout(fn,0)`, which we might have used while trying hacky fixes in the browser.

I've used `process.nextTick` to turn an otherwise synchronous function into an asynchronous one. When we are done processing, we're going to pass the result as _the second parameter_ of the `done` callback. The first parameter should **always** be `err`, if an error occurs, we are passing that as the first parameter, rather than throwing an exception. If no error occurs, we are fine passing any falsy value.

Consuming this module is still really easy.

```js
// async/app.js

var math = require('./math.js');

math.sum(1, 2, function(err, result){
    if(err){
        throw err;
    }
    
    console.log('I can\'t believe the result is:', result, '!');
});
```

We are now _waiting reactively_ for the `sum` function to let us know when it's done. This is the _basest of asynchronous_ examples in **Node.JS**. Note how we _changed modes_, and use `throw` here; this is fine as long as we are in a synchronous path, `throw`ing errors should always have the end result of **process termination**, so keep that in mind when you are dealing with this type of situations. This is _acceptable for our console application_, however in a web application we probably would prefer to just return an HTTP status code 500, _internal server error_, for the current request.

You'll also have to consider the option of **bubbling errors** through _multiple asynchronous calls_. This, for example, might not be the best error-handling approach:

```js
// async/wrong.js

var math = require('./math.js');

math.sum(1, 2, function(err, result){
    if(err){
        throw err;
    }

    math.sum(result, 3, function(err, result){
        if(err){
            throw err;
        }

        console.log('I can\'t believe the result is:', result, '!');
    });
});
```

A more sensible approach might be to avoid throwing errors all over the place, but handle those in a centralized location.

```js
// async/better.js

var math = require('./math.js');

math.sum(1, 2, function(err, result){
    if(err){
        return then(err);
    }

    math.sum(result, 3, function(err, result){
        if(err){
            return then(err);
        }

        then(err, result);
    });
});

function then(err, result){
    if(err){
        throw err;
    }

    console.log('I can\'t believe the result is:', result, '!');
}
```

This is however, getting pretty verbose. Let me skip to a module for a bit, and then come back and explain what's going on. We're going to use the control flow module called [async](https://github.com/caolan/async "caolan/async on GitHub"), to improve the readability of our code.

```js
// async/right.js

var async = require('async');
var math = require('./math.js');

async.waterfall([
    function(next){
        math.sum(1, 2, next);
    },
    function(result, next){
        math.sum(result, 3, next);
    }
], then);

function then(err, result){
    if(err){
        throw err;
    }

    console.log('I can\'t believe the result is:', result, '!');
}
```

That's a little better. Since we've been following the right conventions, we can use `async`, which allows us to get rid of all those pesky `if(err)` statements, and **flatten our callback hell** while we're at it. `waterfall`'s API is pretty simple, we give it an array of functions, and these will be called _in series_, when our first `math.sum` completes, it will invoke the `next` callback with the `(null, 3)` arguments. If a function returns a _truthy value_ in the first parameter, this will **shortcut the waterfall** and immediatly jumping to the `then` function, passing the error argument, still in the first position. If no error occurs, then the next function in the sequence is executed, passing any resulting arguments to it (in this case, just the `3`).

This is the recommended way of doing things because it flattens the structure of our code, turning our codebase into something more readable, while at the same time following the same conventions and using the same API that is used everywhere else. You must check out the async module and its [comprehensive API](https://github.com/caolan/async#documentation "async documentation on GitHub"), toy with it for a while.

> That's great and all, but where did `async` come from? It sure as hell isn't part of Node.

I'm glad you asked.

## Node Packaged Modules

`npm` is _a small treasure_ that comes bundled with Node, and helps you _manage dependencies_ in your projects. There is a [huge repository](http://npmjs.org/ "npm repository") you can search, and most people include installation instructions in their GitHub repositories. Ultimately, `npm` is a CLI _(command-line interface)_ tool.

If you have been following the instructions of the [learn-nodejs](https://github.com/ponyfoo/learn-nodejs "learn-nodejs on GitHub") repository I provided, then you already have dependencies installed in your project folder. If not, just run the following command in your terminal.

```bash
$ npm install
```

That's it, now you have everything you need. How does that work? Some weird magic? No, just [package.json](https://github.com/ponyfoo/learn-nodejs/blob/master/package.json "package.json for learn-nodejs"). This file helps us define the dependencies in our project. When you ran `npm install` in your terminal, all it did was install the dependencies listed in the `package.json` file.

```json
{
  "name": "learn-nodejs",
  "description": "Simple NodeJS Application Examples",
  "homepage": "https://github.com/ponyfoo/learn-nodejs",
  "author": {
    "name": "Nicolas Bevacqua",
    "email": "nicolasbevacqua@gmail.com",
    "url": "http://www.ponyfoo.com"
  },
  "version": "0.0.1",
  "repository": {
    "type": "git",
    "url": "https://github.com/ponyfoo/learn-nodejs.gitt"
  },
  "dependencies": {
    "async": "~0.2.9"
  }
}
```

I rarely add dependencies manually to this definition file, in the case of `async`, for example, all I did was run the following command:

```bash
$ npm install async --save
```

That's it. `async` has been added it to the `dependencies` object. Installing a module basically just fetches it, and adds it to a `node_modules` folder, which you should always exclude in your `.gitignore` settings.

If you are interested in developing your own `npm` module, you'll be shocked to learn [how simple that is](/2013/01/23/publishing-nodejs-packages-with-npm "Publishing Node.JS Packages with npm").

Before we jump into building a decent application, lets look at one of the most powerful constructs in Node.

## Events API

Yes! Of course, I was talking about the [event emitter API](http://nodejs.org/api/events.html "Events in Node"). What are events? Well, the documentation explains it like this:

> Many objects in Node emit events: a `net.Server` emits an event each time a peer connects to it, a `fs.readStream` emits an event when the file is opened. All objects which emit events are instances of `events.EventEmitter`. You can access this module by doing: `require('events');`

> Functions can then be attached to objects, to be executed when an event is emitted. These functions are called listeners. Inside a listener function, `this` refers to the `EventEmitter` that the listener was attached to.

Lets write **our own event emitter** and explain a few things along the way. Then, we'll see how it can be used.

```js
// events/implementation.js

var util = require('util');
var EventEmitter = require('events').EventEmitter;

function Heartbeat(interval){
    EventEmitter.call(this);

    var emitter = this;
    var beats = 0;

    setInterval(function(){
        emitter.emit('beat', ++beats);
    }, interval);
}

util.inherits(Heartbeat, EventEmitter);

module.exports = Heartbeat;
```

Don't laugh, that's the best I could come up with. Here I'm simply creating a constructor function for my custom `EventEmitter` implementation. I'm using [util.inherits](http://nodejs.org/docs/latest/api/util.html#util_util_inherits_constructor_superconstructor "Documentation on util.inherits"), as it's the recommended way of performing **prototypal inheritance** in Node applications.

Whenever our emitter `.emit`s an event, all subscribers to that event will be notified, and receive the arguments which where provided when the event was emitted.

Remember what I mentioned about _leveraging your API knowledge_ about the browser with that in Node? `setInterval` is one of those cases.

Fine, how do we use our newly born event emitter? It's simple, really:

```js
// events/usage.js

var Heartbeat = require('./implementation.js');
var a = new Heartbeat(400);
var b = new Heartbeat(1000);

a.on('beat', function(beats){
    console.log('Heart A beat n times:', beats);
});

b.on('beat', function(beats){
    console.log('Heart B beat n times:', beats);
});
```

I'm not even sure I need to explain this, but whenever the emitter invokes `.emit`, every listener for that event, added by `.on`, will have its callback triggered. This _seemingly innocent API_ powers a lot of what Node does.

One last thing, read this quote from the documentation:

> When an `EventEmitter` instance experiences an error, the typical action is to emit an `'error'` event. Error events are treated as a special case in node. If there is no listener for it, then the default action is to print a stack trace and exit the program.

What this means is that if there is no `.on('error', fn)` listener, and your emitter emits an `'error'` event, then your application will die a tragic death.

## HTTP Server

Enough blabbering, here is an **HTTP server in Node**.

```js
// http/server.js

var http = require('http');

http.createServer(function(req, res) {
    res.end('Hello Node', 200);
    console.log('I think I\'ve heard something!');
}).listen(8000);

console.log('Listening!');
```

That wasn't so amusing, it was very _simple and self-describing_, though! Lets try something different, **serving an HTML file from disk**.

```js
// http/html.js

var http = require('http');
var fs = require('fs');
var path = require('path');
var index = path.resolve(__dirname, './index.html');

http.createServer(function(req, res) {
    var stream = fs.createReadStream(index);

    stream.on('open', function(){
        res.writeHead(200, { 'Content-Type': 'text/html' });
    });

    stream.on('error', function(){
        res.writeHead(404);
        res.end();
    });

    stream.pipe(res);
}).listen(8000);

console.log('Listening!');
```

Couple of things. First of all, `__dirname` is a _special local variable_ that contains the absolute path to the directory for our currently executing module. We just learned what events are, the [fs.createReadStream](http://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options "File System API - Node Documentation") method will provide us with an event emitter we can use to stream data to the response. The file will be piped straight into a [chunked](http://en.wikipedia.org/wiki/Chunked_transfer_encoding "HTTP Chunked Transfer Encoding") response, this can be achieved using the [readable.pipe method](http://nodejs.org/api/stream.html#stream_readable_pipe_destination_options "Node Readable Streams"). If the file isn't found, the stream will emit an `'error'` event; we can take advantage of that and respond with a _404 status code_ instead.

This is, however, a very convoluted thing to do to just serve a file. Enter [Express.JS](http://expressjs.com/ "Express Web Application Framework").

## Express Application Framework

Express is built on [Connect](http://www.senchalabs.org/connect/ "Connect "), which expands on Node's HTTP server. There's also [Socket.IO](http://socket.io/ "Socket.IO realtime application framework") for implementing web socket communications, but I won't be getting into realtime for now.

Connect just provides _middleware_, a nice abstraction over what the native HTTP module offers. Express builds on that, adding a lot of awesome features, and **making your life more bearable**. Here is a small sample application built on Express:

```js
// http/express.js

var express = require('express');
var app = express();

app.get('/', function(req, res){
    res.send('hello world');
});

app.listen(8000);
```

The [API](http://expressjs.com/api.html "Express.JS API Documentation") is **incredibly self-documenting**, I wish more projects had an API as clean as Express does.

Enough already. You are mean for laughing at _all of my stupid examples_. You know what else is mean?

## MongoDB, ExpressJS, AngularJS and NodeJS

The [MEAN Stack](http://blog.mongodb.org/post/49262866911/the-mean-stack-mongodb-expressjs-angularjs-and "The MEAN Stack: MongoDB, ExpressJS, AngularJS and NodeJS") is not a hipster thing, as delusional people try to assertain with no real reasoning behind their empty statements. The MEAN stack is a very real thing. Here's a [slideshare](http://www.slideshare.net/mongodb/mongodb2-21677032 "The MEAN Stack Explained - Slideshare") for you to look at.

- [MongoDB](http://www.mongodb.org/ "MongoDB NoSQL Database Server") as a database
- [ExpressJS](http://expressjs.com/ "Express Web Application Framework") as your web framework
- [AngularJS](http://angularjs.org/ "AngularJS MV* Framework") as your client-side framework
- [NodeJS](http://nodejs.org/ "Node.JS Platform") as the platform

The glaring benefit of using a stack such as this is the ease with which you can transfer objects through your application without having to resort to different interfaces, data presentation alternatives, and programming languages. You can really get away with just using JavaScript everywhere.

> I have fun thinking of _JavaScript detractors melting in hell, as the rising tide that is JS continues to plow through and outclass everything in its wake_. People can't get away with hating JavaScript like they did a few years ago anymore. **Embrace it or fall behind.** That's simply all the truth there is.

> You might not like _cross-browser issues_, but those are [pretty much gone](/2013/07/09/getting-over-jquery "Getting Over jQuery"), and you don't have to face any of that in the fancy world of **Node.JS**.

Fine, enough ranting, there are a couple more things for you to look at.

## Jade and Stylus

Writing plain old HTML is boring, and the same goes for CSS. We have been using templates for a while, but these two really shine when paired with Node.

[Jade](http://jade-lang.com/ "Jade Template Engine") is an HTML templating language which is pretty popular in the Node community (there are other options, such as [EJS](https://github.com/visionmedia/ejs "Embedded JavaScript Templates")). With Jade, you can worry less about syntax and more about content. It also supports partials, inheritance, and everything you'd expect from a server-side templating language.

Except it also supports JavaScript, you can drop plain JavaScript in your Jade templates, and it will interpret that as well. You most definitely should take a look at Jade if you haven't yet.

[Stylus](http://learnboost.github.io/stylus/ "Stylus Expressive CSS") allows you to pretty much pick your own adventure, and figure out on your own how you want to be styling the style in which you write your CSS style sheets.

![styling-styles.jpg][3]

Alternatively, though, you could just keep on using [LESS](http://lesscss.org/ "LESS CSS Pre-processor"). Stylus just seems more flexible. I'd be interested to hear opinions from people who actually used it.

## Web Hosting

There are quite a few hosting alternatives, and picking one mostly depends on _how fine grained_ you want your control over the server configuration be.

If you are just starting out, then I might recommend you try out [Heroku](https://www.heroku.com/ "Heroku Cloud Application Platform"), mostly because of _how easy it is_ to get set up. You'll want to check out this [excellent article](http://www.rdegges.com/heroku-isnt-for-idiots/ "Heroku Isn't for Idiots"), too.

## Where Next?

Whoa, try and **digest everything you just learned**, first! Once you're done _researching on your own_, you can start thinking about [build processes](/search/tagged/build "Posts tagged 'build'").

Take the **MEAN** stack for a ride, seriously. You won't be disappointed!

  [1]: http://i.imgur.com/CAe1GHl.jpg
  [2]: http://i.imgur.com/AMuw2vF.jpg
  [3]: http://i.imgur.com/vgQYVRu.jpg "Yo Dawg!"