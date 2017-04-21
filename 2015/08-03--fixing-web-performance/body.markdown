## Understanding TCP, the Bowels of the Web

You would assume there isn't a whole lot we can do about TCP, or that it wouldn't have a big impact on web performance, but it turns out there's a couple of things we can do that are quite useful to web applications. To implement these optimizations you **generally need to be either hosting the application on your own hardware, or on an IaaS service** such as Amazon Web Services or Digital Ocean.

It might also apply to a few PaaS services where you can run arbitrary command line arguments, but _I wouldn't bet the farm on that_.

### Increasing the initial TCP `cwnd` size

One of the configuration values in TCP is `cwnd`, or the _"congestion window"_. This value determines how much data can be sent through the wire, and it grows as connection roundtrips can effectively handle the throughput. If we increase the initial TCP `cwnd` size, we could [essentially be saving a few roundtrips on the connection][33] that would otherwise end up transferring too little data.

Increasing the `cwnd` is useful because you might make it so that the entire `<head>` of a page fits in the first roundtrip on the connection, for example. That's a big deal because we could start rendering the page faster, fetching other resources, etc.

> #### Implementation
> 
> To accelerate connection ramp-up by increasing the initial `cwnd` size, it usually suffices to ensure that we're running the latest version of our favorite Linux flavor. If that doesn't work, then you might want to read an article on [how to manually tune the `initcwnd` value][34], that also goes into _detail about the reasoning behind_ doing so.

### Disabling Slow-Start Restart _(SSR)_

SSR is a mechanism built into TCP that really dampens HTTP. It has a noble purpose, though. The idea is that if a TCP connection goes idle for a while, then we should go back to safer levels of throughput, essentially cutting down on how much data can get through the wire. The problem is that this contradicts everything we're working towards in HTTP. Especially, if you've enabled the HTTP `keep-alive` mechanism _-- which you should have, as we'll see in a minute --_ SSR defeats the purpose. HTTP `keep-alive` reuses a TCP connection across multiple HTTP requests, but if throughput goes down in between requests, then much of the value added by `keep-alive` is lost.

> #### Implementation
> 
> To disable slow-start restart, you can run the following command in your terminal:
>
> ```bash
> sysctl -w net.ipv4.tcp_slow_start_after_idle = 0
> ```

## Web Performance at the HTTP level

There's quite a few tips I have for you regarding the Hyper-Text Transfer Protocol. I'll split each of the points regarding HTTP into their own sub-sections, so that readers who just skim the headlines also get some value out of the article.

### Making Less Requests

The fastest request is a request not made. Depending on how you interpret that phrase, it might sound obvious. _Who would make requests if they had no use for the response?_ In the case of **HTTP 1.1**, that usually translates into bundling requests together, maybe by concatenating static resources or creating a spritesheet for the various icons in your site.

In the context of **HTTP 2.0** this advice is transmogrified. **HTTP 2.0** utilizes a single TCP connection per origin, and all requests are multiplexed over that connection. In this scenario, concatenation and spriting might actually be perjudicial. Most of the time, the benefits in **HTTP 2.0** far outweight the _"drawbacks"_ in concatenation, so that it's still useful to concatenate resources for those clients that are still making **HTTP 1.1** requests to your servers.

> #### Implementation
> 
> This is a broad thing to ask of your applications, so here are <mark>some tips that may get you started</mark> making less requests.
> 
> * Tailor the application's web API to serve the needs of the client-side, and not the other way around
> * Bundle images, scripts and styles into larger files for connections over **HTTP 1.1**
> * Only make requests that are absolutely necessary, when it is necessary to make them
> * Cache their results as aggressively as you can get away with, saving time and improving UX

### Turning on `keep-alive`

As I've mentioned earlier, `keep-alive` is an HTTP mechanism that allows the same TCP connection to be kept open and reused across many HTTP requests. This innocent mechanism is one of the best optimizations you can indulge in, when it comes to HTTP.

Enabling `keep-alive` reduces the amount of hand-shaking, and thus latency, for every request that's kept alive by reusing one TCP connection. **HTTP 2.0** one-ups `keep-alive`, by reusing the same TCP connection for every single request made against an origin.

> #### Implementation
> 
> Luckily for us, `keep-alive` is enabled by default in `nginx` which you should be using! It's also enabled by default in Node.js, and fairly easy to turn on everywhere else, so there's no excuse not to turn `keep-alive` on!
> 
> In all other cases, you can add a `Connection: keep-alive` header to your responses, and that'll suffice.

### Enabling GZip Compression

GZip is one of those magical algorithms that make your content download much faster virtually for free. It generally only takes a flip of a switch in web server software like Node.js or `nginx`. The way it works is it'll identify repeating strings of text in the content and replace them with identifiers mapped to an index table. This makes text-based content way smaller. The only two cases where GZip performs below par is when content is so small that it'd fit in a single roundtrip anyways, and when we're dealing with binary content. In the former case, _-- generally files that are 1kb in size or smaller --_ we're adding all of the GZip processing overhead for none of the benefits, since the file size might even end up increasing. In the latter, GZip doesn't really find much in the way of repeating strings, so it's usually avoided altogether.

> #### Implementation
> 
> Easy to implement in `nginx`, you can just throw in the `gzip on` directive in your `http` configuration section. Here are the settings I regularly use.
>
> ```bash
> <mark>gzip on</mark>;
> gzip_disable 'msie6';
> gzip_comp_level 6;
> gzip_vary on;
> gzip_min_length  1000;
> gzip_proxied any;
> gzip_types text/plain text/css application/json application/x-javascript text/javascript text/xml application/xml application/xml+rss image/x-icon;
> gzip_buffers 16 8k;
> ```
> 
> In Node.js you can turn on GZip with the `compression` package for Express. They provide you with reasonable configuration defaults, but you could still tweak it.

### Caching with Expires and ETag headers

Caching is one of those things everyone nods their heads in agreement and then we rarely actually get around to. Yet, it's probably one of the easiest ways to reduce load in your servers. Remember what we pointed out earlier about requests?

> **<mark>_The fastest request is a request not made._</mark>**

The `Expires` header is used by setting a date in the far future that determines when the content will go stale, meaning a new copy should be requested beyond that point. The header is usually paired with hashes in file names. For example, we generate an MD5 hash of the content of our JavaScript bundle, append it to its filename, throw in an `Expires` header, and serve that. The client will only download that file once. Then, when we update the file, the MD5 hash will change, thus the filename will be different, meaning HTTP will treat it as a different resource altogether, and it'll be download once, again.

Similarly, `ETag` is a header that you're supposed to set to a hash of the content, and the browser will first ask the server if the `ETag` changed, instead of downloading the entire resource every time.

> #### Implementation
> 
> In `nginx`, the `expires` directive is good enough to deal with static assets. Here's an example `location` section that serves static assets with high performance. Note how I'm also turning off the `access_log`, as it may not be as interesting as requests for other endpoints in your site.
>
> ```bash
> location ~ ^/(images/|js/|css/|fonts/|favicon.ico) {
>   root {STATIC_ROOT};
>   access_log off;
>   <mark>expires max</mark>;
> }
> ```
> 
> In Node.js you can turn caching on for static assets with the `serve-static` package for Express. Keep in mind that this won't work for your views or API!

### Using a Content Delivery Network _(CDN)_

Using a CDN can come in handy if you need to maximize performance for static assets. The way they work is that clients are instructed to ask servers that are phisically near them for those assets, reducing latency. A lot of the time, using a CDN is overkill. For example, using a CDN for a blog like Pony Foo would be largely unnecessary.

That being said there's a few free CDN providers _([CloudFlare][85] is one of the most prominent ones)_ that you can easily set up for your projects at no cost to you.

> #### Implementation
> 
> [CloudFlare][85] is one of the easiest ones to use. It can act as a pass-through DNS for your application, and then you can have them point at your backend servers, that end up serving the responses. CloudFlare then ends up caching your content and intercepting requests for static assets, and serving them near the edge of the network, closer to the user.

### What about **SPDY** and **HTTP 2.0**?

Look into enabling these protocols in your servers. Many times, they can yield as much as 60% gains _overall_ and they're mostly a drop-in improvement. That's as good as it gets in the world of performance optimization. We've [already covered][78] the benefits of using a single TCP connection per origin and multiplexing all the requests _(and responses)_ on that connection.

What else does **HTTP 2.0** bring to the table? A couple of things.

There's **header compression**, where a table of _"seen"_ HTTP headers is constructed and used to indicate headers instead of transferring the entire record through the wire every single time. Headers sometimes make up for a large portion of the request response cycle, particularly when you take into account large analytics cookies and so on.

Another improvement brought forth by **HTTP 2.0** is named _proactive server push_. This is a fancy way of saying that in **HTTP 2.0** the server can hint to the client that it should start downloading other resources alongside the HTML. That means you could ask the client to start downloading styles, fonts, and scripts while the HTML is still being loaded.

Besides the four _"hard improvements"_ outlined thus far, _-- one TCP connection, multiplexing, header compression, server push --_ there's also implicit improvements in **HTTP 2.0**, in that it naturally removes the need for hacks of the past such as image inlining, spriting, concatenation, and even minification to some degree.

> #### Implementation
> 
> In `nginx` you can use the experimental [`ngx_http_spdy_module`][35] module to turn on SPDY. You are required to also set up TLS, as that's regularly being shoved down our throats _(for good reason)_ when it comes to implementing the latest and greatest features of the web.
> 
> There isn't wide support for **HTTP 2.0** in `nginx` and friends quite yet, but there's active effort to get it [out the door by the end of the year][36], at least for `nginx`.

## How about some HTML tips?

Sure thing. When it comes to HTML, the single best piece of advice I have for you is: **leverage it**. The best way to get content as fast as possible to the user is to actually serve that content to them -- no intermediaries. In other words, that means that **you should be doing server-side rendering** if you aren't yet.

### Server-side Render all the Things

Instead of getting _"creative"_ and having the human wait for your HTML to finish loading, your blocking styles and images to load, your scripts to load and get parsed and executed, and your client-side rendering to kick in, why not try and serve the content right away?

You can always become a single page application later. I'm not saying single page applications are bad, I'm saying that [Server-First apps][37] are a great idea. The idea is that you render the HTML first, content and all, completely usable, links that humans can actually click on and go places. Then, once the content loads and the user starts making sense of your page, you can start loading JavaScript in the background. When that is executed, then you can start hijacking links, form submissions, and turning on the much appreciated realtime WebSocket communication machinery.

But seriously, get the content out there right away. That's all that matters to the user after all, and if you make them wait six seconds for the client-side JavaScript to render the page while all they see is a loading indicator endlessly looping around, they'll grow sick and tired and leave long before your spinner finishes it's graceless dance.

> #### Implementation
> 
> Server-side rendering is hard in the current state of the web. This is most unfortunate. **Angular doesn't support server-side rendering**. Ember doesn't either _(their "support" amounts to rendering a non-interactive HTML page for web crawlers)_. Meteor isn't for everyone, due to the high level of commitment one has to pour on their platform.
> 
> You could cram shared rendering into a Backbone application if you were to use [`rendr`][38], but it's _a weak framework_ that <mark>**forces you to rewrite your application into a mess**</mark> that just happens to work on the server-side and supports Backbone. It's not [being actively developed][39] anymore either. Bottom line -- there's better alternatives today.
> 
> [React][40] supports shared rendering natively -- and it might just be the best choice today when it comes to shared rendering. Granted, you'll need to have Node.js for the application backend to run the server-side rendering part, but that's increasingly the standard, and **I foresee it becoming ubiquitous when it comes to application development**, simply because of its effectiveness at shared rendering without code duplication in different languages.
> 
> For the adventurous, an alternative might be [Taunus][77]. Taunus runs on Node.js in the server-side, and it's a shared rendering library I built, that's enthusiastic about progressive enhancement and developing applications using `<form>` elements. The goal is that Taunus applications work in a selection of devices and browsers as broad as possible.

### Defer non-critical Asset Loading

This is mostly a remake of the previous point. Non-critical assets should be loaded asynchronously. This means everything from styles, fonts, and images to JavaScript, advertisement, and realtime features. I understand that your business may revolve around ads, but I'm sure there's better ways of progressively interleaving content and ads than what most media sites are doing to the web.

> #### Implementation
> 
> When it comes to `<script>` tags, just add `async` to them. Styles and fonts in `<link>` tags are a bit harder, but the following snippet will get you there.
>
> ```js
> var elem = document.createElement('link');
> var head = document.getElementsByTagName('head')[0];
> elem.rel = 'stylesheet';
> elem.href = '/css/all.css';
> <mark>elem.media = 'only x'</mark>;
> head.appendChild(elem);
> setTimeout(function () {
>   elem.media = 'all';
> });
> ```
> 
> Of course, remember to keep the `<link>` tag, but inside a `<noscript>` element!
>
> ```html
> <noscript>
>   <link rel='stylesheet' type='text/css' href='/css/all.css'>
>   </noscript>
> ```
> 
> Images are easy to defer. Just use an attribute like `src` instead of using `src`, and then when the image becomes relevant _-- because we scroll near it, the rest of the page has finished loading, or any other reason --_ we set the value for `src`, and the image gets loaded. Again, remember to add a `<noscript>` tag with the image in it, using the actual `src` attribute!

## Any Help with Cascading Style Sheets?

Yes! Plenty. I've already [written about CSS performance][8] back in the day, but I'll just write some more here -- just to keep up with the slides!

### Inlining Critical CSS

This tip is tightly tied into the last one. In order to be able to [defer non-critical][79] CSS, we need to identify critical CSS and inline it. Critical CSS is any CSS that's needed to render the content that's above the fold, the content that's first presented to the human when they load the page. There are tools that automate the identification process for you. Then all that's left to do is to inline that CSS inside a `<style>` tag in your app, and defer the rest of the styles.

[![Penthouse used in a variety of ways][43]][42]

The above is a screenshot of the documentation for [`penthouse`][42], a tool you can use to automate the critical CSS identification process.

> #### Implementation
> 
> This one can be cumbersome to implement -- but it _really_, <mark>really,</mark> _**really**_ pays off!
> 
> The simplest way to go about doing this might be using [`critical`][41], which does the heavy lifting of extracting the CSS via [`penthouse`][42], inlining it in the page, and deferring the CSS from the `<link>` tag using a technique like the one we outlined earlier in [_"Defer non-critical Asset Loading"_][79].

You'll notice huge performance gains because of two reasons.

* CSS can be immediately applied to the above the fold content
* There are no longer `<link>` tags blocking rendering while we wait on CSS to be downloaded

### Removing Unused Styles

Possibly one of the mildest techniques I've described so far. Tools exist that let you identify CSS rules that don't impact the site and can be safely removed. The example I usually give is how you sometimes use libraries like Bootstrap that provide you a fast jumpstart into putting together an application with some CSS, and then you use pretty much three of the rules in the library. That way you end up with thousands of rules you don't actually use, and which only add bloat to your site.

> #### Implementation
> 
> The [`uncss`][44] package makes the process of removing unused CSS a breeze -- as long as you're serving individual stylesheets for each page on your site. You can use it directly as a command-line tool, through the [Grunt][45] plugin, the [Gulp][46] plugin, [Broccoli][47], or it's Node.js [API][48].

### Avoiding `m.` subdomains

Not strictly performance related, but I see performance as something that's very closely related to UX, and having an `m.` subdomain is not only futile but it also affects UX _(and maintainability)_ very negatively. It is futile because it's impossible to categorize every device in existant as either mobile or not mobile.

Instead of relying on an `m.` subdomain, go the responsive route. Use mobile-first if possible, and try and design a consistently usable experience that's optimized for performance as well.

> #### Implementation
> 
> Don't try to categorize every single device as either _"mobile"_ or _"desktop"_. There's a lot of in between experiences, and just taking a mobile first responsive web design approach is light years ahead of using `m.`-style viewport-specific experiences.
> 
> <mark>_Just be reasonable._</mark>

### Follow a Style Guide

Pick a style guide, any style guide. Not a magic trick. Picking a style guide _(and actually sticking to it)_, is one of the best things you can do for your project. I've seen far too many projects where CSS is an afterthought, an **impossibly disgusting _"anything goes"_** region of a site that nobody even dares to ask if there are any conventions to be followed or classes to be reused.

[![Here&#x2019;s a sample style guide][50]][49]

When you [follow a style guide][49], at least there are some conventions to be respected, and everyone in the team will be less miserable about having to fix layout or design issues. You can buy me a beer at the next conference we run into each other.

> #### Implementation
>
> Just pick a style guide, any style guide. Maybe start out with [mine][49], but that's biased. You could use something like [GitHub's style guide][83]. Or maybe you could pick [Harry Robert's CSS Guidelines][82]. Or [SMACSS][84]. Just choose one and follow it. Enforce it, create a culture where _everyone_ **understands the importance of developing maintainable CSS**.
>
> _Collaborate to avoid pandemonium across your CSS files._

## How About Fonts? Anything There?

Absolutely. Fonts are hard. Fonts are slow. I have a few pointers about fonts for you. First, load them asynchronously. Secondly, **use fewer fonts** in your future projects. Third, cache the hell out of them!

### Using Fewer Fonts

As I usually point out during the presentation, this is not something you can just head back to the office and strip away. The designer will not be amused. That being said, it's entirely possible to work with them to make sure you use, _for example_, two fonts at most. Constraints are actually great for design, so they won't be disappointed about the challenge.

> #### Implementation
>
> Naturally, you won't be able to remove fonts thoughtlessly from existing projects, as the design would probably suffer from it. The <mark>next time you're tackling a project</mark>, _however_, just make sure to work alongside the designers to come up with a design and a UX that makes sense from a performance standpoint.
>
> A lot of good can come from the intersection between design and performance. To this point, you might want to read [Designing for Performance][21], which was made [freely available online][86] just _a few days ago_.

### Loading Fonts Asynchronously

This is something we discussed earlier when we were talking about WPT and their filmstrip view. You can load a font asynchronously by deferring loading on the `<link>` tag that loads the `font-face`. Meanwhile, a fallback web-safe font could work just as well for the user, who only cares about content _(for the most part)_.

When the custom font is eventually loaded, you can feel free to apply it to the entire document as needed, and be glad that you didn't force your customers to stare at blank chunks of a website for several seconds while fonts loaded synchronously!

> #### Implementation
> 
> You can use the [`fontfaceonload`][81] package from npm to detect when the font is loaded and add a class name to your document, overriding the web safe font you're using by default. That'll be enough to ensure your fonts don't block rendering and to prevent FOIT in one fell swoop.
>
> [![Font face on load slide][80]][81]
>
> Note that you'll still have to load the font asynchronously on your own, using a technique such as [the one described above][78].

### Cache Fonts Aggressively

This one should be obvious enough. Fonts are **really expensive** to load. Make sure you take the necessary precautions so that fonts are only loaded once.

> #### Implementation
> 
> There's several techniques to caching fonts. You can use `localStorage`, the HTTP cache, and many techniques in between. Do your homework and figure out what approach works best for your use case.
> 
> On this front, you would do well to read ["Improving Smashing Magazine's Performance"][51], and pretty much [any article about fonts][52] posted by the folks at Filament Group, such as [this one right here][53].

## Images?

_Yay._ Images are the often overlooked part of web optimization that we all know is the biggest pain point. Yet, we often end up obsessing about cutting bytes from our Browserify bundle using [`bundle-collapser`][54] much more frequently that we obsess about making smarter use of images in our sites. Here's a few tips about images.

### Minify and Shrink

Minifying images is great. It's also a broadly understood concept, most of us already know that we should be minifying images, as we save bytes and most of the time you can't perceive any changes to the image, from a visual standpoint. A more interesting technique that I don't see discussed often enough is shrinking images. Particularly when it comes to user-uploaded content, images tend to be freaking huge. A screenshot on a typical Mac tends to be anywhere from **700K** to **7MB**, but they're also usually around **3000px** wide. A simila situation can be described about pictures taken with a phone.

[![Image optimization tooling, including imagemin][56]][55]

In most cases, you don't need high resolution fidelity in user uploaded content, and can get away with shrinking their images. If you reduce them to, say, at most **700px** in width, you'll save a considerable amount of bandwidth on every request being made for the images. If you have the infrastructure and can get away with saving different versions of the image _(original, small, medium, large)_, then all the better. Your mobile users will be eternally grateful!

> #### Implementation
> 
> When it comes to minification of images, [`imagemin`][55] has your back. It has a plugin based architecture, and there's plugins for [`.gif`][57], [`.jpg`][58], [`.png`][59], [`.svg`][60] and **_even [`.webp`][61]_!** Each of the plugins wraps around robust tools that can be used to perform optimizations on the image binary as well as removing metadata artifacts, and it's all done on your behalf!
> 
> Shrinking images isn't that hard, and we can use GraphicsMagick for that, or the [`gm`][62] package in Node. Here's a programmatic one-liner to shrink images with a callback.
>
> ```js
> function shrink (file, limits, done) {
>   gm(file).autoOrient()<mark>.resize(limits.width, limits.height)</mark>.write(file, done);
> }
> ```

### Defer Images Below the Fold

Images below the fold are usually loaded alongside most of the document, unnecessarily eating up resources that could be better leveraged elsewhere while the images are not in the viewport. There's quite a few ways you could load them asynchronously. Maybe as the user scrolls, like some fancy sites do, or maybe just after the initial page load.

Considering they're non-critical assets, since they're not even visible, the bandwidth would be better spent elsewhere!

> #### Implementation
> 
> We've already covered this earlier! Head back to [_"Defer non-critical Asset Loading"_][79].

### Create Spritesheets using Tools

This one should be obvious to the modern developer, but I'll just add this here in case a designer is reading this and they're working alongside sadistic developers. Don't have your designers maintain spritesheets by hand anymore. It's 2015, try and have them keep icons in individual files, and use tools to create the spritesheets. It'll translate into much faster iteration cycles for you as well -- even if it costs you some satisfaction from being a sadistic bastard.

> #### Implementation
> 
> You could use [`spritesmith`][63] to generate your spritesheets and their accompanying CSS. The cool thing is that they're able to produce output for SASS, LESS, Stylus, plain CSS, and even JSON. You can use the output as variables in your pre-processors, and it's pretty easy to have it produce both **@1x** and **@2x** icons _(retina)_ as well.
> 
> [![][64]][63]

## What about JavaScript?

You should be able to live without it. I always feel kind of weird when I say this out loud at JavaScript conferences. The point is about circling back to what we've discussed about the server-side rendering earlier. Content shouldn't be dependant on JavaScript to be rendered, that's just way too slow on first load. On subsequent loads that's great. For the love of god though, embrace server-side rendering.

### Defer All of It

Again reinforcing the point. You should be able to defer all JavaScript. Ads, Twitter embeds, share buttons, client-side controllers, client-side rendering, you name it, you should defer it. Load it asynchronously and make your site gradually resilient to the harsh and brave new world of spotty mobile connectivity.

> #### Implementation
> 
> Use a combination of server-side rendering, _to quickly get your content to human eyeballs_, and `<script async>` to defer JavaScript without blocking execution. Architect your application in such a way that buttons and forms still work while JavaScript is being downloaded, so basic interaction with the site is still plausible in the slow-loading world of mobile networks.
> 
> See [Taunus][77] as a possible way in which you can boost progressive rendering in your future applications.

### Use Small Modules

Luckily I don't have to fight that hard to explain this one anymore. ES6 modules are on the rise and everyone else seems to be using Browserify. This is great stuff. Let's keep it up. If you're already on the modularity boat, I encourage you to introspect and see if you can build smaller modules than you currently are building. If you're not yet using ES6 modules or Browserify, try and make the switch soon. You're missing out, and the alternatives won't be around for long. Except maybe for Webpack, _meh_.

> #### Implementation
>
> Developing code in small modules is a matter of habit. If you haven't yet, consider [becoming an open-source communist][87], as that'll get you in the habit of writing small, focused modules that follow the unbreakable principle of _"doing exactly one thing, and doing it well"_.
>
> If open-source isn't your bread and butter you would still benefit from using either Browserify or ES6 modules exclusively for a few months. As long as you keep your modules short, even if you force yourself to do it, over time you'll see how you start putting together cleaner API touch points that foster reusability and code clarity.

### Vendor Scripts Sold Separately

Cache vendor scripts on a bundle of their own. This is usually quite useful because vendor scripts tend to not change as often as user scripts, so you can have them cached for longer. If you serve different user scripts on each page, all the better, chances are you still need most of the vendor scripts across all pages anyways.

> #### Implementation
>
> Just split third party scripts from the ones you've written, and serve them as different bundles.

# Phew, that's all I had!

Hope you've found any of this useful and are able to put some of it to good use. In case you've missed it, [the previous part][88] described how to identify issues during continuous integration, which might also come in handy.

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
[71]: http://www.gravatar.com/avatar/cee019b251cf09f440b4427541e46cb8.png?s=400
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
[83]: http://primercss.io/scaffolding/ "CSS toolkit and guidelines that power GitHub"
[82]: http://cssguidelin.es/ "CSS Guidelines by Harry Roberts"
[84]: https://smacss.com/ "Scalable and Modular Architecture for CSS"
[85]: https://www.cloudflare.com/ "CloudFlare CDN"
[86]: http://designingforperformance.com/ "Designing for Performance Minisite"
[87]: /articles/food-for-thought-begins "Blogging and OSS — Food for Thought"
[88]: /articles/talk-about-web-performance "Let's talk about Web Performance"
