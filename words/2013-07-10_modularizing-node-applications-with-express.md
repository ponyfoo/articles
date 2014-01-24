# Modularizing Node Applications with Express

I've spent a few articles talking about [build processes](/search/tagged/build "Search for posts tagged 'build'"); now I want to spend a few words on [application architecture](/search/tagged/architecture "Search for posts tagged 'architecture'"), particularly in _Node.JS web applications using Express_.

In this article, I want to talk about the big picture: **how to separate concerns of applications on different sub-domains in a clean manner**.

Sometimes, you need to deal with requests on two separate sub-domains in your application, say `www` and `blog`, but more often than not, this happens within the _same express application_. While it's _certainly possible_ to handle the routing using a single module, I've come to realize **it's best to use a modular approach**, similar to what [connect.vhost](http://www.senchalabs.org/connect/vhost.html "connect.vhost - Connect") does, but taken up a notch!

[![express.png][1]](http://expressjs.com/ "Express.JS Web Application Framework")

`connect.vhost` simply creates a middleware we can `.use()` in our application, passing it a new server instance for each _host_ we want to handle. With it, we can set up a folder structure that looks _somewhat like the following_:

```
src
  - server.js
  + hosts
    + www
      - vhost.js
    + blog
      - vhost.js
```

With that folder structure in mind, we could code up our `server.js` file using:

```js
'use strict';

var express = require('express');
var app = express();
var www = require('./hosts/www/vhost.js');
var blog = require('./hosts/blog/vhost.js');

app.use(www);
app.use(blog);
app.listen(3000, function(){
    console.log('listening on port 3000');
});
```

Thus, `server.js` would just contain the logic necessary to enable modularization, and nothing more. Before diving into one of our `vhost.js` files, lets figure out how to build a _better `vhost` middleware_. Here is what I propose:

```js
'use strict';

function buildHostRegex(input){
    if (typeof input !== 'string'){
        return input;  
    }
    
    var hostname = input || '*';
    if (hostname.test(/[^A-z_0-9.*-]/)){
        throw new Error('Invalid characters in host pattern: ' + hostname);
    }

    var pattern = hostname
        .replace(/\./g, '\\.')
        .replace(/\*/g,'(.*)');

    var rhost = new RegExp('^' + pattern + '$', 'i');
    return rhost;
}

module.exports = function(factory){
    return function(hostname){
        var rhost = buildHostRegex(hostname);

        var middleware = function(req, res, next){
            if(middleware.on === false){
                return next();
            }

            if(!req.headers.host){
                return next();
            }

            var host = req.headers.host.split(':')[0];
            if(!rhost.test(host)){
                return next();
            }

            if (typeof middleware.server === 'function'){
                return middleware.server(req, res, next);
            }
            middleware.server.emit('request', req, res);
        };

        middleware.server = factory();

        middleware.off = function(){
            middleware.on = false;
        };

        middleware.on = function(){
            middleware.on = true;
        };

        return middleware;
    };
};
```

Using [this module](https://github.com/bevacqua/virtual-host "virtual-host on GitHub"), this separation of concerns becomes almost trivial.

Our `./www/vhost.js` implementation can become:

```js
'use strict';

var express = require('express');
var vhost = require('virtual-host')(express);
var www = vhost('www.*');
var app = www.server;

app.use(function(req,res,next){
    res.send('response from www!');
});

module.exports = www;
```

The important factor in this separation is that we can handle _views, static assets, routing, and pretty much every aspect_ of our application in a self-contained manner. We could always _re-use common pieces of functionality_ through `require`, as per usual.

I can think of a few use cases for this sort of modularization

- Setting up a sub-domain for **API documentation, blog, or custom solution**, while _keeping our main application unaltered_, when resorting to a third-party solution for one of our sub-domains
- Setting up a request catch-all which is great for displaying a **different application altogether** for the initial setup of open-source projects, _helping developers_ to set up their local environments
- Setting up a maintenance catch-all which can be turned on and off as a service, allowing us to have an **emergency off switch** without necessarily turning off job queues, and allowing for a more responsive alternative to turn the server back online

How might the maintenance setup work? Probably something close to this:

```js
'use strict';

var express = require('express');
var app = express();
var www = require('./hosts/www/vhost.js');
var maintenance = require('./hosts/maintenance/vhost.js');

app.use(maintenance);
app.use(www);
app.listen(3000, function(){
    console.log('listening on port 3000');
});
```

In the maintenance `vhost.js`:

```js
'use strict';

var express = require('express');
var vhost = require('virtual-host')(express);
var maintenance = vhost('*'); // catch-all
var app = maintenance.server;

app.use(function(req,res,next){
    res.send('the server is in maintenance mode!');
});

maintenance.off(); // disabled by default

module.exports = maintenance;
```

We might have a _background job_ that allows us to **turn the maintenance vhost on or off**, and that would be it. What's even better, it would be _reusable_ in any applications we craft following this pattern.

If you want to read _more articles like this_, follow the [RSS feed](/rss/latest.xml "RSS feed for this blog"), or subscribe to the email updates!

  [1]: http://i.imgur.com/0q7WUxz.png