Of course, it all depends on context. Nobody likes spending their time dealing with GitHub issues, acting as a support help desk for an open-source project. I've managed to categorize most issues on [dragula][2] as <mark>bug</mark>, **support**, or _feature-request_. That's, for the most part, the order in which I want to prioritize a resolution for each of them.

## Bugs

In the case of [bugs][3], you obviously want to get them fixed and out of the way as soon as possible. This ensures that continued development isn't affected by the bug, and helps avoid back-pressure when **all of sudden** you have 10-20 subtle bugs that end up making your project all but unusable.

Before `dragula` became popular I did most of the bug-fixing myself, because I really cared about fixing them but also because **it's really hard** to get people to fix bugs for you! No wonder, nobody likes it. Nowadays I can get away with asking people if they can take a look, and honestly? Nothing beats that! I'm seeing people get more involved, sending me more pull requests to make dragula better, and in the end, everyone wins.

A lot of the time though, people won't be willing to spend their time working on your projects, and that's okay _(as long as they don't **demand** that you do it, and start spinning insults at you)_. Just remember to be nice, without the bug reports you probably wouldn't have any idea of many of the things that can go wrong with your project! 

## Questions

Dragula gets a ton of questions. Out of ~90 closed issues so far, [~30 were support questions][4]. Questions are easy to close, but answering questions all day can be time consuming. Ideally, when a community starts forming around a project, other people start answering questions. In the beginning, though, you're probably the one with most of the answers. Some questions are elaborate, and some are pretty basic or misguided. You should strive to be nice to everyone, though. They can become valuable contributors to the project **if they feel welcome**.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">What’s wrong with open source software communities in one DM. &#10;&#10;Let’s change this. <a href="http://t.co/ydCkWae3sl">pic.twitter.com/ydCkWae3sl</a></p>&mdash; Jesse Pollak (@jessepollak) <a href="https://twitter.com/jessepollak/status/622536818060201985">July 18, 2015</a></blockquote>

They had the courage to ask you for help, so don't be patronizing and answer their questions. You might learn a thing or two too. For instance, I used to do this kind of thing in demos:

```html
<div id='left'></div>
<div id='right'></div>
```

```js
dragula([left, right]);
```

I used to get **tons** of questions about how to pass in elements to `dragula`, and I was baffled. To me, it was obvious that this was just an example, and that you shouldn't ever rely on `window[id]` bindings in the real world. Most people didn't take it this way, though. It rather seemed as if I was encouraging the use of `id` references on `window`, something I definitely didn't want. Up until the point where I replaced the examples with `document.querySelector`, just to be more explicit, people kept asking questions about this. I learned that while it might look "prettier" in the demos, it also confuses consumers, and that just defeats the whole purpose.

Documentation is particularly important in open-source, and great documentation really helps you get less questions in your open-source repos. Similarly, try and take away potential documentation improvements based on the questions you're getting most often; chances are, an update to your documentation might help you get less questions like those in the future. If anything, you can just point them to the documentation and tag their question **please-read-the-docs**. But *don't be mean* about it!

In general, you just have to remember that while you probably wrote most of the characters in your code base, documentation, examples, issues, and changelogs, people don't have the time to sift through all of it. While some effort is expected, at least when it comes to reading the documentation, sometimes people don't know where in the documentation they should be looking.

## Feature Requests

Dragula doesn't get a lot in the way of feature requests. There's the occasional request to support one thing or the other, but most of it seems to be covered by the existing API. That's neat. In most projects I see feature requests being the kind of issue that stays open for the longest time, and that makes sense. Odds are, the maintainers don't necessarily need to implement the feature, the project already does everything they need for them! This is an area where you'll <mark>need to be able to stand your ground</mark>, as you'll probably have to **say no** to many requests. As time goes by, you'll get a better idea of the kinds of features you want to be implementing and the kind of features you don't.

For the most part, I **try to figure out what the common use case is**, and if that makes sense to me, I may implement that. This is valuable because you might initially dismiss a request as _"too narrow, not enough use cases"_, but there's also the potential to raise your head and find a broad solution that encompasses more use cases, which you might find more appealing too.

As an example, I got a request to [add drag handles to `dragula`][6] early on, allowing somebody to initiate drag events from just a child, instead of clicking anywhere in the element. I didn't need this at the time but I surely would at some point, and it's a popular feature to have, so I decided to go ahead and implement it. I didn't use the *selector-based approach* that was suggested though, because that'd only support the specific use case: drag handles matching a CSS selector. I looked further and added an argument to `moves`, the original click `e.target`. This way, people can use the click target to figure out whether they want to allow the click to translate into a drag event or not, giving them more power and flexibility.

Always consider feature requests in general terms, try to look beyond what they're asking and figure out if there's a better solution that **may address potential future feature requests**!

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][1]._ Remember to **subscribe** if you got this far!

[1]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
[2]: https://github.com/bevacqua/dragula/issues "bevacqua/dragula issues on GitHub"
[3]: https://github.com/bevacqua/dragula/labels/bug "bevacqua/dragula issues labeled bug on GitHub"
[4]: https://github.com/bevacqua/dragula/labels/support "bevacqua/dragula issues labeled support on GitHub"
[5]: https://github.com/bevacqua/dragula/labels/feature-request "bevacqua/dragula issues labeled feature-request on GitHub"
[6]: https://github.com/bevacqua/dragula/issues/20 "Drag Handles issue on bevacqua/dragula on GitHub"
