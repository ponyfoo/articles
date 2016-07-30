# Measuring Performance

There are plenty of ways in which we can measure performance. Measuring performance is important when we're **trying to improve performance** on our sites. You need to know where you are before you know where you can go next. Let's go over a few ways in which we can measure performance.

## Chrome DevTools Audits

In Chrome, open up your DevTools _(_<kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>i</kbd>_)_, and you'll notice that there's an _Audits_ tab that you've probably never used until today. You can use that to run a quick performance audit on the website you're currently visit. Feel free to try it on Pony Foo!

![DevTools Audit in action][1]

We are then met with a series of _"tests"_ that the site has failed to pass. These tests _-- or rules --_ are **one of the easiest ways** in which you can figure out that something is amiss in your current setup. In the _screenshot_ displayed above, we can find the following **actionable** items:

* There's <mark>243</mark> unused CSS rules we could safely remove
* Images are sometimes larger than they need to be
* Cookie size is too large _(probably due to [analytics trackers][2], though)_

Audits is a great web to get almost instant feedback about a site, but it might not be the deepest dive into how your app is performing, there's a limited number of rules that they use to check how your site is doing, and there's no insight into the **effectiveness of caches** we've set up on repeated views _(cache priming)_.

Let's try another tool, this time it's a web service.

## PageSpeed Insights

Despite a recent announcement that confused everyone into thinking PageSpeed is going to [be shut down][3] -- _it [is not][4]_ -- PageSpeed is an awesome tool in the web performance caretaker's arsenal. PageSpeed is a web service [that you can access online][5] and once you've entered the URL to your site, you'll get back a bunch of metrics and action items _(things you should do to improve the performance of an app)_.

If your site isn't publicly accessible yet, because it's a stealth startup of some sort, or otherwise hosted only in your local environment, you can still use PageSpeed. Just start a web server that listens on port `$PORT` and then execute the next couple of lines in Bash.

```bash
npm install localtunnel -g
lt --port $PORT
```

The [`localtunnel`][6] package will create a secure tunnel between your application and the open Internet, and it'll give you back a URL that's proxied back to your local port. You can then just paste that into PageSpeed, and you'll be able to analyze the site without any hosting platforms getting involved. Of course, when performance fine-tuning is of the essence, you should **_-- at least --_** be relying on staging environments, in order to get better estimates and insights into your metrics.

[![PageSpeed Insights into ponyfoo.com][7]][5]

PageSpeed approaches performance measurement in a few different ways for your site. First off, you'll notice that PageSpeed provides you with measurements categorized as "mobile" and "desktop" right off the bat. This comes in handy because they even provide you with some very basic UX advice _(e.g "use larger tap targets")_.

They approach the analysis with a few more advanced techniques too. For example, the desktop version of their test indicates that you should be [inlining critical CSS][8]. This is something we seldom hear about nowadays, and tooling development around the technique has mostly stalled, but it's something that greatly improves the performance of your site _-- if you can realistically get away with it!_

Next up is yet another performance analysis as a service.

## WebPageTest

[WebPageTest][9] _(or, WPT for short)_ is unquestionably the most detailed piece of web performance analysis out there, and it's also gracefully sponsored by Google. They'll give you insight into every request -- down to the TCP level of the connection. They have several different views into the same data, which makes it all the more useful.

Before I go into each view in detail, I'll drop in something Christian Heilmann mentioned to me the other day, which I don't see highlighted often enough. If you want to get accurate results from WPT, **don't just run one test** from one location, one browser, and one connection type. Try multiple configurations, figure out what the experience feels like for a mobile user on <mark>a spotty 3G connection using the Android Browser</mark>, and not just Chrome on broadband.

### Report Overview

First off, WPT presents us with an overview of the report. Here we get a glimpse into the most important performance indicators for our site. For instance, we learn that **we have a SpeedIndex of 1203 on first load**, but that _it goes down to **799**_ after the cache is primed. The <mark>SpeedIndex is an overall numeric score that tells us how quickly the visible page content is getting painted</mark> _-- and the lower it is, the better_.

![Report overview sample screenshot in a test of Pony Foo over WebPageTest][10]

Learning how the site behaves in first load and for a second time is usually revealing. If the difference isn't something like 60%+ faster load, chances are we're missing out on caching opportunities.

### Waterfall View

Here, you get to see how requests block each other and what things you should be loading asynchronously to get the best performance gains. The waterfall view is way easier to explain visually, as most reports are, than trying to put them into words. Here's how it looks like for Pony Foo on first view.

![First view waterfall view][11]

As you can see from the data explosion in this graph, it takes _153ms_ for the HTML to finish downloading, after which we download styles, images, and fonts. Rendering starts around the _1s_ mark, and most of the JavaScript only starts downloading after the _2s_ mark. At that point however, the site was already visible. Of course, we're speaking about Chrome on broadband, so anything less than blazing fast should be unsettling.

The general approach should be that once actual content finishes loading, then we can add improvements via JavaScript, more images, and whatnot. **Ordering resource loading properly will yield some of the better gains in first page load**, not to mention cutting down on the images that are loaded early in the waterfall graph.

How does Pony Foo do in repeat views, after the cache is primed with some resources?

![Repeat view waterfall view][12]

Ah, that's _much better_. As you can see we still have some <mark>requests for analytics and advertisement resources</mark>, but for the most part the CPU is smoking at **100% utilization** and trying to keep up with rendering. The **total load time is halved** from around _3s_ to around _1.6s_, which is kind of what we should expect when aggressively caching images, fonts, JavaScript and other static assets. The request count, _one of the easier-to-track metrics_, has gone from 29 to a measly 6, too.

### Analyzing Request Details

Another view into the same data is the _request details_, where we're presented with the same list of requests, but this time we're getting individual stats for each of them. While not as useful as the rich waterfall view, you can still get something out of this one. Particularly, it'll become evident where you should be cutting down if there's too many images or requests being made against third-party domains through one of your analytics providers, advertisements, or third-party libraries that end up generating tons of traffic.

![Request details in WebPageTest report][13]

It also becomes easy to tell if we are prioritizing content in the correct order, downloading assets as they need to be presented to the user, leaving large images that are out of the viewport in a low priority and getting text in front of "eyeballs", as [Twitter would call human beings][14].

### Optimization Checklist

Just like we've seen in the PageSpeed section, where you get _a score from 0 to 100_ in mobile and another one for desktop, WPT also **provides you with grades**. Instead of discriminating between arbitrary screen sizes, WPT grades your performance in a few different areas. _Here's how Pony Foo scored._

![Pony Foo grades at WebPageTest][15]

Not that bad. Pony Foo doesn't really need a CDN as that'd be overkill for a homebrew blog that doesn't even pay for itself, so that one's out of the picture! When it comes to caching static content, I was surprised the first time I saw one of these reports cards. Over time I learned that pretty much whenever third party services are involved, you're going to get low scores when it comes to caching.

To shed some more light on the issue, WPT also provides us with a detailed view where you can see how each request impacts any particular grade. As you can see in the screenshot below, most of the content that isn't being cached _(or is being cached for a short period of time)_ comes from third-party sites.

![Optimization grades down to the request level][16]

In case you didn't have enough reports for the first section of this article, there's one more valuable piece of WebPageTest reporting that you should know about.

### Filmstrip View

To get a visual analysis of your site, simply choose the _"Visual Comparison"_ tab in the landing page of WPT, and then enter the page you want to run your tests on.

![The WebPageTest dashboard][17]

The filmstrip view is exactly what it sounds like. WPT records a video of your website as it's loading, and then you get to see how the page load progresses, visually. This turns out to be very useful in detecting, and **eventually preventing** flashes of invisible text _(FOIT)_. These can happen when we load a custom web font synchronously, essentially blocking everything else on expensive font downloads. A common **work-around is to use _a web-safe font_ while the page loads**, and then asynchronously load the custom font. When the custom font loads, we just apply a class name to the document, and overwrite the `font-family`. This way we translate the FOIT into a FOUT _(flash of unstyled text)_. It might not be as _"correct"_, but is definitely <mark>better for humans</mark> visiting your site!

![Filmstrip view of Pony Foo][18]

As you can see, WPT is about as detailed as it gets. The benefits don't come without drawbacks, though. WPT is quite slow, sometimes taking _as long as **20s**_ to run a test. Then there's the waiting time. WPT initially places your request to analyze a site on a queue, and you have to wait for a spot to become available before your test runs. To get around that, you might pick a different WPT instance that's not as busy, and you might be able to run your test sooner.

# Automating Measurements _(and Budgeting)_

At this point in my presentation I usually make a pause, and point out how everything we've been discussing so far is of **a _"one-of"_ nature**. I don't want attendees going home, auditing their site with DevTools once, or checking out the WPT service and tabbing around the different reports, and then closing the tab, maybe even implementing a fix or two, never again to see how their application is doing in terms of performance.

Performance measurement takes dedication. This isn't something you do once and don't need to worry about anymore. Performance should be built into everything you think about and do, even the application's interface should be [_designed_ with performance in mind][19].

In order for performance measurements to be effective, we must integrate them into our build and deployment processes. There's plenty of tools that we can use to automate the process of measuring performance. Before we go over them, let's turn our attention to budgets. When it comes to keeping track of performance in build processes, we also need to determine a performance budget. Think of budgets as a virtual <mark>_"you must be this performant to ride the production servers"_</mark> tolls.

Combining measurements on every build with strict performance budgets means that not only you get to identify how every build affects application performance, but you also get to impede deployments should they not meet the minimum performance requirements demanded by the build.

Let's go over a few tools we can use to automate the measurements, first.

## Automating PageSpeed with `psi`

As you can infer from their name, [`psi`][20] is an automated gateway into PageSpeed Insights. It can be used in a variety of ways. There's the [Grunt plugin][21], an example on [how to use it with Gulp][22], a [command-line][23] interface, and a [programmatic API][24]. Effectively, that means you can use `psi` with virtually [any build system you're comfortable with][25].

[![Running psi through various build systems][26]][20]

As you can see, [`psi`][20] allows you to run any site through their system, and you'll get a nice report in your terminal, _or a JSON response if you're using the programmatic API_. You can provide `psi` with a `threshold` option, determining the lowest possible score that would pass the test. If the `threshold` isn't met, then the build will fail, and your application wouldn't be deployed if you were using some sort of continuous deployment mechanism.

> That's a great way of enforcing performance!

## WebPageTest Automation

WPT can also be automated through an npm package, [`webpagetest-api`][27]. The process here is a bit more involved, because you still need to wait in a queue before you can get any results back. You could write a wrapper around [`webpagetest-api`][27] that did the waiting on your behalf, but the package itself isn't very well prepared to do the waiting on its own. Once you get the results back, you'll **notice the insane level of detail that WPT churns out**, making it an invaluable tool regardless of it being a bit clunky to execute the tests.

[![Dealing with the WPT API programmatically][28]][27]

Just remember that you should be running multiple tests through WPT in order to ensure correctness in the results it produces. Especially, try and test your application from different locations and connection types!

## As an alternative, use YSlow

If you don't have enough with the other two services, you could use [`grunt-yslow`][29] as well. This is one of the oldest performance reporting tools in existance, and it came from Yahoo. The problem with it being old is that it doesn't have some of the latest recommendations that we can observe in Google tooling. That being said it's one of the few tools that you can run both as a browser extension and directly in your command-line _(or using Grunt)_, so there's some value to it as well.

[![Using YSlow in the browser or a terminal window is all the same][30]][29]

Note how YSlow also gives you a grade, an overall performance score, and quite a few rules for you to go over and see whether your application is going to perform well in the real world.

## Budgeting and `grunt-perfbudget`

We've talked about performance budgets, but what exactly is that you should be measuring, tracking, and enforcing? There's a few different kinds of metrics that you could leverage.

* _Milestones_, such as "time to first tweet", load time, or _-- in broad terms --_ **how long content takes to load**
* _SpeedIndex_, the indicator generated by WPT that tells you **how quickly the visual load of the page is completed**
* _Quantity-based metrics_, like **request count, image weight, and similarly _easy-to-track_ data points**
* _Rule based metrics_, one of the simplest ways to measure performance, by **keeping track of the scores produced by YSlow, WPT, or PageSpeed**

Using the packages we've mentioned so far you can do all of these and more, but if you're looking for a simpler implementation you should look no further than [`grunt-perfbudget`][31]. This Grunt task has tons of options allowing you to tweak exactly what metrics are important to your application. It leverages WPT to tell wether the performance budget requirements are met or not.

[![Using and configuring grunt-perfbudget][32]][31]

Note that the task might take a while, due to the queuing in WPT. You can however select the kind of connections and locations you want to be testing from, so that also comes in very handy when using [`grunt-perfbudget`][31]!

![How to implement any of this?][65]

The [second part of this article][82] is devoted to finding fixes for the performance issues you'll uncover when measuring performance. You may also refer to [`perfschool`][66] and [JavaScript Application Design][67] if you're interested in getting some hands of experience with measurements, budgets, and performance optimizations.

## DIY Workshop: `perfschool`

The workshopper runs entirely in the command-line, guiding you through a bunch of different situations where you'll need to create secure encrypted tunnels to expose sites in your local environment to services like PageSpeed, you'll learn how to optimize and shrink images, and how to enforce performance budgets. Meanwhile, I try to amuse you with cat pictures rendered directly to your terminal and things like that.

[![perfschool workshopper&#x2019;s terminal][68]][66]

## JavaScript Application Design: A Build First Approach

This book is split into two parts. In the first one you'll find everything about build tasks and automation. You'll learn how to optimize your application for development flows and for releases, [optimizing performance in the application][69] as we've been discussing in the article so far. You'll also learn about picking the right build tool, development flows, environment configuration, continuos integration, continuous deployments, and hosting your apps on Heroku or Amazon Web Services.

The second part of the book is dedicated to application design, covering everything from developing code in small modules and the different alternatives to accomplish that, staying away from callback hell and understanding `this`, scoping, and similar quirks of the language. It also has chapters dedicated to the MVC pattern, one for all kinds of testing techniques for both server-side and client-side JavaScript, and another one devoted to thoughtful REST API design.

When you're done with the chapters, there's also a series of appendices on Node.js, an introduction to Grunt, how tho choose the right build tool, and on JavaScript code quality.

> [![Book cover illustration][71]][70]
> 
> _"Enjoy the ride through the process of improving your development workflow" -- Addy Osmani, Google Developer Advocate_

You can get the book from [Amazon][70], the [publisher's website][67], or in choice physical bookstores. There's also [free code samples on GitHub][72] and a couple of chapters are publicly available on the publisher's site too.

## Further Reading

* [Fixing Performance in the Web Stack][82]
* [Critical Path Performance Optimization at Pony Foo][8]
* [Stop Breaking the Web][73]
* [On The Verge][74]
* [Client-side MVC's Major Bug][75]
* [Choosing Performance][76]

[1]: https://i.imgur.com/aQeEys0.png
[2]: http://blog.lmorchard.com/2015/07/22/the-verge-web-sucks/ "The Verge's Web Sucks"
[3]: https://developers.google.com/speed/pagespeed/service/Deprecation "Turndown information for PageSpeed Service"
[4]: https://news.ycombinator.com/item?id=9500195 "We're only deprecating PageSpeed Service"
[5]: https://developers.google.com/speed/pagespeed/insights/ "PageSpeed Insights"
[6]: https://github.com/defunctzombie/localtunnel "defunctzombie/localtunnel on GitHub"
[7]: https://i.imgur.com/w0hO5MP.png
[8]: /articles/critical-path-performance-optimization "Critical Path Performance Optimization at Pony Foo"
[9]: http://www.webpagetest.org/ "WebPageTest"
[10]: https://i.imgur.com/mjik5WK.png
[11]: https://i.imgur.com/LgFMmRj.png
[12]: https://i.imgur.com/C1avf9e.png
[13]: https://i.imgur.com/s8Ey7Ly.png
[14]: http://techcrunch.com/2015/02/05/twitter-confirms-new-google-firehose-deal-to-distribute-traffic-to-logged-out-users/ "Twitter Confirms Google Firehose Deal To Target Logged Out Users"
[15]: https://i.imgur.com/GvXqSfB.png
[16]: https://i.imgur.com/RyCBNgH.png
[17]: https://i.imgur.com/a7ORyGa.png
[18]: https://i.imgur.com/TGz0yfr.png
[19]: http://shop.oreilly.com/product/0636920033578.do "Designing for Performance by Lara Hogan"
[20]: https://github.com/addyosmani/psi "addyosmani/psi on GitHub"
[21]: https://github.com/jrcryer/grunt-pagespeed "jrcryer/grunt-pagespeed on GitHub"
[22]: https://github.com/addyosmani/psi-gulp-sample "addyosmani/psi-gulp-sample on GitHub"
[23]: https://github.com/addyosmani/psi#cli "CLI to PageSpeed Insights API"
[24]: https://github.com/addyosmani/psi#api "API for PageSpeed Insights"
[25]: /articles/gulp-grunt-whatever "Gulp, Grunt, Whatever on Pony Foo"
[26]: https://i.imgur.com/Bth1KNp.png
[27]: https://github.com/marcelduran/webpagetest-api "marcelduran/webpagetest-api on GitHub"
[28]: https://i.imgur.com/zejyYBV.png
[29]: https://github.com/andyshora/grunt-yslow "andyshora/grunt-yslow on GitHub"
[30]: https://i.imgur.com/s182eSA.png
[31]: https://github.com/tkadlec/grunt-perfbudget "tkadlec/grunt-perfbudget on GitHub"
[32]: https://i.imgur.com/TssYhZ7.png
[33]: https://developers.google.com/speed/articles/tcp_initcwnd_paper.pdf "An Argument for Increasing TCP’s Initial Congestion Window"
[34]: http://www.cdnplanet.com/blog/tune-tcp-initcwnd-for-optimum-performance/#change-initcwnd "Tuning initcwnd for optimum performance"
[35]: http://nginx.org/en/docs/http/ngx_http_spdy_module.html "nginx module: ngx_http_spdy_module"
[36]: https://www.nginx.com/blog/how-nginx-plans-to-support-http2/ "How NGINX Plans to Support HTTP/2"
[37]: /articles/server-first-apps "Server-First Apps are a Good Idea"
[38]: https://github.com/rendrjs/rendr "rendrjs/rendr on GitHub"
[39]: https://github.com/rendrjs/rendr/graphs/contributors "Activity graph for Rendr on GitHub"
[40]: http://facebook.github.io/react/ "Facebook React"
[41]: https://github.com/addyosmani/critical "addyosmani/critical on GitHub"
[42]: https://github.com/pocketjoso/penthouse "pocketjoso/penthouse on GitHub"
[43]: https://i.imgur.com/hrqmyge.png
[44]: https://github.com/giakki/uncss "giakki/uncss on GitHub"
[45]: https://github.com/addyosmani/grunt-uncss "addyosmani/grunt-uncss on GitHub"
[46]: https://github.com/ben-eb/gulp-uncss "ben-eb/gulp-uncss on GitHub"
[47]: https://github.com/sindresorhus/broccoli-uncss "sindresorhus/broccoli-uncss on GitHub"
[48]: https://github.com/giakki/uncss#usage "Uncss API usage documentation"
[49]: https://github.com/bevacqua/css "bevacqua/css CSS Quality Guide on GitHub"
[50]: https://i.imgur.com/cC0Lwi2.png
[51]: http://www.smashingmagazine.com/2014/09/improving-smashing-magazine-performance-case-study/ "Improving Smashing Magazine’s Performance: A Case Study"
[52]: https://www.filamentgroup.com/lab/font-loading.html "How we use web fonts responsibly, or, avoiding a @font-face-palm"
[53]: https://www.filamentgroup.com/lab/font-events.html "Font Loading Revisited with Font Events"
[54]: https://github.com/substack/bundle-collapser "substack/bundle-collapser on GitHub"
[55]: https://github.com/imagemin/imagemin "imagemin/imagemin on GitHub"
[56]: https://i.imgur.com/ldoRjyK.png
[57]: https://github.com/imagemin/imagemin-gifsicle "imagemin/imagemin-gifsicle on GitHub"
[58]: https://github.com/imagemin/imagemin-jpegtran "imagemin/imagemin-jpegtran on GitHub"
[59]: https://github.com/imagemin/imagemin-optipng "imagemin/imagemin-optipng on GitHub"
[60]: https://github.com/imagemin/imagemin-svgo "imagemin/imagemin-svgo on GitHub"
[61]: https://github.com/imagemin/imagemin-webp "imagemin/imagemin-webp on GitHub"
[62]: https://github.com/aheckmann/gm "aheckmann/gm on GitHub"
[63]: https://github.com/Ensighten/spritesmith "Ensighten/spritesmith on GitHub"
[64]: https://i.imgur.com/JF118g3.png
[65]: https://i.imgur.com/l65vPpr.png
[66]: https://github.com/bevacqua/perfschool "bevacqua/perfschool on GitHub"
[67]: http://bevacqua.io/bf/book "JavaScript Application Design: A Build First Approach, on Manning Publishing"
[68]: https://raw.githubusercontent.com/bevacqua/perfschool/master/resources/menu.png
[69]: https://github.com/buildfirst/buildfirst/tree/master/ch04/01b_critical-inlining "Inlining Critical CSS code sample"
[70]: http://www.amazon.com/gp/product/1617291951 "JavaScript Application Design: A Build First Approach, on Amazon"
[71]: https://www.gravatar.com/avatar/cee019b251cf09f440b4427541e46cb8.png?s=400
[72]: https://github.com/buildfirst/buildfirst "JavaScript Application Design code samples on GitHub"
[73]: /articles/stop-breaking-the-web "Stop Breaking the Web on Pony Foo"
[74]: https://adactio.com/journal/9312 "On The Verge, article by Jeremy Keith"
[75]: http://timkadlec.com/2015/02/client-side-templatings-major-bug/ "Client-side templating's major bug, by Tim Kadlec"
[76]: http://timkadlec.com/2015/05/choosing-performance/ "Choosing Performance, by Tim Kadlec"
[77]: https://github.com/taunus/taunus "taunus/taunus on GitHub"
[78]: #making-less-requests "Making Less Requests"
[79]: #defer-non-critical-asset-loading "Defer non-critical Asset Loading"
[80]: https://i.imgur.com/zq74te2.png
[81]: https://github.com/zachleat/fontfaceonload "zachleat/fontfaceonload on GitHub"
[82]: /articles/fixing-web-performance "Fixing Performance in the Web Stack"
