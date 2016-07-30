Then there's building the package for human consumption. That's where things get really cute. You have [CommonJS modules][1] and [browserify][2], AMD modules with [Require.js][3], plain-old raw JavaScript, and a bunch of other "input" formats. There is plenty that can be done in that field, and plenty is done indeed. The output isn't that complicated, but everyone expects you to provide at least the minified (and unminified) code for your module tighly packaged in a single `.js` file.

Solutions such as [Grunt][4] or [yeoman][5] kind-of solve these problems, but leave you but a bunch of code that you might have no idea how it works.

[![paqui.png][7]][6]

[Paqui][6] solves these issues for us, without leaving undesired artifacts behind. That's the nice thing of being **tailored for open-source front-end package development**! In this article we'll examine how to make use of Paqui in our package development workflow, and learn how we can extend its behavior, _if we need to_.

[1]: http://wiki.commonjs.org/wiki/Modules
[2]: http://browserify.org/
[3]: http://requirejs.org/
[4]: http://gruntjs.com/
[5]: http://yeoman.io/
[6]: https://github.com/bevacqua/paqui
[7]: https://i.imgur.com/AksDJZW.png
