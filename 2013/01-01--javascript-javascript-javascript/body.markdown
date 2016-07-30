I've read a few articles about _Node_ and _MongoDB_, but I hadn't really had any **practical experience** yet. I asked myself a few questions, and I feel like _I need to answer at least some of them before going forward_.

- How I would prevent or mitigate _over-posting_ issues?
- How am I going to perform **model validation**?
- How am I going to **separate concerns**?
- How am I going to **structure the application**? I probably should refer to a framework such as [Backbone.JS](http://backbonejs.org/ "Backbone.JS"), [Knockout.JS](http://knockoutjs.com/ "Knockout.JS"), or the like.
- Am I going to use the native _MongoDB_ [driver](http://www.mongodb.org/display/DOCS/Drivers)? Am I going to find [Mongoose ODM](http://mongoosejs.com/ "Mongoose Object Document Mapper") useful at all?
- Lastly, _how the **heck** am I going to render views that require a model_? Will I pair my templating engine with some other [templating](http://mustache.github.com/ "Mustache") [language](http://handlebarsjs.com/ "Handlebars")? Will my implementation do _just fine_ on its own?

> I sifted through [Learning Node](http://www.amazon.com/dp/1449323073 "Learning Node, O'Reiley, Aug 2012"), it serves as a pretty decent **introductory crash-course** and it answered the most basic questions I had.

# Refactoring Node.JS #

The template I started off with had just a `server.js` file that contained everything server-side: http server configuration, error handling, routing, route behavior, etc. I want a more **robust separation of concerns** going forward.

After going through the book, I formed a good idea of how I wanted to structure the server application.

The first thing I'm going to do is to **modularize** the application, I'll _separate concerns_ by having routing modules, controllers, and models, all in _separate logical files_.

## [module.exports](http://nodejs.org/api/modules.html "Node.JS docs") in Node.JS ##

Node.JS has a cute way of separating concerns in what's called _modules_. A module is **self-contained** and exposes an API providing access to explicitly defined methods.

Here's an example **dependency.js**:

```js
var uid = 0; // local

module.exports.startsWith = function(str, text){
    return str.indexOf(text) === 0;
};

module.exports.uid = function(){
    return ++uid;
};
```

In your **server.js**, you would reference it like this:

```js
var dep = require('./dependency.js'),
    model = {
        sn: dep.uid();
        text: 'dependency flavored model'
    };

// ...
```

Not the best of examples, but you get the idea.  

## Routing and Controller Actions ##

Lets examine a more practical example, I want to have my routing defined somewhere else, rather than directly on **server.js**, so I'll replace my routes declaration with the following call, and _defer_ the implementation to another file:

```js
require('/routing/core.js')(server);
```

With that simple statement I can pass the `server` object, and handle any routing directly in my self-contained module.

```js
module.exports = function(server){
    server.get('/*', function(req,res){
        res.render('site.jade');
    });

    server.post('/write-entry', function(req,res){
        console.log(req.body.entry);
        res.end();
    });
};
```

This is pretty awesome, but all I really want from my routes is to _juggle request parameters around_, and I'll leave any real processing to the _controllers_. The `main` controller will be a really thin one:

```js
module.exports = {
    get: function(req,res){
        res.render('site.jade');
    }
};
```
    
For the endpoint previously referred to as `POST /write-entry`, I'll favor a more [RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer "REST API definition") approach this time. The controller pretty much will remain unchanged:

```js
module.exports = {
    put: function(req,res){
        console.log(req.body.entry);
        res.end();
    }
};
```

The routing module ends up being:

```js
var main = require('../controllers/main.js');
var entry = require('../controllers/entry.js');

module.exports = function(server){
    server.get('/*', main.get);

    server.put('/entry', entry.put);
};
```
    
All this code might seem redundant at first, but it will gain value as our application grows.

> As you might have noticed, my attempt at being RESTful is hindered by the rich application structure where all `GET` requests serve the same `text/html` response. This could be easily be mitigated in the future by considering some other endpoint for any **non-static** resource, such as:

- **GET** http://ponyfoo.com/api/1.0/entry _fetch_
- **PUT** http://ponyfoo.com/api/1.0/entry _upsert_
- **DELETE** http://ponyfoo.com/api/1.0/entry/:id _delete_ 

> I'll get to this later, but before release.

## Models in MongoDB, introducing [Mongoose](http://mongoosejs.com/ "Mongoose ODM") ##

So far we've covered _views_, _controllers_, _routes_, but we haven't actually done anything worthwhile in the server-side. Now we'll plunge into **MongoDB**. I decided to use **Mongoose** after exploring my options a little and realizing how _much simpler_ my development would be having it around.

### Setting up the database environment ###

Install the mongoose package through [npm](https://npmjs.org/).

```bash
$ npm install mongoose
```

Then we need a little initialization code to get things going:

```js
var mongoose = require('mongoose');

mongoose.connect(config.db.uri); // configured in config.json
mongoose.connection.on('open', function() {
    console.log('Connected to Mongoose');
});
```

I set up a tentative `config.db.uri = mongodb://localhost/ponyfoo`. This code will _blatantly fail upon execution_, due to the simple fact that _we didn't fire up MongoDB yet_, so we'll go ahead and do that now.

[Download and install MongoDB](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-windows/ "MongoDB setup instructions on Windows").

> I set up **MongoDB** on my development hard drive, sitting at `f:\mongodb`, I configured the data folder within in a `\data` sub-directory, and I also created a succint batch file to fire up **MongoDB** from my project root:

```bash
$ f:\mongodb\bin\mongod.exe --config f:\mongodb\mongod.conf
```
    
The batch file just starts the _MongoDB_ database server the configuration file I created for specifying the `data` folder directly in the _MongoDB_ installation folder. It specifies a  `data` folder:

```
dbpath = f:\mongodb\data
```

That's **it**, you should be able to establish a connection through **mongoose**. Try it now.

Coming from the _Microsoft stack_ and particularly **SQL Server**, I must admit I feel incredibly good about _MongoDB_, and I really appreciate being _this easy_ to set up.

### Our first database schema ###

I will **maintain a modular approach** to models as well, thus, anywhere I need them, I'll just `require` my models module:

```js
var models = require('./models/all.js');
```
    
This module will, in turn, provide a list of _MongoDB_ **document** models we can use:

```js
var entry = require('./entry.js');

module.exports = {
  entry: entry.model
};
```
    
And lastly, each model should expose its schema:

```js
var mongoose = require('mongoose'),
    schema = new mongoose.Schema({
        title: String,
        brief: String,
        text: String,
        date: Date
    });

module.exports.model = mongoose.model('entry', schema);
```

### Upsert world ###
    
This [upsert command](http://mongoosejs.com/docs/api.html) is all that's left between our UI and the _MongoDB_ database:

```js
var mongoose = require('mongoose'),
    models = require('../models/all.js');

module.exports = {
    put: function(req,res){
        var collection = models.entry,
            document = req.body.entry,
            query = { date: document.date },
            opts = { upsert: true },
            done = function(err){
                res.end();
            };

        collection.findOneAndUpdate(query, document, opts, done);
        collection.save();
    }
};
```
    
The last little detail would be updating the UI to actually provide a `date` for our _entries_. We'll deal with that soon enough.

Thus far, this will create an _entry_ the first time, and overwrite it in every subsequent request. This makes the method _ideal_ for easily editing our blog post _entries_, we would just need to wire the UI to provide the identifier we're using for the **upserts**.
    
### MongoDB document identifiers ###

At a glance, you better have a [damn good reason](http://docs.mongodb.org/manual/tutorial/create-an-auto-incrementing-field/) for having an auto-increment _id field. Every single blog post or [Stack Overflow](http://stackoverflow.com "Stack Overflow") [answer](http://stackoverflow.com/q/8384029/389745 "Auto increment in MongoDB") that tells you how to implement an auto-incrementing field _also tries to persuade you **not to do it**_.

I only wanted to have an incremental _id for _routing purposes_.

However, considering all the **evidence against this kind of fields**, I came up with a simple solution, which collaterally produces _better urls_.

I'll simplify it to be:

> [/2013/01/01/javascript-javascript-javascript](/2013/01/01/javascript-javascript-javascript)
    
Now my only constraint is not writing two separate entries with the exact same title and assign them the exact same date.

> **`/:yyyy[/:mm[/:dd[/:slug]]]`**

This approach makes _url hacking_ so easy even a zombie could try to do it.

So there we have a drastically basic _Node.JS_ application that allows us to `PUT` a _MongoDB_ document, and does nothing much besides that, but we did set up a _solid working base_ for code to come.
