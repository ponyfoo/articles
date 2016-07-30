Pony Foo is a web development blog. As such, visitors are mostly privileged web developers using macOS and Google Chrome. Here are some historical stats, pulled from Google Analytics:

- Over 37% of visitors are on a Mac
- Under 34% of visitors are on Windows
- Over 72% use Google Chrome
- Roughly 10% are on Safari, and another 10% are on Firefox
- IE and Edge, combined, amount to barely over 1% _-- at 0.84% and 0.3%, respectively_

The fact that people were seldom using anything other than Chrome gave me a good cover story for not pouring any effort into resolving long-standing _(albeit minor)_ CSS rendering issues in Firefox and Safari. However, 20% of visitors were getting a less than ideal experience, which ultimately wasn't something I could scoff at. Furthermore, it didn't speak well of me to have these blatant rendering bugs lying around in my blog when visited through popular non-Chrome browsers. It wasn't tidy.

In this article I'll go through the bugs I fixed for Safari and Firefox visitors, outlining the problem, possible solutions, and how I ended up fixing them. As it turned out, not all of the bugs had a simple fix, and that's the largest reason why I neglected fixing them for so long, as we'll go over while analyzing each specific bug.

Besides the specific instances of cross-browser issues I detected and fixed, we'll also be discussing the impact of having to deal with heaps of different rendering engines, when it comes to web development in general as opposed to mobile application development. In that regard, this article follows up on what Eran Hammer wrote about the web last month: ["The Fucking Open Web"][tfow]. While under a belligerent light, Eran's article highlights the challenges of web development when it comes to building a polished web application that performs well under the most popular runtimes -- and there's a lot of them, what with Chrome, Firefox, Safari, IE, Edge, Mobile Safari, and _(let me know when to stop)_ developer editions such as Chrome Canary, Firefox Nightly, Rockmelt, Electron-based browsers, crawler bots, RSS readers and other niche browsers -- all of them serving one purpose or another, and application developers being judged as the ones at fault when a cross-browser API inconsistency causes our websites to fail under a particular set of conditions.

Here's one of those niche browsers _-- which by the way, looks freaking gorgeous! 🤓_

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Hyperterm updates:<br>- tab activity indicators<br>- built-in URL handling (no shell changes required, unobtrusive) <a href="https://t.co/jyqAsDYsF8">pic.twitter.com/jyqAsDYsF8</a></p>&mdash; Guillermo Rauch (@rauchg) <a href="https://twitter.com/rauchg/status/748623889446625280">June 30, 2016</a></blockquote>

[tfow]: https://hueniverse.com/2016/06/08/the-fucking-open-web/ "The Fucking Open Web on hueniverse.com"
