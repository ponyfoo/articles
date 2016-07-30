# 1. One Goal

Great programs are designed with **a single goal**. Whether we're talking about a CLI, desktop, mobile, web application, or even an API, doesn't matter. The principle remains the same: one goal. A program that's focused on doing a single thing has far better chances of doing it well, being reusable, and being better at it. There are quite a few good examples, such as [through][1], or [ansi-styles][2] in the Node community of packages which do exactly one thing well.

Programs don't have to be a just a few lines long in order to comply. Both [npm][3] and [connect][4] are great examples of programs which do just one thing well. The goal for `npm` is to provide a package manager which just works. It does double as a development productivity tool, but that's just a side-effect of being a well rounded package manager. On the other hand, `connect`provides a middleware layer for Node's native [`http`][5]. It enables middleware and provides a few basic ones.

![bullseye.jpg][6]

I built [campaign][7], which provides a pluggable email sending layer with a [convention over configuration][8] approach, reducing configuration bloat for a basic email sending service. It doesn't even do any email sending, other modules can already do that, and so [campaign][9] uses those for that purpose. Recently, I started putting together [`λ`][10], a tiny asynchronous flow control meant for the browser, which I'm really proud of. It currently sits at below `3kb` when minified, and it's capable of most of what I find useful in [`async`][11], while staying _10 times smaller_.

If your design has a single goal, then it will be that much easier to make it into a reusable component which you can utilize across projects, and maybe even open-source if you feel so inclined. One of the features of developing open-source software is that you force your programs into **focused things**. You or others can benefit from that reusability in other projects. Developing open-source projects also forces you to **document the API**, helping you think about the purpose of each API member.

>At any scale, do _one thing_ and do it well.

# 2. Excellent API

Let us define API interfaces as a **consumer-facing interface for a component**, regardless of _transport_. An API might be any of:

- The methods exported by a JavaScript package
- Those exposed by a module in that package
- The CLI interface to a command-line program
- REST API endpoints provided by a web application

I find that these translate quite nicely to human interaction design as well, and I find that I treat both with the same kind of respect more and more, even if I strive to meet different goals in each case. Design the interface as if consumers didn't have a clue about your code. **They really don't.** Most people won't even look at the code you write, and thus the interface should be intuitive and easy to use. Similarly, it should be documented well enough that people don't have a reason to look at your code. I'll add more thoughts on documentation in the next section.

##### Not So Hot

Consider, as an example, [Gulp, Grunt, Whatever][12], from last week, where we analyzed the trade-offs between the simplicity in Gulp, compared with the _overload-pandemonium_ in Grunt. While Grunt provides lots of functionality, it also provides many different ways to accomplish the same goal. This is not necessarily a good thing. Due to Grunt's **configuration juggernaut model**, consumers aren't able to properly separate task targets, even if they belong to entirely different workflows. Grunt is pretty well documented, but their API could use some love, and that has proved to be very damaging, as witnessed on Stack Overflow where people repeatedly ask basic questions such as how globbing works. Why `files` should be this, or that, etc.

Twitter's REST API is a remarkable example of **an API I vigorously despised** when I had to work with it a couple of years ago. I'm not familiar with recent developments, but back then you had the streaming API, the search API, and some other API, and they where abundantly inconsistent. API to API, and sometimes method to method, the response types and request parameter names didn't match. Even the status codes were inconsistent, sometimes returning `200 OK` for errors, sometimes a detailed JSON with the error, and some other times, plain text responses when an error occurred. No one C# library consistently "just worked" seamlessly with these API, and they seemed to be [at least as poorly documented][13] as the Twitter API themselves. Generally a pain to deal with.

These days, [Selenium WebDriver][14] has become my go-to [hated API of choice][15]. Also really confusing, with methods inconsistently named, and implementations which let you write tests using [`wd`] [16] suck, and are _marginally documented_, regardless of what language they're written in.

Middle-tier implementations such as Twitter API clients and Selenium drivers aren't to blame for their poor design or even documentation. Implementation suffers if the API is bad. They suffer just as much if the API is decent but the documentation is lacking or non-existent.

##### Consistency, consistency, consistency

There's lots of things you can do to provide a better API than most. Above all, I think the most important factor is naming. The way you name your public API members says a lot about the kind of experience your package is set to deliver. Is it `UserService.getUser`, `Users.get`, `User.find`? Whatever your choice, be consistent about it. API member naming consistency is _crucial_. Think about PHP, how much of it's perceived suck comes from inconsistent API naming and argument overloading?

Only provide **methods which add value.** If you don't have a good reason to provide an API method, then _don't_. It's always easier to add later on, than it is to deprecate, and then remove. On a similar note, you should try to be consistent in the way in which you take parameters. Always take them in the same order, and if you feel like you might add parameters in the future, consider a configuration object. That way, you won't break the API each time you add a new parameter, and all of them become optional!

Whenever you're adding a member to your public interface, think. _Ask yourself these questions._

- Does this member advance the purpose of my API?
- How does it add value?
- Does it fight for value with another member?
- Is it named consistently with the rest of the API?
- Are the arguments arranged in a consistent manner?
- Are the arguments future-proof?

> **The interface makes or breaks the user experience**, and this holds true when talking about a GUI, a CLI, or any other API.

# 3. [`README` Driven Development][17]

Over time, I've developed a habit for writing API documentation which is both extensive and useful, and I've found that my code gets better because of it. Here are some of the most recent packages I've distributed and extensively documented.

- [grunt-ec2](https://github.com/bevacqua/grunt-ec2 "Create, deploy to, and shutdown Amazon EC2 instances")
- [buildfirst][18]
- [campaign](https://github.com/bevacqua/campaign "Compose responsive email templates easily, fill them with models, and send them out")
- [suchjs](https://github.com/bevacqua/suchjs "Provides essential jQuery-like methods for your evergreen browser, in under 200 lines of code")
- [contra](https://github.com/bevacqua/contra "Asynchronous flow control for the browser")

In all of these cases, typing away at the documentation was a great way to think about the API design and how it could be improved. Or in the case of [buildfirst][19], how the sample code could've made better. Pointing people to your unit tests as a learning resource on how to use your library is _a sign of early onset dementia_. What works for me is briefly describing the goals of a library, and provide a complete list of API methods, arguments they take, and results they produce. Often, it's best to pair API method documentation with usage examples, to make it clear exactly what a method can be used for, and how _you_, the package author, think it should be used. That helps people use the library, and it helps you shape it.

If you think documentation is obsolete as soon as you push, **you're doing it wrong**. That may be true for print documentation, manuals, and such. This day and age, libraries are distributed under source-control, and most of us use [semantic versioning][20] for their libraries. Providing documentation alongside the code is required, but it isn't enough. **You need to actively update that documentation, at least every time you publish a new release**, to reflect the latest API changes in your package. Likewise, providing a [CHANGELOG][21] is awesome at letting people know what's going on in terms of active development around your package.

> Allowing documentation to become stale is, in effect, even worse than not providing any documentation at all. **Misleading documentation is worse than no documentation.**

There are a couple of things to keep in mind when writing great README documentation. You need to have a plan. Start with an outline, what questions does your README intend to answer? Is there going to be any documentation besides the README? _Tests don't count._ Who are you writing the documentation for? Is it clients? Fellow developers? The open-source community? What do they need to know? I like creating documents that start with the name or logo of my package. Then I'll provide a quote with a sentence or two which briefly state the goal or purpose, and afterwards I may provide a paragraph describing the module in a little more detail. Once that's out of the way I get to installation, where I describe how the module can be installed from each source that's available to consumers, package managers, straight from GitHub, and whatnot.

Lastly, I like going through each API member and documenting their name and method signature, describing the method as well as each argument it takes, and the result it produces. In all cases I love providing self-contained usage examples for each method. I try not to mix examples with multiple parts of an API too much, since people don't necessarily read the whole thing at once, they might just navigate to specific bits and pieces at a time. For those people, going through a piece of code that uses 3+ API methods starts getting hard to understand. You want usage examples to be as simple as possible. They describe how your API is used, and if they look complicated, it probably means your API is complicated, or that your documentation is poor. Your API shouldn't be complicated. **A complicated API is a bad API.** Your documentation should be elegant, poor documentation isn't pretty to look at. That gets people upset. It's also a waste of time to type out poor documentation. _Write good API usage examples!_

##### You might want to consider answering these questions

- What is this program?
- How do I get it?
- How do I use it?
- What are the methods I can use?
- Can you provide me with any usage examples?
- What license type do you use?
- How can I contribute?

# 4. Open Source It

Some of the best software I've written is open-source. That's not bragging, it's just that you have to be more careful about it. I already stated that open-source forces you to write documentation, which in turn helps you write better interfaces. But open-source doesn't stop at that. Counter-intuitively, **open-source makes your code more secure**. You can't merely distribute API secrets with an OSS piece of code, you might need to [encrypt it][22], or exclude it from your codebase altogether.

![oss.jpg][23]

Open makes you want to write [decoupled code][24], so that other people can consume it. That also means you'll be able to use it in other projects down the road, which should be incentive enough on its own to warrant thinking open-source. That's a key concept. **"Think open-source".** Even if you don't actually open-source the thing, writing something _as-if_ you were going to open-source it at some point, will help you with all of the above. You'll write code that's safer, better documented, and more focused.

**Decouple hacks.** If you're writing code for the browser, don't bake hacks that make your code work in **IE < 10** into your package. Rather, build them [into a companion file][25] you can add if necessary. This decoupling will do a couple _(touché)_ things for you. It'll help you keep the code cleaner, as you won't have ugly pieces of code lying around which don't have so much to do with the purpose of your package, but more to do with the limitations of the platform you're going to run the package on. It will keep the code smaller in those cases where you don't need to support legacy platforms. If your hacks are independent enough, you might even be able to reuse them in other projects! _Yayyy, reusable hacks._ `:rolleyes:`

> Deal in focused components. **Think open-source.**

# 5. Write Tests

Write a single test. Then another. Then progressively write enough so that you [cover all of your API][26]. Then, [keep your tests up to date][33]. Update the suite as you add new API members, or when modifying existing ones. Then update the documentation. Only when the tests pass and the documentation is updated should you allow yourself to commit again.

[![browserling.png][27]][28]

Set up automated testing. This is becoming increasingly painless. Testing modules on [Travis-CI][29] is a joke, it couldn't be any easier to set up. In a few minutes you can have your repository running tests on every push. If you're into running things on browsers, then [Testling][30] and [Sauce Labs][31] are eager to befriend you. I've been using both while working on [contra][32], and they're both awesome. Testling has been particularly useful, although it took me a while to figure out that tests on **IE < 10** were blowing up solely because I was using `should`, which defines a bunch of getters. After switching to `assert`, though, I didn't have any more problems.

> Whenever you add or change a method, update your test suite. Update your documentation to match. 

That's all the advice I have for today!


  [1]: https://github.com/dominictarr/through "dominictarr/through on GitHub"
  [2]: https://github.com/sindresorhus/ansi-styles "sindresorhus/ansi-styles on GitHub"
  [3]: https://github.com/npm/npm "npm/npm on GitHub"
  [4]: https://github.com/senchalabs/connect "senchalabs/connect on GitHub"
  [5]: http://nodejs.org/api/http.html "HTTP module documentation for Node"
  [6]: https://i.imgur.com/m4IXGsC.jpg "Have a single, specific purpose"
  [7]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
  [8]: http://en.wikipedia.org/wiki/Convention_over_configuration "Convention over Configuration on Wikipedia"
  [9]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
  [10]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
  [11]: https://github.com/caolan/async "caolan/async on GitHub"
  [12]: /2014/01/09/gulp-grunt-whatever "Gulp, Grunt, Whatever"
  [13]: http://stackoverflow.com/questions/6587176/tweetsharp-where-did-fluenttwitter-go "Where did FluentTwitter go?"
  [14]: http://docs.seleniumhq.org/projects/webdriver/ "Selenium WebDriver Browser Automation"
  [15]: http://blog.ponyfoo.com/2013/12/20/is-webdriver-as-good-as-it-gets "Is WebDriver as Good as it Gets?"
  [16]: https://github.com/admc/wd "admc/wd on GitHub"
  [17]: http://tom.preston-werner.com/2010/08/23/readme-driven-development.html "Readme Driven Development"
  [18]: https://github.com/bevacqua/buildfirst "JavaScript Application Design Code Samples"
  [19]: https://github.com/bevacqua/buildfirst "JavaScript Application Design Code Samples"
  [20]: http://semver.org/ "Semantic Versioning, or semver"
  [21]: https://github.com/bevacqua/grunt-ec2/blob/master/CHANGELOG.markdown "Sample CHANGELOG, as seen in grunt-ec2"
  [22]: https://github.com/bevacqua/buildfirst/tree/master/ch03/02_rsa-config-encryption "RSA Configuration Encryption"
  [23]: https://i.imgur.com/PhNCOO1.jpg "Open-Source Software"
  [24]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
  [25]: https://github.com/bevacqua/contra/blob/master/src/contra.shim.js "contra.shim.js in bevacqua/contra on GitHub"
  [26]: https://github.com/bevacqua/contra/blob/master/test/unit.js "Unit tests in Contra.js"
  [27]: https://i.imgur.com/J4WIv2x.png
  [28]: https://browserling.com "Visit the browserling"
  [29]: https://travis-ci.org/ "Travis-CI: Continuous Integration Platform"
  [30]: http://ci.testling.com/ "Testling: Run your browser tests on every push"
  [31]: https://saucelabs.com/ "Sauce Labs: Hassle-free Testing"
  [32]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
  [33]: https://github.com/bevacqua/contra/commits/master/test/unit.js "Commit History for unit tests in Contra.js"
