Putting together a client-side MVC framework is no easy feat. [I should know][1]. I tried back in the day, when putting together this blog engine. I failed misreably back then. This time, though, I made an effort to mix the best of both worlds. I decided to go back to basics and write a framework that was built around modularity, MVC, events, and shared-rendering.

## Taunus Architecture Overview

The hardest dependency you'll find in Taunus is on Express. Taunus itself doesn't utilize Express, but it expects routes to be in the same format that Express expects, and it also expects you to pass it an object with a `get` method, which will be called once for every route, with a few middleware functions. As long as you're able to comply with those terms, Taunus will work well for you. The routing constraint is particularly interesting, because Taunus uses [the Express router][2] on the client-side. Thus, using that same router in the server-side becomes a necessity for consistency.

That being said, there's nothing to stop you from making use of `routes` on an `http` server instance, pass a `{ get: fn }` object to `taunus.mount`, and ditch Express!

Taunus deals mainly in the four components explained below. When Taunus is hit with a request, the router will call your controller action in the server-side. Once the controller action method is done, the view renderer will take the model provided by the action, and render the view. The server is now done. When the client-side starts up, it'll invoke the client-side controller for that view. Suppose now that another page is requested. Taunus will query that same endpoint using an `Accepts: 'application/json'` header, and the server will return JSON instead of the fully rendered page. Taunus uses that JSON data in the client-side to render your view, and then calls the client-side controller, just like it did on page load.

![architecture.png][3]

It's easiest to explain how Taunus works with an example walk-through.

1. Incoming request `/articles/taunus-is-awesome`
1. Router matches `/articles/:slug`, and invokes the controller for that action
1. The `articles/slug` action controller fetches a model from the database
1. In the controller action, we assign a model to `res.viewModel`, and call `next()`
1. Taunus renders the partial for this view, and wraps it with the view layout
1. The client-side takes over, and it figures out its the first time around, so it doesn't re-render the view
1. The client-side view controller gets invoked

That's as far as first-time execution goes. Let's move on.

1. You click on "About this blog", which is a link to `/about`
1. Taunus captures that click and issues an XHR request for `/about`, asking for JSON data
1. Incoming request `/about`
1. Router matches `/about`, and invokes the controller for that action
1. The `home/about` action controller fetches a model from the database
1. In the controller action, we assign a model to `res.viewModel`, and call `next()`
1. Taunus responds with the JSON model as-is
1. The client-side takes over, this time it renders the view, using the model from the AJAX call
1. The client-side view controller gets invoked

Note how steps 3 through 6 are exactly the same in both cases. Note how the last step is the same, too. Also note that the steps that are different are dealt with by Taunus, so you don't have to worry about them.

The cool part about Taunus is that it doesn't rely on Backbone, or jQuery, or any client-side libraries. That's up to you. Taunus only handles view routing and the purely MVC aspect of you application. Since it's Common.JS you get dependency injection for free, and the conventional approach means that you won't have to worry about routing in more than one place. You can use the browser's native APIs for pretty much everything you want, so **Taunus doesn't want you to marry it**. That's a pretty good thing, because it's [a car model line][4], and that'd be awkward.

It doesn't really break apart from what you're used to do in Express. You just do your thing, set a view model on the response, and call `next()`. Let's see how Taunus is actually used.

## Taunus On The Server

Taunus provides three different components that operate together, offering a sane development model that doesn't get in your way. Let's start with the routes. In a typical Express application, you'd register routes by hand, like below.

```js
app.get('/pony/foo', pony.foo);
app.get('/pony/bar', pony.bar);
app.get('/author/moo', author.authenticated, author.moo);
```

With Taunus, you're expected to define your view routes in a JSON format instead. It's recommended you put these into an individual module, like below. Note that, under the default module path resolver, controllers are expected to be defined on a per-action basis, and the same goes for views. In fact, the `action` property in the route is used to infer which view should get rendered by Taunus.

```js
var authorAuthenticated = require('./author/authenticated');

module.exports = [
  { route: '/pony/foo', action: 'pony/foo' },
  { route: '/pony/bar', action: 'pony/bar' },
  { route: '/author/moo', action: 'author/moo', middleware: authorAuthenticated }
]
```

Once you've created the routes module, you can boot up `taunus`. Here's a module which takes an Express instance and creates routes in it. Note how I'm perfectly able to mix my Taunus view routes with any other routes I have, such as API routes or error handling middleware.

```js
var taunus = require('taunus');
var routes = require('./routes');
var articleList = require('./article/list');
var errors = require('../lib/errors');

module.exports = function (app) {
  app.get('/api/articles', articleList);
  taunus.mount(app, routes);
  app.use(errors.handler);
};
```

A controller action might look like the snippet below. The `model` property on the view model is what gets passed to partial view template functions. The layout will get the full `viewModel`, instead.

```js
module.exports = function (req, res, next) {

  // fetch data from somewhere
  // var user = ...

  // assign it to the view model
  res.viewModel: {
    model: {
      title: 'User Profile',
      user: user
    }
  };

  // we're done, yield control over to the renderer
  next();
}
```

#### Sharing Routes and View Templates

The client-side portion of your application is where things get a bit more interesting. Like I explained back in the architecture section, the client-side will use the same routes and view templates that the server-side uses. Here's where we run into trouble. Browserify is awesome. Seriously, ragingly awesome. It does come with certain limitations, such as the inability to parse dynamically composed `require(expr)` expressions at compile time. Browserify is only smart enough to figure out how to unwrap `require('expr')` calls. The issue is that you want to initialize Taunus on the client-side as well, and for that you'll need a `routes` object which looks somewhat like the snippet of code below.

```js
module.exports = [{
  route: '/',
  template: require('../views/home/index'),
  controller: require('../../client/js/controllers/home/index')
}, {
  route: '/author/compose',
  template: require('../views/author/compose'),
  controller: require('../../client/js/controllers/author/compose')
}];
```

The above might look fine for two routes, but imagine maintaining that by hand? It's pointless! You already have a routes module, the one you used on the server. Surely you can build a small script that turns your server-side routes into these client-side routes in a heartbeat! That's what I did!

## Taunus On The Command-Line

Taunus comes with a small CLI interface that can compile your routes. Without any options, the program will print a client-side routing module to standard out.

```shell
taunus
```

The CLI comes with a few options. Currently, these options are available, among others that aren't worth mentioning in this article.

Option                      | Description
----------------------------|----------------------------------------------------------------------------------------------------
`-o`                        | Instead of `stdout`, the output is dumped to the `client_routes` file, as defined in `.taunusrc`
`-w`                        | Watch for changes to the server-side routes and recalculate the output. Works well with `-o`
`--standalone path/to/file` | Export `taunus` as a global, and your routes, using a single stand-alone bundle file

**Taunus wants you to use Browserify really bad**, although you have the option of not using Common.JS in your own code, and that's okay.

> Can I avoid using Browserify?
>
> -- **Yes!** However, you'd still have to compile Taunus using Browserify, like below. This command won't just Browserify Taunus, but it'll also compile your route definitions into client-side routes, and place them in `taunus.routes`! The `-w` flag still works just fine. `-o` is implied.
>
> ```shell
> taunus --standalone client/js/vendor/taunus.js -w
> ```

Once you've compiled your routes and browserified your bundle, you can set up Taunus on the client-side.

## Taunus on the Browser

If you went for Browserify, the code will be nicely modular. _[Precious, _precious modularity_][5]_. Pat yourself on the back. [Click here for an example build file][6].

```js
var taunus = require('taunus');
var routes = require('path/to/client-side/routes');
var elem = document.querySelector('main');
taunus.mount(elem, routes);
```

If you've decided to ditch Browserify, then it'll be even easier to set up!

```js
var elem = document.querySelector('main');
taunus.mount(elem, taunus.routes);
```

Once that's set up, all you'll need to do is create your routes, view templates, server-side and client-side controllers! Taunus will be mostly be out of your way. You can, however, get in the way of Taunus, by listening to the [events it emits][7] and reacting to them.

There's still a long way to go, but I'm sure it'll drastically improve as I toy with it and learn about its weaknesses!

> What do you think about Taunus?

  [1]: /2012/12/29/single-page-design-madness "Single Page Design Madness (2012)"
  [2]: https://github.com/aaronblohowiak/routes.js "Minimalist routing library, extracted from connect"
  [3]: https://i.imgur.com/cHacWdA.png "Taunus Stack"
  [4]: http://en.wikipedia.org/wiki/Ford_Taunus "Ford Taunus on Wikipedia"
  [5]: /2014/05/16/modularizing-your-front-end "Modularizing Your Front-End"
  [6]: https://github.com/bevacqua/ponyfoo/blob/158a1a130d35f77b2d6f0c22f8651a6f21334083/build/debug "`npm start`!"
  [7]: https://github.com/bevacqua/taunus#events "Taunus Events on GitHub"
  [8]: http://i.imgur.com/mfVdU9V.png "01_design.png"
