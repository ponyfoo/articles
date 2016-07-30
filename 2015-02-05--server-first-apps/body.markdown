# Angular, Ember, and React

I've [already summed up my thoughts on Angular][1]. The more I think of it, the more convinced I am that Angular is _the **Bootstrap** of JavaScript_. It's a great way of prototyping an application or building a backend service, but there's _plenty of reasons_ why you shouldn't be building a customer-facing application with it.

> I hope 2015 is the year where we take out _"dedicated client-side rendering"_ like Angular's from our **metaphorical best practices grab-bags**. React and Ember are doing a good job of bringing people to their senses when it comes to one-sided rendering.

I'm not sure how the initiative to move Ember to shared-rendering, **FastBoot** will work, but if it hijacks `<form>` submissions and generally does the right thing with those _(both before and after JavaScript gets executed on the client)_, then I'll be quite sold on the idea. I'm glad to see that Tom Dale seems to have come around from clamouring that ["Progressive Enhancement is Dead"][12], but I still think developing client-first applications and then rebuilding what should have been the original HTML is **just backwards**.

React is more of a "shared-rendering" native citizen, which makes it friendlier when it comes to progressive enhancement. It's shared-rendering capabilities used to be mostly an option for Node.js developers, but [Facebook recently revealed `react-native`][2] as a way to write native Android and iOS applications on React, making it even more appealing as it now enables cross-platform development, a lot like how [Google shares code][14] across platforms using [j2objc][13].

# Progressive Enhancement

**The web is not native**, though. React and Ember don't hinder our ability to develop a progressively enhanced application, but they don't exactly encourage it either. I'd call them — along with Angular — **client-first frameworks**. Client-first doesn't encourage progressive enhancement. Quite the contrary, client-first actively discourages progressive. That's a real problem.

These frameworks are nice and definitely boost our productivity, but we should never stop thinking about building applications in such a way that they'll actually work well _(not just **render** well)_ for people [on slow or intermittent networks][3].

I think these last few months we did a good job of thinking critically about whether the Angular way is the right way. I'm convinced that the web would be a far better place if we developed most applications in a **content-first** manner.

Principles of a progressively enhanced user experience should be commonplace by now. You build an application on pure HTML and CSS, in such a way that it's able to deliver most of your core experience\* right off the bat. This is important because sometimes JavaScript may take a few seconds to download. That's why I made the point about using `<form>` elements to allow users to interact with the site, it enables more parts of the core experience.

> It's insane, what we are doing. **We are deferring JavaScript and loading it asynchronously and then depending on it to deliver our core experience?**

_\* Unless you're a **realtime video-conferencing service** (or anything canvas-based), but even then you could take a progressive approach, where you inline the absolutely necessary JavaScript to enable the video-calling functionality, and defer the rest. Much like you'd do [when deferring non-critical CSS][3]._

Suppose you have a TODO list. Checking items off would just be a matter of clicking on them in a client-first application, then the changes would be persisted in the background. In contrast, a server-first approach probably would've had a `<form>` with the TODO list and some sort of <kbd>Submit</kbd> button. Right? But what if each TODO item was a `<form>`? What if each of them was a `<button>` within its own `<form>`? Then you could have almost the same functionality as people have come to expect from client-first applications, but in an entirely progressive way!

The HTML would look like this, except with proper form `action`s, `href`s, and CSS classes for styling.

```html
<ul>
  <li><form><button>This is an option</button></form></li>
  <li><form><button>This is another option</button></form></li>
  <li><form><button>This is yet another option</button></form></li>
</ul>
```

It could look like the screenshot shown below, which comes from [a product I'm building][5], and doesn't involve any client-side JavaScript just yet.

[![stompflow.png][4]][5]

Not any client-side JavaScript yet? **_That_ can't be good!** You must be thinking. Turns out _developing applications like its the year 2002_ is super productive — you don't have to spend any time carefully picking a delightful animated loader gif, or debating with your staff about **what's the best way to do data-binding**.

That's the main argument against not using your fancy frameworks, right? But they're _so productive!_ Well, using HTML and **PRG** is fast, too! You just forgot they even existed.

Sure, the **PRG** pattern is "slow", and client-first is perceptively faster — _but guess what?_ Upgrading an **HTML-PRG** experience into an AJAX experience is a matter of writing a few lines of code, if it's done right. From there, turning the experience into a real-time experience is the only challenge left. And, honestly? That's just a matter of listening for the appropriate events and responding to them!

# A Server-First Web

[Taunus][6] is a server-first shared-rendering MVC engine that prioritizes content and encourages progressive enhancement. It's what [this blog][9] runs on top of, it's what the [documentation mini-site][8] runs on top of, and it's what I'm using in Stompflow — pictured in the screenshot above, [but yet to be released][7].

Being server-first has its perks as well, just like client-first does. For instance, I [maintain email templates][10] using the **same templating engine** that I use on web views. Being server-first also means that I don't have to worry about **state vs behavior** as much, because I have the same views on both sides and I can re-render them at any time in the client-side.

Being server-first means above all that the application will work well no matter what. It'll work well if client-side JavaScript takes a few seconds to execute, because it was designed to do so. It'll continue to do well after client-side JavaScript lands. You can take advantage of the conventionality of HTML forms and hijack form submissions just as conventionally, making the form submission via AJAX and then handling the response by redirecting or rendering some data.

> You no longer fight against the disgrace IE8 unleashed onto you, but _embrace it_. You now **fight against the idea that you should cram every single new feature onto users on old browsers**.

Server-first is _non-commitment_ to client-side technologies. With [Taunus][11] you don't _have_ to pick a client-side data-binding library, but you can choose to do so. We already agree that it's easier to deal with vanilla components than jQuery plugins, or Angular directives, or React components, or even Web Components.

So, why not use a framework that empowers you to work this way? Using modular components that aren't tied to the framework itself, which is more of a glorified "stay-out-of-the-way router" that dictates you how to do shared rendering by convention.

Fine, [Taunus][11] is tied to a server-side technology: Node.js _(or io.js!)_

That's not an issue for React, or Ember. Not even for Angular.js.

But you know what? _That's a good thing._

Because you are forced to **commit to a server-side technology _first_**.

[1]: /articles/stop-breaking-the-web "Stop Breaking the Web"
[2]: https://www.youtube.com/watch?v=KVZ-P-ZI6W4 "'Introducing React Native' talk at ReactConf"
[3]: http://ponyfoo.com/articles/critical-path-performance-optimization "Critical Path Performance Optimization at Pony Foo"
[4]: https://i.imgur.com/NqHl1zm.png
[5]: http://blog.stompflow.com/articles/iterative-prototyping-for-the-web "Iterative Prototyping for the Web"
[6]: http://taunus.bevacqua.io/ "Taunus: Micro Isomorphic MVC Engine for Node.js"
[7]: http://www.stompflow.com "Stompflow: Hassle-free Project Management"
[8]: https://github.com/taunus/taunus.bevacqua.io "taunus.bevacqua.io source code on GitHub"
[9]: https://github.com/ponyfoo/ponyfoo "ponyfoo.com source code on GitHub"
[10]: https://github.com/ponyfoo/ponyfoo/blob/master/views/server/emails/article-published.jade "This template will be running hot when the article gets published!"
[11]: https://github.com/taunus/taunus "taunus on GitHub"
[12]: http://tomdale.net/2013/09/progressive-enhancement-is-dead/ "Progressive Enhancement is Dead"
[13]: https://github.com/google/j2objc "google/j2objc on GitHub"
[14]: http://arstechnica.com/information-technology/2014/11/how-google-inbox-shares-70-of-its-code-across-android-ios-and-the-web/ "How Google Inbox shares 70% of its code across Android, iOS, and the Web"
