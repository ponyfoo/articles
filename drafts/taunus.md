# Taunus

I've mentioned [Taunus][2] in [one of my latest articles][1]. I believe Taunus _is interesting_, not because it introduces innovative paradigm shifts or the like, but rather, because it takes a proven concept, and iterates upon it. Bluntly put, Taunus builds upon the design of [Rendr][4]. By all accounts, Rendr was amazing. I read a lot about Rendr before even trying it out. I was really excited about it. What could be wrong? I mean, you had convention over configuration, shared rendering, and reused modules. If you tried either Ruby on Rails or _ASP.NET MVC 3+_, both of those are pretty conventional as well. Eventually, I had to use Rendr in order to determine if it was _"good enough"_ to recommend it in [JavaScript Application Design][11], as the "go-to" approach for shared-rendering in large scale applications.

That would set me up to write about Rendr in my book and forget about shared rendering. I figured it'd be mostly a drop-in plugin for Backbone. I was thorougly wrong. Getting started with the bare minimum viable Rendr is **nothing short of painful**. You can check it out for yourself. The [so-called "simple" example][6] in Rendr's repository on GitHub almost rendered me to tears. I get it. Rendr requires Backbone. Backbone requires jQuery. Fine, I accept those terms. Rendr takes a **"Convention over Configuration"** approach. That's awesome! I loved that back when I was involved in C# MVC application development. Well, yeah. Except Rendr isn't so much about **"Convention over Configuration"**, but more about **"Convention, deal with it"**. To the point where you must conform to multiple different rules.

- You [**must** browserify][7] your client-side code in obscure ways
  - jQuery must be shimmed even though it's available on `npm`
  - Furthermore, the server-side piece of Rendr uses a hard-coded version of jQuery, independently of the one you pick for the client-side
  - You must define aliases in a so-and-so way ([Richard Feynmann tainted my writing style][8], so be it)
- Your templates **must** be placed into `app/templates/compiledTemplates.js`. Seriously?
- Handlebars. Backbone. jQuery. Deal with it.
- Awkward APIs like `this.app.fetch`
- A thorough lack of documentation

After asking [Spike Brehm][9] about it on Twitter, and figuring out [he's pretty much moved on][10] from Rendr, I've decided not to adopt it, and strike out on my own. I'm really looking forward to the **Building Isomorphic Apps** talk, which he'll be giving in JSConf in a couple of weeks.

[![taunus.png][3]][2]

Putting together a client-side MVC framework is no easy feat. [I should know][12]. I tried back in the day, when putting together this blog engine. I failed misreably back then. This time, though, I made an effort to mix the best of both worlds. I decided to go back to basics and write a framework that was built around modularity, MVC, events, and shared-rendering.

## Taunus Architecture Overview

The hardest dependency you'll find in Taunus is on Express. Taunus itself doesn't utilize Express, but it expects routes to be in the same format that Express expects, and it also expects you to pass it an object with a `get` method, which will be called once for every route, with a few middleware functions. As long as you're able to comply with those terms, Taunus will work well for you. The routing constraint is particularly interesting, because Taunus uses [the Express router][14] on the client-side. Thus, using that same router in the server-side becomes a necessity for consistency.

That being said, there's nothing to stop you from making use of `routes` on an `http` server instance, pass a `{ get: fn }` object to `taunus.mount`, and ditch Express!

Taunus deals mainly in the four components explained below. When Taunus is hit with a request, the router will call your controller action in the server-side. Once the controller action method is done, the view renderer will take the model provided by the action, and render the view. The server is now done. When the client-side starts up, it'll invoke the client-side controller for that view. Suppose now that another page is requested. Taunus will query that same endpoint using an `Accepts: 'application/json'` header, and the server will return JSON instead of the fully rendered page. Taunus uses that JSON data in the client-side to render your view, and then calls the client-side controller, just like it did on page load.

![architecture.png][13]

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

The cool part about Taunus is that it doesn't rely on Backbone, or jQuery, or any client-side libraries. That's up to you. Taunus only handles view routing and the purely MVC aspect of you application. Since it's Common.JS you get dependency injection for free, and the conventional approach means that you won't have to worry about routing in more than one place. You can use the browser's native APIs for pretty much everything you want, so **Taunus doesn't want you to marry it**. That's a pretty good thing, because it's [a car model line][15], and that'd be awkward.

It doesn't really break apart from what you're used to do in Express. You just do your thing, set a view model on the response, and call `next()`. Let's see how Taunus is actually used.

## Taunus As A Package

Taunus provides three different components that operate together, offering a sane development model that doesn't get in your way.

- Server-side API
- Client-side API
- Command-Line Interface









[1]: /2014/05/16/modularizing-your-front-end "Modularizing Your Front-End"
[2]: https://github.com/bevacqua/taunus "Taunus: Micro MVC Framework"
[3]: https://camo.githubusercontent.com/b98a5dc441b3a71a01e2e46639ddf57737c2c721/68747470733a2f2f7261772e6769746875622e636f6d2f62657661637175612f7461756e75732f6d61737465722f7265736f75726365732f7461756e75732e706e67
[4]: https://github.com/rendrjs/rendr "airbnb/rendr on GitHub"
[5]: /2014/05/07/shared-rendering-with-rendr "Shared Rendering with Rendr"
[6]: https://github.com/rendrjs/rendr/tree/master/examples/00_simple "Simple Rendr App Template"
[7]: https://github.com/rendrjs/rendr/blob/master/examples/00_simple/Gruntfile.js#L66-L98 "Rendr Browserify in Gruntfile"
[8]: http://www.amazon.com/Surely-Feynman-Adventures-Curious-Character/dp/0393316041 "Surely You're Joking, Mr. Feynman!"
[9]: https://twitter.com/spikebrehm "@spikebrehm on Twitter"
[10]: https://twitter.com/spikebrehm/status/461939437585190912
[11]: https://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
[12]: /2012/12/29/single-page-design-madness "Single Page Design Madness (2012)"
[13]: http://i.imgur.com/cHacWdA.png "Taunus Stack"
[14]: https://github.com/aaronblohowiak/routes.js "Minimalist routing library, extracted from connect"
[15]: http://en.wikipedia.org/wiki/Ford_Taunus "Ford Taunus on Wikipedia"

[isomorphic taunus browserify npm-run]
