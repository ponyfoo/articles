# Pony Foo as a Blogging Engine

When I [started putting together ponyfoo two years ago][1], it was my first baby steps into the world of Node.js, coming from the .NET and C# universe. I was impressed by how little effort it took to do anything, and everything seemed to be well contained and single-purposed. I fell in love with this idea.

It was also my first jab at putting together my own MVC framework, which was a great learning experience that ultimately led me to build [Taunus][2] this year. The original MVC framework in Pony Foo was too tightly coupled to survive, but it had some nice ideas. It took a conventional approach to input validation and error messaging, didn't use a hash router _(you know, those "lovely" URLs like `/#!articles/2014`)_, and it was decent enough to get me started.

Then I started dreading it. It did client-side rendering only. It was [breaking the web][3]. It was so very slow. It failed to let web crawlers crawl and it _expected them to walk_. This is probably the thousandth time I'll say this, but the fact that Google is soon going to become a _Web Walker_ is not enough, not yet. What about Facebook, or LinkedIn _(gasp)_? They'll crawl your site too, and client-side rendering won't help you.

<blockquote class="twitter-tweet" lang="en"><p>I hereby define &quot;Web Walkers&quot; as the client-side-JavaScript-aware evolution for Web Crawlers.</p>&mdash; Nicolas Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/549938982197661696">December 30, 2014</a></blockquote>

Luckily, this year I did get around to rewriting the engine. It had gotten to a point where I didn't even want to publish articles. That's how slow it was. If I didn't want to browse my own blog, who would?

I compiled [a list of best practices and performance optimizations][4] I put together to make the site faster. Way faster. I'm glad I did that. I would've probably abandoned the blog if it wasn't for the rewrite. As part of the rewrite, I addressed several things people complained about often, such as the ability to post comments without an account, getting rid of the annoying "reading time" label that followed you around, and the hiragana bullet points.

# Pony Foo as a Blog

This year, Pony Foo saw **roughly 250k page views**, compared to the _~100k_ last year. One day in particular hit _9520_ page views, according to Google Analytics, and another hit _8848_ page views.

![pageviews.png][5]

Just like last year, here's the top ten articles according to page views on Google Analytics.

- [Gulp, Grunt, Whatever](/articles/gulp-grunt-whatever) _~33k_
- [Stop Breaking the Web](/articles/stop-breaking-the-web) _~25.6k_
- [Choose: Grunt, Gulp, or `npm run`](/articles/choose-grunt-gulp-or-npm) _~22k_
- [My First Gulp Adventure](/articles/my-first-gulp-adventure) _~17k_
- [CSS: The Good Parts](/articles/css-the-good-parts) _~14k_
- [Angle Brackets, Rifle Scopes](/articles/angle-brackets-rifle-scopes) _~11k_
- [Grunt Tips and Tricks](/articles/grunt-tips-and-tricks) _~6.7k_
- [Angle Brackets, Synergistic Directives](/articles/angle-brackets-synergistic-directives) _~4.7k_
- [Teach Yourself Node.js in 10 Steps](/articles/teach-yourself-nodejs-in-10-steps) _~4k_
- [Critical Path Performance Optimization](/articles/critical-path-performance-optimization) _~3.7k_

# Attend _all_ the Conferences!

 When I wrote last year's review I had a secret. I wanted to give talks at conferences. I didn't want to write it in that blog post, but I really wanted to do it. Problem is I had never even attended a conference, let alone give a conference talk! In March I went to my first conference ever, [JSFest SF][6]. _It was so much fun!_ I even went to a Beer.js meetup. I had never been to a meetup.

A week later I went to [JSConf UY][7]. Around that time I was asked to speak at [JSConf US][8], and a week later at [QCon NY][9]. JSConf US was exciting, to put it mildly. The location was ridiculous, that's the best word I can come up with to describe it. People were nice, food was amazing, and the event was carefully planned. We played with rockets, I gave interviews _(!)_, I had fun.

Giving my talk, I was terrified. It was a _large_ crowd, these people knew their JavaScript. Still, I [managed to survive on stage for 20 minutes][10], even though I would lie if I said thoughts like "look at all these people, what are you even doing on stage? hey, get back to the speaking part!" didn't cross my mind as I was giving the talk.

> I don't think the talk itself matters; I was most proud of not running off the stage and locking myself up in a toilet stall.

Good thing my talk was on day one, since I was oblivious of everything that was happening around me until the moment I got off stage. Then I could finally enjoy the conference!

I think the most important thing in that experience was how nice and reassuring everyone I spoke with after the talk was to me. Over time I did get better, when it comes to nervousness. At QCon there was a much smaller crowd _(larger conference, but it had like 10 different tracks)_, I had already broke the ice at JSConf so I was more relaxed, confident that I could deliver a better talk even if it was the same title.

In August I went to Zürich for the [Frontend Conference][11], and this time I gave a different talk, [Browserify All The Things][12]. This was the first time I thoroughly enjoyed the experience. At JSConf I was scared out of my mind until after my talk was over, at QCon I was a bit disappointed at the turnout, since people were spreaded over a lot of different tracks, but at FrontendConf the audience was just right, and I enjoyed giving the talk.

Turns out Switzerland is filled with really nice people, who knew? I coincided again with other speakers _(namely, [Axel][13])_, which was kind of nice. I had barely even dabbled in the local scene and I was making friends with distant community members all over the world.

Soon afterwards I went to [From The Front][14], in Italy. The conference was in a nice theater in Bologna. I gave the tooling talk again, had **stupidly tasty** food. _Bologna is renowned for its food and foodies._ When I finally got back home I decided to attend a few conferences in Buenos Aires, namely [RubyConf AR][15], [PHPConference][16], and [JSConf AR][17].

Around then I went to [CampJS][18] in Australia, which I already covered elsewhere, and gave [the Browserify talk][19] again. When I came back we put together [NodeSchool in Buenos Aires][20] for the first time, which had a great turnout.

My last conference for the year was [JSFest Oakland][21], where I gave the talk on Browserify once again. It was kind of nice, given that [JSFest SF][22] was my first conference ever and then I got to give a talk at the event a few months later. JSFest is one of my favorite conference formats, where every day there's typically a driving theme _(Browserify, Hapi, CSS, Node.js, etc)_, and then an after party of some sort _(Gin.js, Beer.js, Dance.js, etc)_. What's even better, **it lasted a whole week**!

# Reading

Unfortunately, my reading slowed down this year. Maybe it was the conferences, maybe it was the book writing, but the fact is that I read a lot less books this year than in 2013. Here's a list.

- [Zero to One](http://www.amazon.com/Zero-One-Notes-Startups-Future/dp/0804139296)
- [The Year Without Pants](http://www.amazon.com/Year-Without-Pants-WordPress-com-Future/dp/1118660633)
- [Rework](http://www.amazon.com/Rework-Jason-Fried/dp/0307463745)
- [Lean UX](http://www.amazon.com/Lean-UX-Applying-Principles-Experience/dp/1449311652)
- [Inspired](http://www.amazon.com/Inspired-Create-Products-Customers-Love/dp/0981690408)
- [High Performance Browser Networking](http://www.amazon.com/High-Performance-Browser-Networking-performance/dp/1449344763)
- [NoSQL Distilled](http://www.amazon.com/NoSQL-Distilled-Emerging-Polyglot-Persistence/dp/0321826620)
- [The Win Without Pitching Manifesto](http://www.amazon.com/Win-Without-Pitching-Manifesto/dp/1605440043)

Considering you're reading this blog, the one you should definitely read if you haven't already is [High Performance Browser Networking](http://www.amazon.com/High-Performance-Browser-Networking-performance/dp/1449344763). [Inspired](http://www.amazon.com/Inspired-Create-Products-Customers-Love/dp/0981690408) is also a great book, but it might not be as relevant. HPBN is an instant classic describing the world under the comfortable hood of HTTP and web browsers.

I failed on my mission to read about compilers and operating systems or databases, but I've already purchased a few books on those topics. Now I just have to read them!

Here's hoping 2015 brings about even more conferences to attend!

  [1]: https://github.com/ponyfoo/ponyfoo/commit/774d3da4887dc9e776144f4287977a6ca172927e "First commit to ponyfoo/ponyfoo on GitHub"
  [2]: http://taunus.bevacqua.io/ "Taunus: Micro Isomorphic MVC Engine for Node.js"
  [3]: /articles/stop-breaking-the-web "Stop Breaking the Web on Pony Foo"
  [4]: /articles/critical-path-performance-optimization "Critical Path Performance Optimization at Pony Foo"
  [5]: https://i.imgur.com/bZHmLsl.png "Screen Shot 2014-12-30 at 12.39.24.png"
  [6]: http://sf.jsfest.com/ "JSFest San Francisco March 2014"
  [7]: http://jsconf.uy/ "JSConf Uruguay"
  [8]: http://2014.jsconf.us/ "JSConf US 2014"
  [9]: https://qconnewyork.com/ny2014/schedule-2014.html "QCon NY 2014 Schedule"
  [10]: https://www.youtube.com/watch?v=Y0DCZdAruvo "Front End Ops Tooling"
  [11]: http://2014.frontendconf.ch/en/ "Frontend Conference Zürich"
  [12]: https://www.youtube.com/watch?v=uZ_1_fddWns "Browserify All The Things"
  [13]: http://twitter.com/rauschma "Axel Rauschmayer on Twitter"
  [14]: http://2014.fromthefront.it/ "From The Front 2014"
  [15]: http://rubyconfargentina.org/index_en.html
  [16]: http://2014.phpconference.com.ar/
  [17]: https://www.jsconfar.com/
  [18]: http://campjs.com/
  [19]: https://www.youtube.com/watch?v=uZ_1_fddWns "Browserify All The Things"
  [20]: http://nodeschool.io/buenosaires/ "NodeSchool BA website"
  [21]: http://oakland.jsfest.com/ "JSFest Oakland"
  [22]: http://sf.jsfest.com/ "JSFest San Francisco March 2014"
