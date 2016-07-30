# Productivity, Then?

In a sense, it's _all about perceived productivity_. If you pause and think about it, we've made the shift towards client-side rendering because it's _"more productive"_. That's why the whole "no-backend" approach even exists. If we ignore the _poorly named_ "Offline First" philosophy for a minute, _"hey, you can implement this in under five minutes and not even have to spin up a server instance!"_ sounds like a reasonable goal.

But in reality, how much more productive are using Angular than using something else? What is productivity, really? Is it about saving time, keystrokes? Or is it about standing back, taking a walk, and being able to think a problem through and come up with a solution? Because, let's face it, some of our most productive engineering sessions **happen while we're in the shower**, not due to saving precious milliseconds with our over-engineered build systems _(where we actually spend more time than we used to, ironically)_.

We surely aren't all just building _throw-away prototypes_, are we? And even when we do, we are definitely **throwing them away**, since that's what they're for. Right?

In the complex world of front-end development that exists today, why are we even bothering to learn complicated domain languages in order to do something that's detrimental to the web platform? You could argue it's more productive, sure.

Productivity only takes you so far, though. There are lots of frameworks out there, and while developing in Angular might be confusingly fun, you don't really take away a lot from it. There's all these complicated abstractions that don't translate very well when you try to move on and use something else. I can almost hear somebody complaining just about now, "Angular's not that hard".

> Sure, it's not. But _ask yourself_, when are you going to ever come across things like **directives, transclusion, isolated scopes**, and factory service value providers that return a single integer?

You gain very little by specializing in Angular, because once the technology moves on, you'll have a harder time adapting, since you'll have to peel back many layers of abstraction until you can get back out to the real world.

# What is [Taunus][1]?

Taunus latches onto Express or Hapi and lays out a few conventions _(which you can configure as per the ["Convention over Configuration"][2] paradigm)_. To get started, you might want to look at the [Tutorial in the documentation][5], or play around with [giffy.club][3], a [silly site][4] we built to demonstrate how you can use Taunus to build your apps in a way that makes sense from a UX standpoint while still being server-side rendered.

Just because you're on an article on Pony Foo, we'll walk over how rendering an article works, since [ponyfoo][6] is open-source. First of all, we have a traditional Express application _(you can also use Hapi)_, with a few routes in it. Then we add a call to `taunus-express` to _mount_ Taunus in the back-end.

```js
taunusExpress(taunus, app, {
  routes: routes,
  layout: layout,
  getDefaultViewModel: getDefaultViewModel,
  plaintext: {
    root: 'article', ignore: 'footer,.mm-count,.at-meta'
  },
  deferMinified: production
});
```

Ignore the rest of the options for now, and let's focus on the `routes` property. This points to a module where the view routes that are used by Taunus are declared. You'll notice [the _exported array_ follows the patterns][8] in Express when it comes to routing, allowing you to use parameters, wildcards, regular expressions, and whatnot. The routes are ultimately used to set up raw Express `GET` routes with all the relevant middleware applied to them. You can also supply any extra middleware of your own, besides the controller which is assumed _(although optional)_.

These routes are kept in a module that exports an array because Taunus comes with a CLI that will be used to translate those Express routes into something the front-end understands. This is an important piece of the puzzle, because it means you don't have to maintain a list of routes for the front-end that's effectively a duplicate of the routes you use to render views in the server-side.

The route that we care about is `/articles/:slug`.

```js
{ route: '/articles/:slug', action: 'articles/article' }
```

This innocent looking route follows a few conventions. Because we indicated that the action is `articles/article`, Taunus comes up with a few expectations, all of which are optional, that is, these files don't _necessarily have_ to exist.

- A server-side controller in `$SERVER_CTRL/articles/article.js`
- A view exposed as `$VIEWS/articles/article.js`
- A client-side controller in `$CLIENT_CTRL/articles/article.js`

I intentionally used variables like `$SERVER_CTRL` to highlight the fact that all of these [directories are configurable][10].

# Server-Side Flow

The general idea in server-side controllers is that you're getting ready to tell Taunus **what** `action` to render and what `viewModel` to use. Pony Foo's `articles/article` action controller ends up populating the `viewModel` property in the response.

A controller example can be found below, inspired by the [author/compose][16] action. In this example we set the `viewModel` that'll be used but we omit the `action`. That means the `action` will be _inferred from the route_, just like the server-side controller's location was.

```js
module.exports = function article (req, res, next) {
  res.viewModel = {
    model: {
      articles: [{
        title: 'Stop Breaking the Web',
        description: '...'
      }]
    }
  };
  next();
  };
};
```

> Note also how we're using `next()` to fall into [Taunuses' rendering engine][17].

Once we've figured out the `viewModel` and `action`, Taunus will internally determine **how** to render the response, [based on request headers and query string][14]. If the request had asked for JSON, then the response is in JSON and it contains the `viewModel.model` for your partial view. If the request had asked for HTML, then the response will be in HTML, and rendered by using the template functions for both the layout and the partial view. When a request asks for plain text, Taunus renders an HTML response and [figures out the plain text][15].

![This article as a plain text response][31]

_<sub>This article can be rendered as plain text in terminal, just use `curl`!</sub>_

Following the progression we've laid out, suppose we are going to be handling the first request made by a browser against [ponyfoo.com/articles/random][18]. The server-side controller comes into play, decides what the `viewModel` should be, and it also has the ability to change what view should be rendered.

Then, _the view gets rendered_. This is done by compiling the view template and then [passing that to the layout function][24] as a plain HTML string `partial`.

# Views as Pure Functions in Taunus

Views are expected to be CommonJS modules that export a single method, the view method. Here's an example view method.

```js
module.exports = function (model) {
  return '<a href=' + model.url + '>' + model.title + '</a>';
};
```

Security terribleness aside, it can be nice that Taunus doesn't make lots of demands about view rendering engines, and instead just asks that you somehow write a `function(model)` that returns an HTML string, and place it in a module at `$VIEWS/$ACTION_NAME`.

Of course, expecting you to define your views _directly_ using JavaScript functions is kind of bonkers, as it'd be pretty cumbersome and clunky. Fortunately, most templating engines offer the ability to _"pre-compile"_ your views into JavaScript methods, and export those to files. Thus, you can pick your favorite template engine _(granted that it can compile views into JavaScript functions)_, and use that to code up your views for Taunus applications.

I _personally like [Jade][36]_, so I use that. The function above could be expressed in a Jade template as follows:

```jade
a(href=url)=title
```

The problem with Jade is that **their compiler inlines entire view templates** when you use the `include` keyword, making compiled views **fat**. I presume this is because Jade strives to not make assumptions about compiled views being _related_ which would mean they'd be able to use `require` statements instead of inlining the whole dependency tree. Inlining becomes an issue when you plan to reuse views in a few places, because your views get fat fast and there's a lot of duplicate code in the compiled templates. I made [jadum][11] to solve the duplication issue. It uses `require` statements pointed at other files when you use `include` in your Jade templates. Jadum _knows_ a view doesn't just exist in isolation, but that they are part of a whole and live in the tree structure `jadum` created on your behalf.

Why does Taunus want **pure** view functions? Because _they offer great simplicity_. It's simple for Taunus to use your views in both the server and the client, since these are just functions that you can `require`. This way, Taunus doesn't need to worry about providing its own view engine. There's **plenty** to choose from already.

The [articles/article.jade template][12] in Pony Foo is compiled into a module [using jadum][13], and then fed into Taunus. This sounds hard, but `jadum` lets you compile every single template in your app with [a single shell command][19].

```bash
jadum views/**/*.jade -o .bin --no-debug --obj '{"basedir":"views"}'
```

In Pony Foo, we keep compiled view functions in `.bin/views`, in a tree structure. After Taunus renders a view, it needs to take control of rendering in the client-side, when JavaScript finally loads. This is done in a couple of steps.

# The Client-Side "Wiring" Module

First off, we need a build step [using the Taunus CLI][9] to build out the "wiring module". This module is basically a collection of every assumption Taunus is going to make about your app.

Here is how the wiring module looks like for Pony Foo [_(except in reality it's actually a bit larger)_][21]. It follows the CommonJS module spec by convention, but you may produce a standalone `<script>` version if you're not using Browserify.

[![A screenshot of Taunuses' client-side wiring module][20]][21]

Why is this file necessary? A couple of reasons. First off, Browserify isn't able to read dynamic `require` statements, _unless they're look like `require(__dirname + '/foo.js')`_, and you definitely don't want to be typing all of these by hand. Secondly, if you're using Hapi then you can use the [hapiify transform][22] to convert Hapi routes into something [the front-end router][23] understands. Finally, this file is a clearly defined bridge between your back-end routes and what the front-end is doing.

All you have to do to generate the wiring module is run the following command.

```bash
taunus --output
```

# Mounting Taunus in the Front-End

Once we've incorporated the wiring module into our build process, we need to `.mount` Taunus in the front-end, just like we did in the back-end _(through `taunus-express`)_. Mounting [takes a few parameters][25]. It needs a DOM element that exists in your layout where the view should be rendered. It asks for the wiring module we've just generated. It also has some `options`.

```js
taunus.mount(main, wiring, { bootstrap: 'manual' });
```

> The `bootstrap` option asks the question: _"how are you going to get the `viewModel.model` to the client-side view controller on first load?"_

Once client-side JavaScript hits, Taunus is mounted on the client, and it starts to hijack link clicks _(and `<form>` submissions, unless you've disabled [gradual][26])_. Taunus will also immediately execute the client-side view controller for the current action, if one exists. That's why you need to figure out how to provide the `model`, because the client-side controller might need it!

When a link is clicked, instead of navigating away _like the web regularly does_, Taunus will issue an AJAX request. On the server-side your view controller will generate a view model just like it did the first time, and trust Taunus again to render the appropriate response. Taunus will notice the browser is asking for JSON this time around, [and respond in kind][27].

When the AJAX response gets back to the client-side, Taunus will figure out whether the `action` has been changed by the response, or if it should still use the route's default. Then, Taunus will render said action's view template by passing it the model we just received from the AJAX response. Once it has the view's HTML, it'll simply render it on the DOM element that was passed to `taunus.mount` a while back. Finally, the client-side view controller will be invoked, if one exists, just like on first load.

Along the way, Taunus emits events, caches responses _(if you want it to)_, and ensures the server-side is serving data for the view templates cached in the client-side _(otherwise you may want to reload the page because of outdated views in the client)_.

# How Much does Taunus _Assume_?

At first, the only assumption Taunus makes is that it can render views for most of the links that point to the current domain. That is, if there's a link to, say, _[/articles/first][28]_, it'll just be a link. Then, when Taunus gets mounted in the client-side, an event handler will be bound to that link. When the link gets clicked, Taunus will check if it matches one of its routes _(in this case, it matches the `/articles/first` route)_, and if it does, it'll trigger the AJAX mechanism and [use the history API][29] to behave just like a true **single page application**.

If the `history` API isn't supported, then Taunus will fall back to setting `location.href`, like in the old days. Sure, it's slower than using hashed routes. But it's more accessible, and it's not like there's that many users in browsers without a functioning `history` API nowadays anyway. The point is that the site will still work if the `history` API isn't there, because **Taunus is an upgrade on top of plainly server-side rendered HTML views**. This is key if we want to provide the baseline we discussed earlier, and we want to make _as few assumptions as possible_ while doing so.

# What About Data Binding?

Taunus doesn't offer a data binding mechanism out the box _(`skyrocket` is such a mechanism, but [it has a rough API][30] at the moment)_. Instead, it strongly expects you to rely on the progressive web. 

> The more **progressively** your site is going to be built, the more it makes sense to use Taunus to build it.

Let's suppose for a minute that you aren't such a big proponent on data-binding, because you feel sometimes it's too magic, confusing, or just because you got so far into this article, or all of the above. What else could we do to lighten the load on your end while providing a **progressively enhanced** experience where Taunus does the heavy lifting of improving the UX on your behalf _while you develop sane plain server-first web apps_?

# Gradual: Automated AJAX Form Submission

I've just recently open-sourced [gradual][26]. As we strive to simplify the usability of Taunuses' API, we've incorporated it into the core experience. It provides a mechanism to turn regular HTML forms into AJAX.

> This is powerful because we end up with **a pure SPA** out of _just HTML `<a>` and `<form>` tags._

For `gradual` to behave properly, you need to add support to your back-end so that `<form>` submission responses _(these aren't managed by Taunus)_ use a `Content-Type` of `text/plain` if the request contains `as-text` as a query string parameter. This is due to  `formium`, which makes form submissions against an `<iframe>` because it's better than AJAX _(you get a loading spinner and form autofill values are persisted in the browser)_. Some browsers render `application/json` responses inside `<pre><code>` tags or even prettier HTML pages, if the human has some sort of browser extension that does that.

Gradual also lays out **a few conventions** of its own. See, Taunus is a strong proponent of conventions, as they drastically simplify what you have to do as the implementor. That's why we build sites in this way. You set up an anchor tag, then it becomes an AJAX request that fetches a model, renders a view, and executes a controller. But you don't care about _how_, you just want it to _work_. Same goes for `<form>` submissions, you just want them to feel quicker.

Forms usually end up in a redirect. Given that you're using Taunus, you're probably using the [taunus.redirect][32] API in the server-side for all your form redirection needs. That method is just an abstraction that either does an HTTP redirect or responds with a JSON payload that denotes a redirect. If `gradual` receives one such response, it'll make sure to follow the redirect. This is probably the easiest way to handle a form submission's response. You just redirect somewhere else, if it was made by the browser using plain HTML, then the redirect is followed by the browser, and if Taunus made the submission, then the _(JSON)_ redirect is followed by `gradual`.

Another _very_ common kind of response is validation. In the traditional web you simply add some messages to the `session` and discard them after one use _(a so-called session "flash")_. Gradual is keen on taking these validation messages from [a variety of places][33] in your response and displaying them accordingly.

So what is it you should do in a server-side controller that handles a form submission?

- Flash via session
- Redirect via Taunus

Taunus will then grab the flash messages _(if any)_ and display them using the `partials/form-validation` view template.

# Realtime Communication with Skyrocket

Here is where things get really interesting. It takes some setup _(we're definitely **trying** to make this one simpler)_, but when you do set it up, **it's pretty awesome**.

The idea is that a form submission can end up in a view update, instead of a full redirect, because even if the redirect goes through Taunus, it _still has to hit the server again_. In `skyrocket` we came up [with a schema][34] where you're able to respond with `model` updates you want to apply to a `model`, or _operations_ you want to apply to a certain _path_ on the `model`.

Consider for example the following response:

```js
{
  updates: [{
    rooms: ['/issues/34'],
    model: {
      title: 'Implement realtime communication via skyrocket'
    },
    operations: [{
      op: 'edit',
      concern: 'todos.1',
      model: {
        completed: true
      }
    }]
  }]
}
```

This works in a pretty straightforward manner. If the person getting this response to a form submission that marked a TODO as completed, is listening for events on the `/issues/34` rooms, he gets back an update that will be automatically applied against the model in his view controller. He'll then get a callback where he can re-render parts of the view as he deems necessary. The client-side controller might look like this:

```js
var $ = require('dominus');
var skyrocket = require('skyrocket');

module.exports = function controller (model, container, route) {
  var rocket = skyrocket.scope(container, model);

  rocket.on('/issues/34', function reaction () {
    // the model has been updated at this point
    var todos = $.findOne('.todos', container);
    var action = 'issues/todos';
    taunus.partial(todos, action, model);
  });
};
```

That'll be enough to apply changes to the view when the model changes, immediately, as the response comes through, without the need for a redirection. We can also do one better. If you've configured Skyrocket for realtime, then you're also able to broadcast this update, using **the exact same schema**, to other  connected clients interested in the `/issues/34` room _(or "entity")_, and they'll also be able to handle that just as well, because the code is routed through the same handlers, except that the data is getting routed through WebSockets _(e.g `socket.io`)_ in realtime.

You don't have magical data binding in Taunus, other than the magical model updates via [Skyrocket][35], but you do get the ability to easily update portions of your view via WebSocket just by implementing an HTML `<form>` response handler.

That's pretty rad.

_P.S: I'm running a little survey regarding the content direction in this blog, if you'd like to help me with your feedback, [come this way][37]._

[1]: https://taunus.bevacqua.io "Taunus documentation site"
[2]: https://en.wikipedia.org/wiki/Convention_over_configuration "Wikipedia definition"
[3]: https://github.com/taunus/giffy.club "giffy.club on GitHub"
[4]: http://giffy.club "giffy.club"
[5]: http://taunus.bevacqua.io/getting-started "Getting Started with Taunus"
[6]: https://github.com/ponyfoo/ponyfoo "ponyfoo/ponyfoo on GitHub"
[7]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/controllers/routing.js#L70-L78 "Adding Taunus view routes with taunus-express"
[8]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/controllers/routes.js#L19 "The articles/article route"
[9]: http://taunus.bevacqua.io/api#command-line-interface "Taunuses' command-line interface documentation"
[10]: http://taunus.bevacqua.io/api#the-taunusrc-manifest "The .taunusrc manifest"
[11]: https://github.com/bevacqua/jadum "bevacqua/jadum on GitHub"
[12]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/views/shared/articles/article.jade "articles/article Jade view template for Pony Foo"
[13]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/build/build-production#L17 "Build script for Pony Foo in production"
[14]: https://github.com/taunus/taunus/blob/a26dcc6b0da926e49f60952aadc093bebef3a933/lib/render.js#L70-L76 "The code is pretty easy to follow, taunus/taunus on GitHub"
[15]: https://github.com/bevacqua/hget "Render websites in plain text from your terminal with hget"
[16]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/controllers/author/compose.js "The author/compose server-side controller on ponyfoo/ponyfoo"
[17]: https://github.com/taunus/taunus/blob/a26dcc6b0da926e49f60952aadc093bebef3a933/lib/render.js#L41 "Taunuses' rendering engine on GitHub"
[18]: /articles/random "What will it be!?"
[19]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/build/build-production#L17 "How is Pony Foo built for production?"
[20]: https://i.imgur.com/rIvOCQz.png
[21]: https://gist.github.com/bevacqua/bc169cc641c356ee97d3 "The wiring module in ponyfoo/ponyfoo when autogenerated by `taunus -o`"
[22]: https://github.com/taunus/hapiify "taunus/hapiify on GitHub"
[23]: http://github.com/bevacqua/ruta3 "bevacqua/ruta3 on GitHub"
[24]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/views/server/layout/layout.jade#L65 "Rendering the view within the layout"
[25]: https://github.com/ponyfoo/ponyfoo/blob/34e81b2b676f9e4f7f12f2d3cec2a061aa4796ad/client/js/main.js#L24 "Mounting ponyfoo/ponyfoo on GitHub"
[26]: https://github.com/taunus/gradual "taunus/gradual on GitHub"
[27]: /articles/progressive-web?json "The Progressive Web as JSON"
[28]: /articles/first "Pony Foo Begins"
[29]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history "Manipulating the browser history"
[30]: https://github.com/taunus/skyrocket "taunus/skyrocket on GitHub"
[31]: https://i.imgur.com/6SfH143.png
[32]: http://taunus.bevacqua.io/api#-taunus-redirect-req-res-url-options- "Check out the taunus.redirect documentation"
[33]: https://github.com/taunus/gradual#custom-validation "Custom validation in Gradual documentation on GitHub"
[34]: https://github.com/taunus/skyrocket#skyrocket-schema "The Skyrocket realtime update schema on GitHub"
[35]: https://github.com/taunus/skyrocket "Skyrocket on GitHub"
[36]: http://jade-lang.com/ "Jade Template Engine"
[37]: https://docs.google.com/forms/d/1pccaq_Tq0QSGKbP9czdEq0uPbFzthKmBY1Kvkjl2Slc/viewform "The survey is on Google Forms"
