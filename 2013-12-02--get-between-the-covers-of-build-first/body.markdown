> **TL;DR**
>
> The Build First philosophy will help you approach application building in a more disciplined way. Adopting a build-oriented mentality, designing large but maintainable JavaScript applications, and deploying from the command line are some of the key take-aways in the book.

### _Build First_ In a Pinch

As you might've read, this is a book about designing JavaScript applications in a build-oriented manner. To get a hint about the contents of Part I, on build processes, I've compiled a list with a few articles on the subject you can peek at. Part I is dedicated entirely to the build and deployment processes. You'll learn how to automate integration testing, deployments, builds, and even development. Grunt _(as a tool)_ is taught from scratch, but the concepts should stick with you even if you part ways with Grunt.

- [Deploying Node Apps to AWS Using Grunt](/2013/09/19/deploying-node-apps-to-aws-using-grunt "Deploying Node Apps to AWS Using Grunt on Pony Foo")
- [Continuous Development in Node.js](/2013/09/26/continuous-development-in-nodejs "Continuous Development in Node.js on Pony Foo")
- [Grunt Tips and Tricks](/2013/11/13/grunt-tips-and-tricks "Grunt Tips and Tricks on Pony Foo")
- [Understanding Build Processes](/2013/05/22/understanding-build-processes "Understanding Build Processes on Pony Foo")
- [Managing Code Quality in Node.js](/2013/03/22/managing-code-quality-in-nodejs)
- [Upgraded Asset Management](/2013/07/22/upgraded-asset-management "Upgraded Asset Management on Pony Foo")

Part II is dedicated to complexity management in JavaScript applications. Here, chapters cover topics such as modularity and package management, asynchronous programming styles, unit and integration testing _(again, in JavaScript)_. Solid JavaScript programming in general. I've also written a few relevant articles on the subject as well.

- [The Angular Way](/2013/08/27/the-angular-way "The Angular Way on Pony Foo")
- [Uncovering the Native DOM API](/2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API on Pony Foo")
- [Getting Over jQuery](/2013/07/09/getting-over-jquery "Getting Over jQuery on Pony Foo")
- [Fun with Native Arrays](/2013/11/19/fun-with-native-arrays "Fun with Native Arrays on Pony Foo")
- [The Web Wars](/2013/05/13/the-web-wars "The Web Wars on Pony Foo")
- [Taming Asynchronous JavaScript](/2013/05/08/taming-asynchronous-javascript "Taming Asynchronous JavaScript on Pony Foo")
- [Pragmatic Unit Testing in JavaScript](/2013/03/28/pragmatic-unit-testing-in-javascript "Pragmatic Unit Testing in JavaScript on Pony Foo")
- [Information Hiding in JavaScript](/2013/02/21/information-hiding-in-javascript "Information Hiding in JavaScript on Pony Foo")

### _Build First_ In a Nutshell

> **Build First** is all about keeping a maintainable, modular code-base that can be continuously delivered in a consistent manner, from the very beginning.

The importance of [#buildfirst](https://twitter.com/#buildfirst "Hashtag #buildfirst on Twitter") stems from the fact that plugging a build process into an existing project is **so hard**. Some things just have to be baked into a project starting with its inception, if they are to succeed. Have you tried refactoring a website written without a client-side MVC framework, entirely dependant on jQuery, into using Angular.js? It's **an extremely hard thing to do**. It's even harder to do it right, not leaving the project as a half-baked zombie which is _really just jQuery with cream on top_, turning it into the awful kind of dessert you most definitely don't want to be eating.

The absence of build processes presents _similar complications_. If we abstain from implementing an automated build and deployment process, **we might be putting our business at risk**. I can't really come up with _a reason not to_ implement one, other than [dramatic attempts to attain _"gold sinking powerhouse"_ status](http://bevacqua.io/bf/knight "How to lose $172,222 a second for 45 minutes"). Regrettably, _not all_ of us are [playing an RPG](http://www.mine-control.com/zack/uoecon/uoecon.html "The In-game Economics of Ultima Online") with our business, some of us can't have the luxury of that kind of risk.

You are probably going to **need a build process** at some point, regardless. Even if you argue your way out of automated, one step deployments, you are still going to need to do simply things such as bundling and minifying your static assets, or more advanced stuff like _cache busting_, appending hashes to your filenames (e.g: `/js/ad161513.all.js`) so that you can set far-future `Expires` headers.

> Aren't you just tired of adding icons to that spritesheet and updating the relevant CSS all by yourself? You [should automate those things!](/2013/10/16/spritesheets-grunt-and-you "Spritesheets, Grunt, and You")

The other side to _Build First_ is concerned with actually building out the application. Not everything is in the process, obviously. A modular architecture with good separation of concerns is also key in keeping your code testable, and **easily understandable** across your team. Together, we'll explore different frameworks that can help us build a cleanly structured application, while staying away from _tightly coupled_, jQuery-intensive code. Dependency resolution, asynchronous or otherwise, also gets covered.

Asynchronous flows might represent a problem if you're not that well versed in JavaScript programming, other than your first-level callback. I invite you to explore different approaches to tackle this problem, such as Promises, the [async](https://github.com/caolan/async "caolan/async on GitHub") control flow library, and [ES6 Harmony generator functions](http://wiki.ecmascript.org/doku.php?id=harmony:generators "harmony:generators on ES6 wiki").

### Reference Links

In the book there are various places where I reference articles _other authors_ have written, and this presented two problems. Links are hard to type if you are holding in your hands a print copy of a book. I know because I've been there. I was also worried about link rot (links no longer working, [_because Internet_](http://www.theatlantic.com/technology/archive/2013/11/english-has-a-new-preposition-because-internet/281601/ "English has a new proposition, because Internet")). These are the reasons why [bevacqua.io](http://bevacqua.io "bevacqua.io is my personal website"), the website that hosts the promotional landing page for the book, was originally conceived. It started out with a small JSON file with all the links referenced in the book, and a short identifier which you could actually type into the browser by hand.

For example, one of the first links that appear on the book is [bevacqua.io/bf/knight](http://bevacqua.io/bf/knight "Knight Capital's Downfall"), which is way more convenient to type by hand than, say, [_pythonsweetness.tumblr.com/post/64740079543/how-to-lose-172-222-a-second-for-45-minutes_](http://pythonsweetness.tumblr.com/post/64740079543/how-to-lose-172-222-a-second-for-45-minutes "How to lose $172,222 a second for 45 minutes"). Later on, I figured I could put all of that **relevant material regarding topics that are at the core of my book** to good use. So, the site slowly started progressing, and with a few descriptions and icons, [the #buildfirst Resources page](http://bevacqua.io/buildfirst/resources "Reference Links for the JavaScript Application Design book") was born.

### Companion Code

**Good companion code makes a world of difference** when it comes to technical books that are supposed to be teaching you how to write better code. I've spent enough time reading books to know just how _important good quality code samples_ can be. Quality code samples should be properly documented, [available online on a GitHub repository](https://github.com/bevacqua/buildfirst "Accompanying code samples and snippets for the JavaScript Application Design book") which is periodically updated, and sufficiently structured so it isn't unthinkable to look up a particular snippet of code.

I'm really happy with the effort I've poured into developing documentation for these code samples. For instance, [this code sample, showing how to automate database related tasks](https://github.com/bevacqua/buildfirst/tree/master/ch02/09_mysql-tasks "MySQL Database Tasks") is barely mentioned in the book, because the content isn't really relevant to the context where it is mentioned in. Still, the code sample fills the content gap, connecting the sample with what's discussed in the book, and highlighting aspects that are not discussed elsewhere.

Here's a few other ones I had fun coding and documenting:

- [Encrypt your configuration with RSA keys](https://github.com/bevacqua/buildfirst/tree/master/ch03/02_rsa-config-encryption "RSA Config Encryption Code Sample"), easter egg included!
- [_Build First_, Deploy, Build Again](https://github.com/buildfirst/heroku-grunt "Grunt-aware Heroku Deployments")
- [Deploying to Amazon EC2](https://github.com/bevacqua/buildfirst/tree/master/ch04/07_aws-deployments "Deploying to Amazon Web Services")

# Trivia you probably don't care about

This is **actually the first thing** I wrote about, because it's _what drove me to write the book_. I then moved it to the bottom, because you would've probably skipped it anyways. If you're enthusiastic about this book, you might find these facts to be a tad more interesting.

When I originally pitched the book to [Manning](http://manning.com/ "Manning Publications Co."), I **had a book about Node.js in mind**. Over the months, the focus changed to code quality, development workflow, and build process optimizations. Here is an excerpt taken right from _the proposal I sent to Manning_, listing of topics I was **entertaining back in May**.

> I'd like to write about how to set up an effective development environment, and how to architect a coherent application in JavaScript, both for the front-end and the back-end.
>
> ### Topics the book might cover would include:
>
> - Node fundamentals, common patterns, I might want to spend 2-3 chapters in introducing some of the lesser known aspects of JS development
> - Web Application Architecture, I would separate this in two, back-end and front-end, talk about architecture in Node, architecture in the back end, how to keep them loosely coupled, and how to let them communicate. I wouldn't stay high level but give actual practical advice or examples, such as efficiently using Express and AngularJS
> - Environments, how to set up an environment that favors productivity, mastering everyday tools, such as text editors, consoles, and git
> - Testing, a quick glance over the different types of testing, tools, and how to automate it
> - The Build Process, why one step builds are important, how everything comes together when building, different tools you can use

Their publisher replied to my proposal the next day, and we began dancing around the proposal, putting together a table of contents. As time went on, I progressively realized I literally had **no freaking idea** how to write a book. That's where Manning's editors came in, they've been great so far, I've learned _a lot_ about the process and book writing itself, and I believe I'm becoming better at pushing keys **in the appropriate order** on my keyboard.

The book was originally named _Build-First JavaScript Applications_, which was pretty confusing. After a call where we first discussed the topics I'd be writing about in the book, I put together a _formal proposal_, where the description still roughly matches the book in its current form.

> ### Book Description
>
> Build-First JavaScript Applications has a two-headed mission. It will teach the reader how to tackle application development using a well thought-out build process from the get-go. On a second level, readers will gain insight into applying these concepts to JavaScript and Node applications.
>
> The book sets the bar with a section about **build processes**. The reader will learn what a build process is, and why he should be interested in implementing one. He’ll be introduced to tasks, environments and deployments, and learn how to keep a productive development environment.
>
> A second section of the book focuses on **managing complexity in JavaScript**, introducing topics such as asynchronous programming and dependency management. We’ll look at keeping our code testable, and learn strategies for testing our modules.
>
> These concepts are put to test in the third section, **application design**. This section will describe and design a moderately complex application, and guide the reader through its development. Eventually, we will have applied all the knowledge gathered in the previous chapters into a tangible application that can be accessed online, and the reader will have embraced that knowledge.

The table of contents has been reworked quite a bit by now. Node has been relayed to the background, mostly regarded as a dependency for Grunt. However, parts of the book which require a back-end use Node for that. Deployments are also explained using Node. Testing, however, is mostly dedicated to front-end efforts.

# Etcetera

There's a lot more coming, and I couldn't be more thrilled to see this project moving forward! I'm sure I'll write an update about the book as I make progress through parts II and III in the book. I'm really happy with the feedback I've read so far on Twitter, HN, et al. I hope the feedback keeps pouring in!

If you want to comment on anything related to this project, you can email me at [buildfirst@bevacqua.io](mailto:buildfirst@bevacqua.io "Send me an email about #buildfirst"), or just drop a comment here.
