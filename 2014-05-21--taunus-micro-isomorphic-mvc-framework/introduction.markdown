That would set me up to write about Rendr in my book and forget about shared rendering. I figured it'd be mostly a drop-in plugin for Backbone. I was thorougly wrong. Getting started with the bare minimum viable Rendr is **nothing short of painful**. You can check it out for yourself. The [so-called "simple" example][1] in Rendr's repository on GitHub almost rendered me to tears. I get it. Rendr requires Backbone. Backbone requires jQuery. Fine, I accept those terms. Rendr takes a **"Convention over Configuration"** approach. That's awesome! I loved that back when I was involved in C# MVC application development. Well, yeah. Except Rendr isn't so much about **"Convention over Configuration"**, but more about **"Convention, deal with it"**. To the point where you must conform to multiple different rules.

* You [**must** browserify][2] your client-side code in **obscure ways**
  * [`brfs`][3] won't work at all
  * jQuery must be shimmed even though it's available on `npm`
  * Furthermore, the server-side piece of Rendr uses a hard-coded version of jQuery, independently of the one you pick for the client-side
  * You must define aliases in a so-and-so way ([Richard Feynmann tainted my writing style][4], so be it)
* Your templates **must** be placed into `app/templates/compiledTemplates.js`. Seriously?
* Handlebars. Backbone. jQuery. Deal with it.
* Awkward APIs like `this.app.fetch`
* **Thorough lack of documentation**

After asking [Spike Brehm][5] about it on Twitter, and figuring out [he's pretty much moved on][6] from Rendr, I've decided not to adopt it, and strike out on my own. I'm really looking forward to the **Building Isomorphic Apps** talk, which he'll be giving in JSConf next week.

[![taunus.png][8]][7]

[1]: https://github.com/rendrjs/rendr/tree/master/examples/00_simple
[2]: https://github.com/rendrjs/rendr/blob/master/examples/00_simple/Gruntfile.js#L66-L98
[3]: https://github.com/substack/brfs
[4]: http://www.amazon.com/Surely-Feynman-Adventures-Curious-Character/dp/0393316041
[5]: https://twitter.com/spikebrehm
[6]: https://twitter.com/spikebrehm/status/461939437585190912
[7]: https://github.com/bevacqua/taunus
[8]: https://camo.githubusercontent.com/b98a5dc441b3a71a01e2e46639ddf57737c2c721/68747470733a2f2f7261772e6769746875622e636f6d2f62657661637175612f7461756e75732f6d61737465722f7265736f75726365732f7461756e75732e706e67
