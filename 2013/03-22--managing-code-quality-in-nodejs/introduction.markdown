This article is mostly a follow up on the [CI][1] post. I'll describe how [Grunt][2] helped me change the _test_ and _build_ processes used in this blog's [engine][3].

> Before using Grunt, I didn't really have a **real** build process. _Sure_, `git push heroku master` _triggered a build_ on their end, but I didn't control any of it, all I did was `node app.js`.

Similarly, my [Travis-CI hook][4] just made sure there weren't any conflicts with my npm packages. I could do _better_.

[1]: /2013/01/18/continuous-integration-and-automated-deployments
[2]: gruntjs.com
[3]: https://github.com/bevacqua/ponyfoo
[4]: https://travis-ci.org/bevacqua/ponyfoo/builds
