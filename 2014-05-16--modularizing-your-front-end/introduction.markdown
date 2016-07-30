[![contra.png][2]][1]

I've also discussed [campaign][3], where I modularized [email sending and templating][4]. That was really mostly a port of what was already in Pony Foo's code base, and thus [open-source][5], but written in a more portable way that could be used across other projects or efforts. In that occasion I explained the motives for developing the library as well, which mostly revolved around having a reusable email sender that was easier to bring into a Node application than the existing alternatives.

In this article I'll **describe the development process, choices I've made, and contents of my latest open-source** projects. In lieu with Pony Foo's humble, [_humble beginnings_][6], I've decided to continue on the path set forth back when I wrote [campaign][3], and [re-do Pony Foo from scratch][7]. The difference is that this time I've been putting more of a **focus on modularity**, basing off of what I've learned over the last year _(and from the glaring mistakes I made implementing the original blog engine)_.

* [`flexarea`][8] is a tiny library that helps humans resize `<textarea>` elements
* [`hint`][9] is a pure CSS implementation of the tooltips you can see today on **Pony Foo**
* [`taunus`][10] is a _micro MVC_ framework **I feel extremely excited about**

Caution: This article contains words about [Browserify][11], [`npm run`][12], [Gulp][13], framework-less JavaScript, and ponies. **Reader discretion is advised.** Also, not all of it might make sense.

[1]: https://github.com/bevacqua/contra
[2]: https://raw.github.com/bevacqua/contra/master/resources/contra.png
[3]: https://github.com/bevacqua/campaign
[4]: /2014/01/07/email-sending-done-right
[5]: https://github.com/bevacqua/ponyfoo
[6]: /2012/12/25/pony-foo-begins
[7]: https://github.com/bevacqua/ponyfoo/tree/redo
[8]: https://github.com/bevacqua/flexarea
[9]: https://github.com/bevacqua/hint
[10]: https://github.com/bevacqua/taunus
[11]: https://github.com/substack/node-browserify
[12]: http://substack.net/task_automation_with_npm_run
[13]: https://github.com/gulpjs/gulp
