If I were to define Taunus in _"elevator pitch"_ style, I would say:

> Taunus is the logical step forward after server-side MVC web frameworks such as Rails or [ASP.NET][1] MVC. It turns server-side rendered apps in Node.js _(or io.js?)_ into single-page applications after the initial page load by hijacking link clicks, form submissions, and defining a format you can leverage for realtime communications.

Building an app in [a Server-First fashion][2] is important because then you aren't taking [a huge leap of faith in assuming][3] that your customers have a browser capable of supporting all the **bleeding edge features** your dedicated client-side application demands.

After the **initial load**, which **should be blazing fast** so that your customers are happier _(tons of [research][4] point to [this fact][5])_, you can should turn to a single page application, hijacking links, making AJAX requests that ask for the bare minimum _(view models)_ and then rendering those view models directly in the client-side.

# Why Server-First Matters

_Server-First_ matters because it's the exact opposite of [breaking the web][6]. You establish a bare minimum experience that you know most people can access, [a baseline][7], and you go from there. This baseline isn't just there for SEO purposes or to be more amicable to people turning off JavaScript.

Think of the ways in which your app is shared on the web. What other places is it rendered to? Services that crawl around it. With client-side rendering, Twitter and Facebook display a pile of garbage instead of _descriptive metadata and a thumbnail_ whenever someone links to your site. Humans might think your site is bogus and not even click on links leading to it, because the description on their Facebook feed just shows a bunch of [Mustache templates][8] and gibberish.

Search engines other than Google are **completely oblivious to your content**. Even Google is not as good at crawling client-side rendered apps as you think they are. Often times, _you also get penalized_ for not being fast enough.

**Mobile performance degrades substantially** in client-side rendered applications as opposed to those server-side rendered. Both because the connection is slower, and because the scripts you depend on to actually render your site take a long time to download. When they do, mobile devices take longer to parse them and execute them, because they're not as powerful as the Mac Book Pro you use during development.

![Demand Progress!][9]

Not doing server-side rendering might be **just as bad** as not designing a website to be responsive.

> It's _about time_ we `.shift()` _"SEO purposes and `<noscript>`"_ from our list of **excuses** for not doing server-side rendering anymore.

[1]: http://ASP.NET
[2]: /articles/server-first-apps
[3]: https://remysharp.com/2015/07/02/assumptions
[4]: http://blog.codinghorror.com/speed-still-matters/
[5]: /articles/critical-path-performance-optimization
[6]: /articles/stop-breaking-the-web
[7]: https://adactio.com/journal/9206
[8]: https://mustache.github.io/
[9]: https://i.imgur.com/EfS2ijh.png
