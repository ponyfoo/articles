# Semantic HTML and Content Relevance #

The absolute first step was *semantic markup*. That is, using **HTML 5** tags, such as `<article>`, `<section>`, `<header>`, `<footer>` and so on. For a _full list of HTML 5 tags_, [visit MDN](https://developer.mozilla.org/en-US/docs/HTML/HTML5/HTML5_element_list "HTML 5 Tag List").

This helps crawlers assign **weight** (importance) to each piece of HTML in your page. It makes your pages _future-proof_, meaning that when and if crawlers start giving more importance to semantic markup, you will be ready for it. Additionaly, _it makes your CSS more semantic, too_, which can't hurt.

Always make sure your content is relevant to the keywords you aspire to be found for, _don't just spit a bunch of keywords_ onto your site and expect good things to happen. Your visitors won't take your site seriously if you do that, **which is your end goal anyways**.

> What is the point in being _well-ranked_ if your visitors don't _consume or appreciate_ the contents of your site?



# Analytics #

The second step I took was adding [Google Analytics](http://www.google.com/analytics/ "Google Web Analytics") to my solution. I later went on and added a couple of other services: [Clicky](http://clicky.com "Clicky Web Analytics") and [New Relic](http://newrelic.com/ "New Relic Application Monitoring), to improve my analytics and have at least some sort of _uptime monitoring_. All these services are really easy to include _effortlessly_ in your application, and they provide **a lot of value**.

Analytics can tell you _what pages_ users land on, what pages are the _most linked to_, where your users _come from_, as well as _how your users behave_ and what they are looking for. In summary, it's really important to know what's going on with your site in the grand scheme of things, and figure out how to proceed, and analytics tools are a _great way_ to accomplish just that.

You also definitely want to sign up for [Webmaster Tools](https://www.google.com/webmasters/ "Google Webmaster Tools"), which will be **immensely helpful** in putting it all together, and will also help you track your index status closely.



# AJAX Crawling #

This is the most complex step I took. Since this site is entirely AJAX, _meaning users navigating only load the site once_, the page is for all intents and purposes _static_, meaning that no matter what page you go to, you get _the same HTML_, and then JavaScript takes care of displaying the appropriate view. This is clearly a **dealbreaking issue** for web scrapers, because they cannot index your site if _all your pages look the same_.

> Without a proper AJAX crawling strategy, any other efforts to improve SEO are **utterly useless**. You need _a good crawling strategy_ that allows search engines to get the content a regular surfer would find in each page.

The desired behavior would, then, be some sort of reflection of this _mockup_:

[![crawler.png][1]](https://i.imgur.com/JdHh6JF.png)
  
### Implementation ###

In my research, I figured out a [headless browser](http://zombie.labnotes.org/ "Zombie.js headless browser") was the way to go. A **headless browser** is basically a way to make a request to a page _and execute any JavaScript_ in it, without resorting to a _full-fledged_ web browser. This would allow me to **transparently** serve web crawlers with static versions of the dynamic pages on my blog, without having to resort to _obscure techniques_ or manually editing the static versions of the site.

Once I had this down, it was just a matter of adding a little helper to _every single **GET** route_ to handle crawler requests differently. If a request comes in and it matches one of the known web crawler user agents, a _second request_ is triggered on behalf of the crawler, against the same resource, and through the headless browser.

After waiting for all the JavaScript to get executed, and after a little cleanup (since the page is static, it makes sense to me to _remove all script tags_ before serving it), we are ready to serve this view to the crawler agent.

One last step if you care about performance is dumping this into a file cache that is relatively **short-lived** (meaning you'll _invalidate that cached page_ if a determined amount of time elapses), in order to save yourself a web request in subsequent calls made by a crawler agent against that resource.

If you are curious about how to implement this, [here is my take for this blog](https://github.com/bevacqua/ponyfoo/blob/v0.2/src/logic/zombie.js "Crawler AJAX Support Implementation"), it is implemented in Node.

Note that _this might not be the latest version_. It's the one contained in the **v0.2** tag. Although I don't expect to change it much.

> Once that's settled, and working, you can do awesome stuff such as updating your `<meta>` tags through JavaScript, and the web crawlers will pick up on it!



# Metadata #

Metadata is **crucial** to being _well-positioned_ in search results. There are lots of meta content you can enrich your site with. I'll talk about some of the metadata you can include, particularly about what _I_ have chosen to include.

_Use `<meta>` tags_

The single most important `<meta>` tag is `<meta name='description' content='...'>`. This tag should uniquely describe each page in your site. Meaning different description tags should never have the same _content_ attribute value. You should keep the description tag _brief_, though _not too short_, since it'll be the description users will get when your page gets _a search results impression_.

The `<meta name='keywords' content='...'>` tag is much discussed, and seems to have been [mostly irrelevant](http://googlewebmastercentral.blogspot.com.ar/2009/09/google-does-not-use-keywords-meta-tag.html "Google does not use the keywords meta tag") for a while now, but _it won't do any harm_ should you decide to include it.

_Provide [Open Graph](http://ogp.me/ "Open Graph protocol") metadata_

**Open Graph** is a set of _meta properties_ pushed by [Facebook](https://developers.facebook.com "Facebook Developers"). These are mostly useful _when sharing links_ to your site on social networks. Try to include a relevant thumbnail, the actual title of each page, and an appropriate description of that page. You definitely should include these in your website if you care about **SEO**.

_Implement [Schema.org](http://schema.org/ "Schema.org vocabulary") microdata_

**Schema.org** microdata allows you to mark up your site with attributes indicating what the content might be. You can read more about microdata on [Google](http://support.google.com/webmasters/bin/answer.py?hl=en&answer=176035 "About microdata"). This will help [Google](http://google.com "Google Search Engine") display your site in search results, and figure the types of content published in your site.



# Search engine Support #

There are at least a couple of ways to _help search engines_ get on the right tracks when _indexing your website_. You should take full advantage of these.

_Provide a [robots.txt](http://www.robotstxt.org/ "robots.txt explained") file_

There isn't _a lot_ to say about [robots.txt](http://www.robotstxt.org/ "robots.txt explained"), don't _depend_ on crawlers honoring your rules, but rather make your site follow the [REST guidelines](http://en.wikipedia.org/wiki/Representational_state_transfer "Representational State Transfer - REST") to the letter, so that your _unsuspecting data_ doesn't get **ravaged** by a curious spider.

_Publish a [sitemap.xml](http://www.sitemaps.org/ "sitemaps.org")_

By submitting a sitemap, you can give hints to a web crawler about how you value your site's content, and help it index your website. You can read more about sitemaps on [Google](http://support.google.com/webmasters/bin/answer.py?hl=en&answer=156184 "About Sitemaps") or [here](http://www.sitemaps.org/ "sitemaps.org").



# Alternative traffic sources #

It is always wise to provide users with _alternative means_ of accessing your website's content.

_Implement OpenSearch Protocol_

A while back I talked about [implementing OpenSearch](/2013/02/05/implementing-opensearch "Implementing OpenSearch"). This allows users to _search_ your site _directly_ from their browser's _address bar_.

_Publish Feeds_
  
I don't think feeds such as [RSS](http://en.wikipedia.org/wiki/RSS "Really Simple Syndication") need an introduction. I _recommend_ to publish at least a single feed to your site's content. Keeping it up to date is _just as important_.


  [1]: https://i.imgur.com/JdHh6JF.png "Desired behavior"
