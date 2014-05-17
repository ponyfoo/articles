# Taunus

I've mentioned [Taunus][2] in [one of my latest articles][1]. I believe Taunus _is interesting_. Not because it introduces innovative paradigm shifts or the like, but rather, because it takes a proven concept, and iterates upon it. Bluntly put, Taunus builds upon [Rendr][4]. By all accounts, Rendr was amazing. I read a lot about Rendr before even trying it out. I was really biased about it. What could be wrong? I mean, you had convention over configuration. I'm sure you've experienced either Ruby on Rails or _ASP.NET MVC 3+_, and both of those feel pretty good convention-wise. Eventually, I had to use the library in order to determine if it was _"good enough"_ to recommend it as the "go-to" approach for shared-rendering in large scale applications.

That would set me up to write about Rendr in my book and forget about shared rendering. I figured it'd be mostly a drop-in plugin for Backbone. I was thorougly wrong. Getting started with the bare minimum viable Rendr is **nothing short of extremely painful**. You can check it out for yourself. The [so-called "simple" example][6] in Rendr's repository on GitHub renders you to tears. I get it. Rendr requires Backbone. Backbone requires jQuery. Fine. Rendr takes a **"Convention over Configuration"** approach. That's awesome! I loved that back when I was involved in C# MVC application development. Well, yeah. Except Rendr isn't so much about **"Convention over Configuration"**, but more about **"Convention, deal with it"**. To the point where you must conform to multiple different rules.

- You [**must** browserify][7] your client-side code in obscure ways
  - jQuery must be shimmed even though it's available on `npm`
  - Furthermore, the server-side piece of Rendr uses a hard-coded version of jQuery, independently of the one you pick for the client-side
  - You must define aliases in a so-and-so way ([Richard Feynmann tainted my writing style][8], so be it)
- Your templates **must** be placed into `app/templates/compiledTemplates.js`. Seriously?
- Handlebars. Backbone. jQuery. Deal with it.
- Awkward APIs like `this.app.fetch`. Isn't this meant to **minimize** clutter?

After asking [Spike Brehm] about it on Twitter, and figuring out [he's pretty much moved on][10] from Rendr, I've decided not to pursue it, and strike it out on my own.

[![taunus.png][3]][2]

Putting together a client-side MVC framework is no easy feat. I should know. I tried back in the day, when putting together this blog engine. I failed misreably back then. This time though, I made an effort to mix the best of both worlds.

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


[isomorphic taunus browserify npm-run]
