# Modularizing Your Front-End

In the past [I've wrote][1] about a small alternative to [`async`][5], named [`contra`][2], which is _barely over 2kb_, and has the browser at its heart. It comes with the usual suspects, allowing you to craft asynchronous flows in series or concurrently, as well as a few functional methods, an event emitter implementation, and a queue which powers most of the library.

[![contra.png][3]][2]

I've also discussed [campaign][8], where I modularized email sending and templating. That was really mostly a port of what was already in Pony Foo's code base, and thus [open-source][7], but written in a more portable way that could be used across other projects or efforts. In that occasion I explained the motives for developing the library as well, which mostly revolved around having a reusable email sender that was easier to bring into a Node application than the existing alternatives.

In this article I'll **describe the process and contents of three of my latest open-source** projects. In lieu with Pony Foo's humble, [_humble beginnings_][9], I've decided to continue on the path set forth back when I wrote [campaign][7], and [re-do Pony Foo from scratch][10]. The difference is that this time I've been putting more of a **focus on modularity**, basing off of what I've learned over the last year _(and from the glaring mistakes I made implementing the original blog engine)_.

- [`flexarea`][12] is a tiny library that helps humans resize `<textarea>` elements
- [`hint`][11] is a pure CSS implementation of the tooltips you can see today on **Pony Foo**
- [`taunus`][13] is a _micro MVC_ framework **I feel extremely excited about**

Caution: This article contains words about [Browserify][14], [`npm run`][15], [Gulp][16], _isomorphic view rendering_, framework-less JavaScript and ponies. **Reader discretion is advised.**

[2]: https://github.com/bevacqua/contra "Asynchronous flow control with a functional taste to it"
[3]: https://raw.github.com/bevacqua/contra/master/resources/contra.png
[1]: /2014/03/07/a-less-convoluted-event-emitter-implementation "A Less Convoluted Event Emitter Implementation"
[5]: https://github.com/caolan/async "async utilities for node and the browser"
[6]: http://blog.ponyfoo.com/2014/01/07/email-sending-done-right "Email Sending Done Right"
[7]: https://github.com/bevacqua/ponyfoo "Pony Foo on GitHub"
[8]: https://github.com/bevacqua/campaign "Compose responsive email templates easily, fill them with models, and send them out"
[9]: http://blog.ponyfoo.com/2012/12/25/pony-foo-begins "Pony Foo Begins"
[10]: https://github.com/bevacqua/ponyfoo/tree/redo "ponyfoo@redo on GitHub"
[11]: https://github.com/bevacqua/hint "Awesome tooltips at your fingertips"
[12]: https://github.com/bevacqua/flexarea "Pretty flexible textareas"
[13]: https://github.com/bevacqua/taunus "Micro MVC Framework"
[14]: https://github.com/substack/node-browserify "browser-side require() the node.js way"
[15]: http://substack.net/task_automation_with_npm_run "Task automation with npm run"
[16]: https://github.com/gulpjs/gulp "The streaming build system"



[isomorphic ponyfoo taunus browserify npm-run gulp]
