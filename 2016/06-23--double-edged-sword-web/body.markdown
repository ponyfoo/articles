# The Web Wields a Double-Edged Sword. ⚔

On the one hand, having a diverse choice of runtimes allows for the web's user base to pick their favorite in terms of customization, tabbed view management, privacy settings, and so on.

That same plurality prevents a single company from controlling the web platform.

There's **no gated access** to web publishing, no app stores, and we have a *seamless "installation" process* that's so lightweight _-- in comparison --_ that we can afford to go through it on every visit to a site.

Competition keeps the web platform a vibrant one, we have so many features and upcoming features being worked on that no single person truly knows the full stack. They may understand the big picture, as we do, or they may be very well versed in a particular aspect of the platform, such as SVG, WebRTC, ServiceWorker, or accessibility.

Not being proprietary, the temptation a single company may have had to introduce breaking changes into the platform every other year never had a place in the web, which still retains the ability to render decades-old websites.

On the other hand, individual browsers with significant market share are company-owned and drive the web in terms of specification and implementation. Sometimes, browsers disagree about features before they're specified, resulting in different implementations. Naturally, bugs can also be a source of divergence across browsers.

Different browsers have different bugs, resulting in libraries like jQuery or normalize.css being highly valued for their convenience when it comes to developer productivity and not having to deal with the nuisance that is cross-browser development. This deficiency in productivity when developing applications that are meant to run on a diverse set of consumer devices and browsers, and not just one or two devices and a single browser in controlled environments _-- such as an enterprise application for companies mandating use of Internet Explorer --_ is for the most part what Eran alludes to when he writes [_"putting together a great [web] experience [is] too fucking expensive"_][tfow].

Let's take a look at the concrete examples of cross-browser development issues I ran into and fixed _(to some extent)_ for Pony Foo.

# CSS Element Outline in Firefox

Pony Foo uses the CSS `outline` property to draw attention to focused elements. The following picture displays three different links and a text input, all of them focused -- under Google Chrome. The article title outline felt particularly clever when I first implemented it. It's a thick dotted outline with an artistic feel to it, yet it was relatively easy to achieve in CSS.

![CSS outline on Pony Foo elements under Firefox][outlines-chrome]

At some point in time, it was pointed out to me that Firefox renders outlines quite differently. My clever article title outline trick didn't look so elegant in Firefox. They have a different opinion on how to render outlines when it comes to multi-line text. I didn't include the input in this screenshot, but suffice it to say that the outline is rendered outside of the input instead of within. Elements with `:before` or `:after` pseudo-content are taken into account while drawing the containing element's outline, even if the [pseudo content itself is absolutely positioned][ff-outline-fix]. **In soviet Firefox, element outlines you.**

![CSS outline on Pony Foo elements under Firefox][outlines-ff]

The most commonly proposed fix to Firefox's `outline` bug when dealing with absolute positioned pseudo content is to ditch `outline` entirely, and to use `box-shadow` instead. While such a fix is mostly a drop in replacement in some scenarios, it's pretty hard to draw a dotted outline using `box-shadow`. Not impossible, you can do all sorts of things with `box-shadow`, but definitely not a trivial thing to implement.

Having neglected this bug for so long, I felt uneasy about switching to `box-shadow` just for 10% of my visitors, so I waited it out. When it came to deciding how to fix it, I decided not to. I took the liberty of using a user-agent detection library _-- I know, **how dare I?** --_ and then created my first `hacks.css` stylesheet.

```css
.ua-firefox * {
  outline: none !important;
}
```

_Oh, well._

At times it's important to remember that websites don't necessarily have to look exactly the same on every browser. Other than web developers, humans have little interest in that. Humans will seldom use different browsers. Unless there are gross differences across browsers, like using entirely different sets of font faces, **humans are not going to care**. We need to learn to let go. User-agent detection might not be the prettiest practice in our toolset, but it is available for us to temporarily patch over these kind of problems. While, yes, no `outline` is bad user experience, a misdrawn `outline` is even worse. I'd rather have no `outline` for Firefox-wielding humans than have the hideous _"a two year old kid drew all over my article's title"_ type of `outline` pictured above.

It's all in the gradual improvements.

# "Dotted" Borders as a Design Tool

Pony Foo is **a colorful place on the internet**. On the home page, you are greeted with headlines for each article, each using different colors. To render this playful aspect of Pony Foo's design, I've experimented for quite a bit before settling for `border-top: 9px dotted $COLOR` rules. This looks super pretty on Chrome! The squares shown below *don't really convey a "dotted" style* when the border is this thick. At a width of `1px`, however, the intermittence of the border does tell a "dotted" story. As the border grows in thickness, though, the "dotted" story dilutes into an unspoken "squared" border style.

![Dotted border style in Google Chrome][borders-1]

At some point, it was pointed out to me that **Firefox and Edge use, well, dots.** Presumably, this was done so that the `dotted` border style is actually "dotted", *regardless of thickness*. Frankly, it just doesn't look as pretty for my use case. It's not the design tool I leveraged. Not anymore. It's something entirely different. Of course, Firefox and Edge are not to blame here. Or are they? Of course not. In any case, there's a discrepancy between Firefox, Edge, and other browsers. One that makes the site look pretty poorly designed on Firefox and Edge.

![Dotted border style in Firefox][borders-2]

I wanted to preserve my "squared" borders in Chrome et al. This was a Firefox/Edge problem, not something that's easily patched with one of user-agent sniffing or deprecating the design for every other browser. Eventually, I settled for an `outset` border style when a Firefox or Edge user agent was detected. This allowed me to introduce minimum changes to my CSS code base, while fixing the not-so-good-looking dotted border style in Firefox and Edge, and preserve my existing styles in other browsers.

```css
.ua-firefox .dc-colored,
.ua-edge .dc-colored {
  border-top-style: outset;
}
```

Again, websites don't need to look the same, _all the way down to pixel perfection_, on every browser. Otherwise, what good is progressive enhancement? Have we been lying to ourselves all this time? **No.** Different browsers. Different purposes. Different looks. It's obvious, once we let go of these terrible mantras.

![Solid border style in Google Chrome][borders-3]

If you're curious about how I do user-agent detection in my Express app, here's a code snippet you can use as a middleware. Once you have that code in your app, all that's left is injecting `req.ua` into your views. In my case, I prefix `req.ua` with `ua-` and inject that into my `<html>` elements.

```js
import { parse } from 'useragent';

export default function sniffUserAgent (req, res, next) {
  const nonalpha = /[^a-z]/g;
  const agent = req.headers['user-agent'];
  const { family } = parse(agent);
  const ua = family.toLowerCase().replace(nonalpha, '');
  req.ua = ua;
  next();
};
```

*Oh, and you'll need to `npm i -S useragent`.*

# Tables in Firefox and IE

Pony Foo uses `display: table` in quite a few places. One caveat when rendering tables in Firefox and IE is that `max-width` rules can be ignored by the rendering engine if they are contained within tables without [`table-layout: fixed`][tablefix]. As it turns out, [the specification][tablespec] is on Firefox's and IE's side for this particular inconsistency. Who is correct isn't important. What should be important is that browsers put consistency across themselves first. Consistency across browsers should be the most sought-out aspect of feature development.

Yet, consistency is consistently not attained. It's sometimes more interesting to work on new features than to fix existing issues on the underlying platform, some of these inconsistencies have been neglected for years. Reporting them time and again to browser vendors could mitigate their impact _(by having them patch the features)_, to the extent that they're willing to prioritize those fixes over newly developed features.

Oh, look! A composite worker! \**Rushes to drool all over it.*\* 🤓💦

# Line Height in Firefox

This one reminded me that even when we leverage libraries meant to ameliorate the effect of inconsistencies across browsers, some of them might still slip through the cracks. In this particular situation there were *a few text inputs that were misaligned* with their accompanying buttons roughly by one pixel. Upon looking into the issue, it turned out that Firefox had a `line-height` of `19.45px` whereas Chrome and others used `18px`.

Presumably, this issue arised from differences in how different browsers calculate the default `line-height` value for elements. I didn't care enough to dig deeper into the issue than that. *Sometimes, this is okay too.*

# Background Gradients in Safari

Headlines on the home page use a gradient between the background color and transparent to overlay a preview image into the content without introducing a hard break between the content and the image. This saves room and allows me to introduce a revealing effect by adding a `transition` on `opacity` when the element is moused over.

![A background gradient in Google Chrome][gradient-chrome]

Some helpful soul sent me an email telling me that the gradient was "broken" in Safari. Upon inspection, it turned out that Safari dislikes the `transparent` aspect of the gradient, for some unspoken reason. Luckily, a fellow Stack Overflow user noted that using [`rgba(0, 0, 0, 0.00001)`][gradientfix] instead of `transparent` would get the job done and avoid rendering issues.

Weirdly enough, they ran into the issue under Chrome and not Safari, whereas I was able to reproduce it in Safari but not in Chrome.

A different time, a different king.

![A background gradient in Safari][gradient-safari]

Okay, I promise there's only one bug left for us today.

# Relative `<svg>` URL Strokes and the Phantom of Animation

This was one of those *"I can't believe this ever worked"* type of bugs. I noticed that the following `<svg>` graph wasn't rendering the page views plot line on Safari. After running into a wall for several minutes, I figured out that the `stroke` for that plot line was a relative URL: `stroke: url(#sg-pageviews-gradient)`. This was being set on the CSS stylesheet. A URL in an stylesheet is relative to that stylesheet.

[![A graph of subscribers and page views over time][graph]][sub]

I have no idea why this *did* work on Chrome, but moving the `stroke` property away from the CSS stylesheet and into JavaScript code, assigning it to the `<path>` element as an attribute, fixed the issue. *Inconsistency across browsers, again.*

At the same time this bug kind of reminded me of [the weirdest bug I've ever come across][svgbug], also related to `<svg>`. In that article, I go on to describe a rendering bug that depended on so many conditions that **I completely lost my sanity for a whole day**.

# Words on RSS

RSS was sort of an special kind of "bug" I had, where I had simply forgotten to include style information in my feed items' HTML. This caused a plethora of rendering inconsistencies between the actual site, the feed's contents, and what humans expected to see. Always inline style information in your RSS feeds.

Sure, readers may ignore them entirely, but at least you've tried. Pony Foo Weekly renders [quite decently on Feedburner][fb], for example.

# The Fickle Nature of User-Agent Detection

All of the bugs presented so far go to show two things.

One, **browsers are still as inconsistent as ever, if not more so.** There is no end in sight for this plight, as browsers keep on piling more features and knowledge is further specialized and bucketed so that no single person understands the whole thing in depth. This is the biggest source of friction when it comes to developing applications that simply work across browsers: some inconsistencies may be ironed out, many others will be take their place. Web development is going to stay cumbersome for the foreseeable future.

Two, and this is not something that follows from the bugs I found and patched, but more of a realization. *Yes.* User-agent sniffing is evil, bad, unsophisticated, inelegant, an oft-overused, misleading band-aid. **But,** user-agent sniffing does serve a *purpose*. Snuffing out browser-specific bugs that cause rendering anomalies *-- while a fix is being worked on or thought up --* is laughably easy when you allow yourself to leverage user-agent information, despite the heaps of advise on how user-agent detection is evil. Under the right circumstances, even `goto` and `On Error Resume Next` have useful applications.

That's not to say you should patch all the things over with a user-agent-sniffing self-`!important` rule and call it a day. It's a tool, like any other. You may use it, but you should always ping your best judgement first. It may be a good idea to have a [`hacks.css`][hacks] stylesheet where you keep track of every single awful hack your site leverages, so that you can readily identify them all, and slowly pluck them away from your site as you find more *engineeringly-sound* solutions.

Preferrably, ones that don't include the made-up word: *engineeringly*.

# So, Native Mobile Apps?

Well, yes. *If all you care about is optimal developer productivity,* you'll never ever attain that in web development. That's the price you pay for having an application that runs on the world's largest runtime platform, that's *able to fend off against half a dozen rendering bugs and still display a somewhat usable experience* to the humans visiting your corner of the internet. That's the price you pay for not having gated access, for not having to deal with arbitrary rules imposed upon you whenever you publish an update to your application, for having a choice when it comes to where and how your application can run.

And, no. **Do not give me that bullshit about Android or iOS being easier to develop for because they're single platforms.** Nothing is stopping you from sniffing user-agents, developing for a single browser, and blocking all the rest with a big fat *"Your browser is not supported"* notice, as bad as that'd be. How is that any different from developing for a single mobile operating system? Well, it's different in that at least on the web you don't have to put up with App Stores, a proprietary API, breaking changes, or licensing fees.

So, native mobile apps? *No.* **Not fucking native.**

[tfow]: https://hueniverse.com/2016/06/08/the-fucking-open-web/ "The Fucking Open Web on hueniverse.com"
[outlines-chrome]: https://i.imgur.com/VW7FAUH.png
[outlines-ff]: https://i.imgur.com/N8gGNm8.png
[ff-outline-fix]: http://stackoverflow.com/a/10662977/389745 "CSS “outline” different behavior behavior on Webkit & Gecko"
[borders-1]: https://i.imgur.com/jIBNFpC.png
[borders-2]: https://i.imgur.com/AAKTPPX.png
[borders-3]: https://i.imgur.com/vAJQtI8.png
[gradient-chrome]: https://i.imgur.com/FkkYkuT.png
[gradient-safari]: https://i.imgur.com/7micfRu.png
[graph]: https://i.imgur.com/aCtnyXZ.png
[sub]: /subscribe "Get Pony Foo in your inbox!"
[tablefix]: http://www.carsonshold.com/2014/07/css-display-table-cell-child-width-bug-in-firefox-and-ie/ "CSS display:table-cell child width bug in Firefox and IE"
[tablespec]: https://www.w3.org/TR/CSS2/visudet.html#the-width-property "Content width: the 'width' property"
[gradientfix]: http://stackoverflow.com/a/11829561/389745 "CSS3 gradient rendering issues from transparent to white"
[hacks]: https://github.com/ponyfoo/ponyfoo/blob/ea2a97608d54eb47331510fe16966a11255f253f/client/css/hacks.styl "hacks.styl in ponyfoo/ponyfoo on GitHub"
[fb]: https://feeds.feedburner.com/ponyfooweekly "Pony Foo Weekly on Feedburner"
[svgbug]: /articles/weirdest-bug-ever "SVG and the DOM, or “The Weirdest Bug I’ve Ever Encountered” on Pony Foo"
