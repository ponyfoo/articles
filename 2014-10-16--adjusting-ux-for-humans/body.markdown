# Landing page experience

Few aspects of UX rank **as highly** as an enticing landing page. The landing page makes or breaks engagement. I'd recommend you read the book [Don't Make Me Think][1], a classic in this area, which makes you do the thinking in behalf of your humans, so they don't have to figure out how to use your site. Instead, **interactions should be straightforward** and there should be an obvious course of action, ideally a single course of action, when humans would like to perform a given task on your application.

[The Design of Everyday Things][2], another great book regarding UX and usable designs, also deserves a read. This one is more focus on the psychology of UX, and _in fact_ the book was originally named "The _Psychology_ of Everyday Things".

Anyways, back to the case study at hand! Here's how the landing page looks like on mobile today.

![inception.jpg][3]

This is a pretty typical landing page in mobile. You have a link to the search input, shaped like a button. This lets me avoid adding extra JavaScript just to enable search on small devices, as I'm just linking to `#search`. Then, the human can come back to the top of the page with a link that gets them right back to `#top`.

The homepage used to have a few full articles in it, and then infinite scrolling _**(why of course!)**_ could be used to load the rest of the articles. That sounds great in hindsight, but the _hard truth_ is that if people go to a blog's home they don't want to read every single article end-to-end. No, they want to **see what content you have**. Then, if one of your articles is of interest to them, they may want to go ahead and read the full version.

Having full articles rendered on the landing page also meant that the page would render slowly, and it'd waste that time rendering articles nobody was going to read through! Considering that **the landing page typically is the most visited page** on a site, I was in pretty bad shape. Thus, if you now go to the landing page _(or any search results page)_ you'll be met with excerpts of articles and a huge call to action prompting you to read the full thing, rather than a sea of words just to get to the next article which may or may not be interesting to the reader.

I've also come to realize that infinite scrolling wasn't really fit for a blog, so I removed it entirely, only allowing humans to manually go through the pages. Humans can also refer to the ["Archives" link][4], located at the sidebar _(or footer in the case of mobile)_. The archives enumerate every article ever published on the blog, alongside their tags and publication date. Using the paging functionality would make little sense, as humans will either come _looking for the latest articles_, which are right on the landing page; _or for specific ones_, in which case they'd just use search.

> In removing the infinite scrolling feature, I **force humans to use search** to look for specific content, actually improving their experience!

# Progressive enhancement

This is a crucial aspect of delivering a solid user experience. Progressive enhancement doesn't _just mean_ supporting IE7 and allowing humans to browse your site with JavaScript turned off. It means _improving the performance_ of your site by delivering an experience that's **progressively loaded**.

That means that you **won't do client-side rendering exclusively**. You'll do server-side rendering as well, improving load times and letting the user read some content before JavaScript comes down the wire. If `history` is unavailable, then **you won't cram a hash router in your _very confused_ humans' faces**. It means making reasonable choices and conceding the fact that **your site doesn't have to look the same on every browser**. What's that? It does? Oh so you're rendering HTML in the terminal for `curl` user agents? _Yeah._

If you take progressive steps, **you can vastly improve UX**. Think of a mobile client that's struggling to load your site for the first time around, and hasn't downloaded JavaScript yet. If you don't have any progressive enhancement in place, then chances are subscriptions won't work until JavaScript is fully loaded. If you were to use a good old `<form>` tag, then _you might get away_ with subscribing that human to your mailing list, **rather than failing them miserably**. The same logic applies to a commenting system, for example.

In a similar fashion, make sure you provide adequate fallbacks when performing tricks in JavaScript such as deferring font loading, or image loading, or style loading. You wouldn't want to leave your humans stranded if they have JavaScript disabled!

# Case study: Commenting experience

Before the redesign, the comment system provided an awful experience. Humans were expected to register, confirm via email if they hadn't used a social provider, log in, all before finally being able to share their thoughts. Not logging in wasn't even an option. This was a terrible misread on my part, figuring my blog was important enough for humans to crave an account to link their comments to. It's not.

The redesigned experience **doesn't even try to get the humans to log in**, but rather asks them for their name, email, and their comments, on the spot. They can _optionally add a personal website_, too, as that's customary in most blog-commenting situations in the wild. In order to make their lives a bit better, commenter's metadata is stored on their browser's `localStorage`, in case they want to write more comments.

This simple change tells the human visitor that they're welcome to share their thoughts with the rest of the world immediately. The registration constraint forces humans to shy away from commenting, as it becomes too complicated to post a comment!

Progressive enhancement, using a `<form>` element, does present us with **a hurdle we must jump through** in this case. Spam bots would eat us alive if we blindly allow them to submit comments about cheap viagra, Nike shoes, and Rolex watches. Luckily we could just ask the human to clear a field when JavaScript is turned off, proving he's not a robot. When JavaScript is on the field won't even exist! On the server-side we merely verify that the field isn't filled. If it exists and it has a non-blank value, then it's probably a robot!

```html
<noscript>
  <input value='Promise not to be a bot? Clear this field!' placeholder='This field is only for tricksters...' name='verify' />
</noscript>
```

Of course this _wouldn't stop bots running on headless browsers_ a la [PhantomJS][5], but it's more than enough to get us started in a progressively enhanced orientation.

<div class='md-border'>![nojs.png][6]</div>

The commenting user experience brings up an interesting point, and that's the new approach to subscriptions.

# Subscription model

There used to be two different ways a user would become a subscriber to Pony Foo's emailing list. The simplest way was when they'd **express interest directly**, by submitting their email through the subscriptions form. The alternative was that users would _become subscribers_ when they registered to place a comment.

My problem with the latter is that the human who went through all that trouble to post a comment **maybe didn't want to become a subscriber**, but they would be _forcibly added_ to the mailing list! In this iteration, the experience was slightly modified so that when a human posts a comment they get an email, _amicably inviting them_ to subscribe, and letting them know **what's in it for them** if they do.

Here's an screenshot of the invitation email.

<div class='md-border'>![invitation.png][7]</div>

It's always important to **get out of the way of your humans**. That means I won't spam them whenever they publish a comment. Instead, I should only extend an invitation _the first time_ they post one. It's not like they can't come to the site and subscribe on their own whenever they please!

In this scenario, getting out of the way also means providing _an easy means to unsubscribe_. Being a web worker you've probably seen [all kinds of dark patterns when it comes to attempting to unsubscribe from a mailing lists][8] where people are forced to enter some more information on a web page before they can unsubscribe, or they're not removed unless they agree to something else, click on a button, or worst of all, **they're just met with an error page**.

Providing a high-visibility link to unsubscribe in every email is just as important as doing exactly what you advertise: unsubscribe them and be done with it! Making it harder is only going to make humans angry, and then they might even **complain on the Internet about your dark UX shady practices!**

# Styles and fonts

I've never considered myself good at stylistic choices, although I try super hard to improve in these areas. I've read several books on UX and design, but fonts are well outside of my comfort zone. During the redesign, I wanted the site to have **a stronger identity**, and to that end I've made a few adjustments to the fonts used throughout the site. I selected a playful title font for my headings [_(Cardo)_][9], a playful but highly readable font for the first paragraph in articles [_(Merriweather)_][10], and kept the original font for the rest of the text _(Helvetica Neue)_, as it's mega easy to read.

> Do the new font choices improve UX? I didn't really measure it, but it _sure looks prettier_ to me! I would have to measure to know for sure, or at least, ask around!
>
> Playful fonts can go a long way in showing your visitors **you care about what they see**.

Styles were also slightly adjusted, and although these are mostly design related, _rather than UX related_, some of the choices here also make an impact in the user experience.

For instance, headings used to be underlined but this made them a tad harder to read, so **underlines were removed**. Bullet points used to have `list-style-type: hiragana` characters rather than the regular `disc` value. This led people to believe _I was brilliant, crazy, or maybe presumptuous_, so I decided to go for the safer `disc` bullets.

I decided to go for **a flatter version** of the design I used to have, although I'm still using shades here and there, so I didn't exactly take a pure _"flat design"_ approach. I feel the current design nicely delimits the different pieces of the layout, which in my opinion does provide a cleaner-cut UX. In addition, this choice helped me redesign the buttons into **huge call-to-action rectangles**, which is quite an improvement, as buttons used to be a pretty ugly sight.

Ooh, did I mention **the redesign introduced print styles**? Be aware of the performance implications of adding those styles as a separate stylesheet: [they block rendering][11]. Yes, _even if they're put a `media='print'` tag_. Here's how it looks like.

![print.png][12]

The rest of the styling remained _largely unchanged_, allowing **Pony Foo** to build upon the identity that was established by the original design.

# Links, page titles, and routing

Back when I put together the blog I favored "semantic" URLs over practical ones. This means that I'd have routes such as [/2012/12/25/pony-foo-begins][13], instead of the routing scheme that I picked this time around, [/articles/pony-foo-begins][14]. The former allowed humans to hack away at URLs and find articles in a specific date, but in practice this was **just a variation of the infinite scrolling mechanism** which did more harm than good.

Humans are still allowed to browse articles by date [_(here are the articles published during 2014)_][15], but humans searching the site wouldn't have an interest in searching for articles written on a specific date, and thus they'd have to _know_ about the [/articles/2014][16] URL before being able to visit it.

Similar convenience URLs were introduced, such as [/articles/first][17], [/articles/last][18], and _perhaps the most "useful" one_ [/articles/random][19]. Another UX boost was provided by allowing the user to navigate to [/rss][20], [/feed][21], and [/articles/rss][22], all pointing to the RSS feed at [/articles/feed][23]. This is a concession to the fact that _sometimes people look up the RSS feed by hand_, and when they're unable to find it they are left guessing what the URL may be.

Of course, UX demands that I keep the site working through the URL changes, and that means serving **301 Moved Permanently** responses to queries to URLs _in the old scheme_, as to satisfy human and search engine needs alike.

# Search experience and related articles

The experience in search was largely refactored as part of the redesign, too. Search used to merely involve a naïve RegExp parsing algorithm, returning any article that partially matched the human's query. The improvements introduced were twofold.

Firstly, the search engine itself was improved using a combination of [natural][24], [gramophone][25], regular expressions, and common sense. Of course I could've used a packaged solution like [ElasticSearch][26] or [Lucene][27], but those would've involved more work to get up and running. Besides, they felt like **overkill for a simple one-man blog** operation. If everything else fails, then a regular expression is used _as a last-ditch effort_ to find relevant articles.

Second, the front-facing experience was improved. If you type in a query containing a tag like _"[js]"_, then the engine will know to look for articles tagged `js`. An _experimental feature_ is to return random articles if a query fails to produce any relevant results, as an attempt to **find something else of interest** to the human.

Lastly, _the same search engine_ used by the front-end to perform queries is used in the back-end to compute related articles, so that **relevant articles are suggested** when a human finishes reading an article.

I had longed for related articles and now they're finally here!


  [1]: http://www.amazon.com/Dont-Make-Me-Think-Usability/dp/0321344758 "Don't Make Me Think: A Common Sense Approach to Web Usability, on Amazon"
  [2]: http://www.amazon.com/The-Design-Everyday-Things-Expanded/dp/0465050654 "The Design of Everyday Things, on Amazon"
  [3]: https://i.imgur.com/mZL61Hg.jpg
  [4]: /articles/archives "The archives link to every single article ever published on Pony Foo!"
  [5]: http://phantomjs.org/ "PhantomJS is a Headless Web Browser"
  [6]: http://i.imgur.com/tuvKnoL.png
  [7]: http://i.imgur.com/dfUjZHn.png "Screenshot of the invitation email"
  [8]: http://darkpatterns.org/library/roach_motel/ "Also known as the 'Roach Motel' dark pattern category"
  [9]: http://www.google.com/fonts/specimen/Cardo "Cardo on Google Fonts"
  [10]: http://www.google.com/fonts/specimen/Merriweather "Merriweather on Google Fonts"
  [11]: https://www.nccgroup.com/en/blog/2014/10/does-a-print-css-file-slow-your-site-down/ "Does a print CSS file slow your site down?"
  [12]: https://i.imgur.com/0D5RpY5.png "Print preview for an article on Pony Foo"
  [13]: /articles/pony-foo-begins "Pony Foo begins"
  [14]: /articles/pony-foo-begins "Pony Foo begins"
  [15]: /articles/2014 "Articles published on 2014"
  [16]: /articles/2014 "Articles published on 2014"
  [17]: /articles/first "First article ever published on Pony Foo"
  [18]: /articles/last "Last article published on Pony Foo"
  [19]: /articles/random "A random article published on Pony Foo"
  [20]: /rss "The RSS feed for Pony Foo is at /articles/feed"
  [21]: /feed "The RSS feed for Pony Foo is at /articles/feed"
  [22]: /articles/rss "The RSS feed for Pony Foo is at /articles/feed"
  [23]: /articles/feed "The RSS feed for Pony Foo is at /articles/feed"
  [24]: https://github.com/NaturalNode/natural "NaturalNode/natural on GitHub"
  [25]: https://www.npmjs.org/package/gramophone "gramophone on npmjs"
  [26]: http://www.elasticsearch.org/ "Open Source Distributed Real Time Search & Analytics"
  [27]: http://lucene.apache.org/ "Apache Lucene and Solr"
