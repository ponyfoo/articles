# Measuring performance _(and business metrics)_

Measuring the performance of your site helps you **figure out where you are on the scale**, how bad _(or how well!)_ your site is performing, and it also helps you keep track of the effect of changes made to your application.

There are quite a few different tools you can use to gauge performance and business metrics. When it comes to business metrics, you'll typically use a tool like [Google Analytics][1] or [Mixpanel][2]. Both are pretty good at tracking human interactions, studying engagement, [A/B testing][3], and comparing different products or categories. I won't bore you about tracking business metrics though, because that's _probably up to someone else_ in your company.

Instead, let's focus on gauging performance. Popular services for measuring web application performance include [YSlow][4], [PageSpeed Insights][5], and [WebPageTest.org][6].

## Using Yahoo YSlow

While lately it has been declining in popularity, YSlow is still a **good source of performance tips**. Nowadays, YSlow consists of browser extensions and there's [the `grunt-yslow` Grunt plugin][7] as well.

YSlow provides many basic suggestions, such as minification and bundling, as well as more advanced ones. Here's a list of the highlights, _mostly caching-related_ rules.

- Use a Content Delivery Network _(CDN)_
- Use `Expires` headers
- Enable gzip compression
- Configure `ETags`
- Reduce cookie size

It's a great tool to get started in the fantastic world of web performance optimization.

#### YSlow with browser extensions

Using the browser extension is merely a matter of installing it and clicking on the button. YSlow will open up another window, and once you run a test you'll get a grade sheet like the one below.

![YSlow test results][8]

#### YSlow in Grunt with `grunt-yslow`

The Grunt plugin couldn't be easier to configure. You just install `grunt-yslow` and add the following piece of code to your `Gruntfile.js`.

```js
module.exports = function (grunt) {
  grunt.loadNpmTasks('grunt-yslow');
  grunt.initConfig({
    yslow: {
      pages: {
        files: [{
          src: 'http://ponyfoo.com',
        }],
        options: {
          thresholds: {
            weight: 500,
            speed: 5000,
            score: 90,
            requests: 15
          }
        }
      }
    }
  });
};
```

Then you're all set, just run the command below and you'll get the measurements right in your command line!

```shell
grunt yslow
```

![YSlow in your terminal using Grunt][9]

While the automated Grunt plugin is **great at getting a YScore score in a pinch**, it doesn't produce _the same level of detail_ provided by the browser extension, which is why I'd recommend using the extension at least once before attempting to automate the process using Grunt or some other automated task runner.

That being said, using the Grunt plugin allows you to do a few things. First, you automate the process, meaning you won't be having to click your way through the browser extension anymore. You could run YSlow on every build now! Second, it allows you to set thresholds, meaning that you can define constraints such as _"no more than 15 requests"_, or "the page must load in under 5 seconds". If one of those thresholds is exceeded, then the Grunt task will fail, _causing the build to fail_, stopping the deployment _(in case you're doing [automated deployments][10])_.

#### YSlow in your command-line

You can use a combination of [PhantomJS][11] and the [yslow.js script][12] in order to run YSlow from the command-line. You can choose from a few different output formats _(such as JSON or plain text)_, verbosity, and more. Here's how to invoke it.

```shell
phantomjs yslow.js --info basic --format plain ponyfoo.com
```

The output should look something like the screenshot below.

![YSlow CLI test results][13]

In this day and age, the test suite provided by YSlow might seem all too basic. PageSpeed can probably help you better understand your performance issues.

## Using Google PageSpeed Insights

PageSpeed Insights offers a good mix of rules and suggestions for mobile web development, tweaked for both mobile and desktop connections. They offer a screenshot so that you can see what Google is looking at, and every rule comes with a detailed description which explains _how to implement a fix_, and the reasoning why your site was marked as insufficiently optimized in that regard.

PageSpeed provides [a super simple web interface][14] where you type in an URL, wait a minute, and see your score plus some suggestions. Here's an example.

![PageSpeed test results][15]

It's great, it let's you in on many secrets, and it heavily implores you to optimize your images, inline assets and get non-critical content out of the critical rendering path. PageSpeed is also content-aware, and it has the ability to tell you whether your content fits on the device viewport, whether you are using a legible font size, and suggest that you eliminate rendering-blocking JavaScript and above the fold CSS.

We'll take a harder look at these optimizations in a future article, for now let's focus on the PageSpeed tool on Grunt, Gulp, and as a CLI.

#### PageSpeed in your command-line via `psi`

Google provides [a package called psi][16] for interacting with PageSpeed Insights. You can use that package as a CLI or a programmatic API. The CLI is super easy to use.

```bash
npm install -g psi
psi http://ponyfoo.com
```

You can optionally configure it with an API key provided by Google, in order to avoid throttling. Here's how the output looks in your terminal.

![PageSpeed Insights in your terminal][17]

My conclusions here are similar to those for YSlow: try it by hand in your browser first, then establish the automated tests in place to automate the measure-optimize feedback loop.

#### PageSpeed under Gulp

Here's a `Gulpfile.js` to run PageSpeed over Gulp. You'll need to install `psi` for this one again, but you'll be using its API this time.

```js
var gulp = require('gulp');
var psi = require('psi');
gulp.task('mobile', function (cb) {
  psi({
    nokey: 'true',
    url: 'http://ponyfoo.com',
    strategy: 'mobile',
  }, cb);
});
```

Grunt is similarly easy to configure!

#### PageSpeed in Grunt

You'll need [grunt-pagespeed][18], which uses psi under the hood to deliver the goods.

```js
module.exports = function (grunt) {
  grunt.loadNpmTasks('grunt-pagespeed');
  grunt.initConfig({
    pagespeed: {
      production: {
        options: {
          nokey: true,
          url: 'http://ponyfoo.com',
          locale: 'en_US',
          strategy: 'desktop',
          threshold: 80
        }
      }
    }
  });
};
```

The last performance testing tool we'll be going over is [WebPageTest.org][19].

## Using WebPageTest.org

WebPageTest is also maintained by Google, and its uniqueness comes from their detailed reviews of the TCP connection every step of the way. The screenshot below shows how they indicate how the browser is behaving under the hood. Are connections being reused _(keep-alive)_? What's the **roundtrip latency** for the connection? **What's the time to first byte?** How long until the page is _visually ready_?

![Connection waterfall from WebPageTest.org][20]

Another very interesting aspect of **WPT** is the _"filmstrip view"_. This view allows you to quickly identify issues with your web page on first load. In my case, for example, it sentences that I should probably do something about font loading, because it's blocking rendering longer than it should!

These kinds of quick observations can be made by analyzing **WPT** test results. I guarantee that you'll find quite a few things wrong with your site!

![Filmstrip view for Pony Foo on WebPageTest.org][21]

When it comes to automation, we can use the [webpagetest][27] package from npm, although it requires an API key. At the moment getting a WebPageTest API key is kind of awkward: you need to either host your own instance of WPT, or send the project owner an email and hope for the best.

Once you have an API key, you could run it from your terminal like so:

```shell
webpagetest test http://ponyfoo.com --key '<your-key>'
```

Similarly, there's [grunt-wpt][29] if you belong to the Grunt caste.

# Further learning

I almost forgot to mention [grunt-perfbudget][22]! It lets you define [a performance budget][23] for your application, something you definitely should take a look at! The plugin offers a bunch of options to measure things like how long the page takes to load, how fat it is, speed index, and many more. You can even configure the type of connection you'd like to test it on _(DSL, 3G, **LTE**, <sub>dial-up</sub>, etc)_.

Ooh, have you checked out [perf.fail][24]? It's a pretty amazing blog chock-full of case-studies about companies doing performance research and their findings, and companies not focusing resources on performance _(and the abysmal implications)_.

If you already haven't, also watch [Addy][25]'s [CSS Performance Tooling][26] talk at CSSConf EU 2014.

  [1]: https://www.google.com/analytics/web/
  [2]: https://mixpanel.com/
  [3]: http://www.smashingmagazine.com/2010/06/24/the-ultimate-guide-to-a-b-testing/ "The Ultimate Guide to A/B Testing"
  [4]: http://yslow.org/ "YSlow analyzes web pages and why they're slow based on Yahoo!'s rules for high performance web sites"
  [5]: http://www.webpagetest.org/ "WebPageTest.org helps test a website's performance"
  [6]: https://developers.google.com/speed/pagespeed/insights/ "PageSpeed Insights helps you make your web pages fast on all devices"
  [7]: https://github.com/andyshora/grunt-yslow "andyshora/grunt-yslow on GitHub"
  [8]: https://i.imgur.com/b95evOX.png "YSlow test results"
  [9]: https://i.imgur.com/APY8f2y.png "YSlow test results using Grunt"
  [10]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
  [11]: http://phantomjs.org/ "PhantomJS is a headless web browser"
  [12]: http://yslow.org/yslow-phantomjs-3.1.8.zip "yslow for PhantomJS"
  [13]: https://i.imgur.com/BLu38RA.png
  [14]: http://www.webpagetest.org/ "WebPageTest.org helps test a website's performance"
  [15]: https://i.imgur.com/DmkgPDk.png
  [16]: https://github.com/addyosmani/psi/ "addyosmani/psi on GitHub"
  [17]: https://i.imgur.com/Z9zLreY.png
  [18]: https://github.com/jrcryer/grunt-pagespeed "jrcryer/grunt-pagespeed on GitHub"
  [19]: http://www.webpagetest.org/ "WebPageTest.org helps test a website's performance"
  [20]: https://i.imgur.com/uabEpUz.png
  [21]: https://i.imgur.com/LwpIkoj.png
  [22]: https://github.com/tkadlec/grunt-perfbudget "tkadlec/grunt-perfbudget on GitHub"
  [23]: http://timkadlec.com/2014/05/performance-budgeting-with-grunt/ "Performance budgeting with Grunt"
  [24]: http://perf.fail "perf.fail: do, learn, fail forward."
  [25]: http://addyosmani.com/ "Pay a visit to Addy's blog, too!"
  [26]: https://www.youtube.com/watch?v=FEs2jgZBaQA "CSS Performance Tooling by Addy Osmani"
  [27]: https://github.com/marcelduran/webpagetest-api "marcelduran/webpagetest-api on GitHub"
  [28]: http://www.webpagetest.org/forums/showthread.php?tid=884 "'Missing API key' thread on WebPageTest.org forums"
  [29]: https://github.com/sideroad/grunt-wpt "sideroad/grunt-wpt on GitHub"
