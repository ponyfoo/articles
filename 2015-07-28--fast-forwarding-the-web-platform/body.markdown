# On _"Web vs. Native"_

This is the ultimate argument, right? The biggest issue here is **misinformation**. While, yes, obviously, the web platform is not as well-suited as native mobile apps in some ways -- you probably wouldn't want your mobile experience of realtime video chats to be web-based, because it might be terribly tricky to get that right, there's also a lot of this bandwagon effect where somebody says something and _almost everyone else just nods their heads in agreement_.

### Instant Articles

A great example is one of Facebook's recent features being unveiled -- the so-called _"Instant Articles"_ feature. Instant Articles is fancy marketing speak for Facebook crawling news sites that have paid **a hefty fee** for Facebook to do so, storing their articles in a standarized data structure of their choosing, rendering a "faithful-enough" version of the media site's article, and presenting that _"view"_ when humans try to access one of the media site's pages, **instead of the actual website**.

[![Instant Articles demo page][1]][2]

This is useful because media sites are the bane of the web. They sometimes take double-digit seconds to load, incur several MB worth of downloads. Thus, data charges go off the roof, not to mention when the human is relying on roaming and they have to pay **€10 for 200MB worth of content**, essentially <mark>**paying roughly €2 per page view**</mark>. Of course, you could try and pin that on the mobile networks, but they've been charging insane amounts of money to roamers for over a decade, and we've made it a problem only recently.

With Instant Articles, media sites are **taking the worst possible step in the right direction**. They're effectively saying:

>  Our site is garbage, **BUT**, if you visit us through Facebook, well... then they'll solve our embarrassing performance issues on our behalf. If you're on mobile, make sure to visit our site through Facebook, otherwise it might take you seven seconds to see this page.
>
> _-- Implied message from every media site on Instant Articles_

**This screams of missed opportunity**. The notion of investing _tons of money_ in ensuring that Facebook solves your performance problems when people visit your site through Facebook **(and only in that case)**, is as terrible of a value proposition as any. You would be way better off hiring a performance consultant like [Tim Kadlec][3] to fix your web performance issues for good.

This brings me to my last point, the _"let's blame our problems on the web"_ approach.

### Blaming slowness on the web, The Verge way

If you've gotten this far, chances are you haven't missed The Verge's article on how ["the mobile web sucks"][4]. In their article they basically blame the web on the fact that dozens of media deals and analytics human-data-siphoning that goes on when the page is first loaded. This is well summarized by [Les Orchard][5] in their article.

> Holy crap. It took over 30 seconds. In the end, it fetched over 9.5MB across 263 HTTP requests. That's almost an order of magnitude more data & time than needed for the article itself.
>
> [![The Verge's web sucks][7]][6]
>
> _Head over to [blog.lmorchard.com][6] for their full analysis._

A couple hundred requests, almost 10MB in data transfers, how the hell do you expect the mobile web to not suck in **those conditions**? That page view just costed you €2 in data roaming charges. You can buy a couple of physical _WIRED_ magazines for that cost.

Just because **your website is slow** doesn't mean the web platform is at fault. Any platform you misuse is going to slow down to a crawl. Instead of spending time and energy figuring out who's to blame for your slow site, try and do something about it. Maybe you don't need to talk to 22 different analytics providers on every page load, **maybe a single service can broadcast events on your behalf** to all of those. Maybe you don't need to <mark>load 1.3MB worth of images on page load</mark>, maybe you could defer most of those, most of the ones below the fold, _until after the text content has loaded_.

_"Dis-services"_ like t.co don't do much in the way of helping, either.

It's time for media sites to begin acting as **media** sites and stop acting as advertisement platforms that only care about cramming more ads into their pages. As long as that keeps happening, their websites will continue to be as slow as snails.

It's time we [stop breaking the web][8], we start [building performance into our sites][9], and we start caring about the fundamentals of the web platform. Otherwise, how can we expect to advance the more sophisticated aspects of its development?

### Back to the Classical Elements

Consider this. If `<select>`, `<input>`, `<textarea>`, and client-side validation were to start being developed today, what would they look like? Would they look like impenetrable <mark>**black boxes**</mark> with _arbitrary constraints_ set forth by each individual user agent? Or would they be <mark>modular</mark>, extensible, easy to integrate into your designs **without obscure hacks**, and as flexible as any userland component that's developed on top of them? If your answer is anything different than _"exactly how they are today"_, why on earth are we not doing anything to change them? They're still the most ubiquitous form of user interaction on the web _(besides links)_, and we've been resting on our laurels for far too long.

I've always found standards teams to be of a similar nature, but I'll admit that's mostly because of my ignorance. I have little idea about how they operate, how a proposal moves through the ranks, how much _effective "power"_ an average developer's voice has _(from what I hear, it's mostly browser vendors that get a say)_, and how the effort to implement certain features across browsers is coordinated.

That being said, **I'd love for more people to get involved in the standards process**, asking for these things everyone seems to be taking for granted even though we've _been developing hacks around them for years_.

I'll leave you with a video you might want to see. It's [Chris Heilmann talking][10] about: <mark>**"Advancing the Web Without Breaking it"**</mark>.

[![Christian Heilmann talking about these issues][11]][10]

[1]: https://i.imgur.com/lG35RuG.jpg
[2]: http://instantarticles.fb.com/ "Instant Articles on Facebook"
[3]: http://www.timkadlec.com/ "Tim Kadlec's personal website"
[4]: http://www.theverge.com/2015/7/20/9002721/the-mobile-web-sucks "The mobile web sucks, according to The Verge"
[5]: https://github.com/lmorchard "lmorchard on GitHub"
[6]: http://blog.lmorchard.com/2015/07/22/the-verge-web-sucks/ "The Verge's web sucks"
[7]: https://i.imgur.com/ywos2Y8.png
[8]: /articles/stop-breaking-the-web "Stop Breaking the Web on Pony Foo"
[9]: http://timkadlec.com/2015/05/choosing-performance/ "Choosing Performance by Tim Kadlec"
[10]: https://vimeo.com/134279371 "Christian Heilmann – Advancing the web without breaking it? – Beyond Tellerrand Düsseldorf 2015"
[11]: https://i.imgur.com/xk3gppe.jpg
