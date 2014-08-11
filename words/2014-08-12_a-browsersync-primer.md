# A BrowserSync Primer

[BrowserSync][1] is a browser testing tool, similar to [LiveReload][2]. It also synchronizes across browsers and is going to provide [HTML injection][3] in the very near future, alongside with the CSS and JavaScript injection it already features.

[![browsersync.png][4]][1]

This tool enables rapid continuous development by saving you the trouble of refreshing the view you are debugging whenever you make changes to the JavaScript or CSS code.

You won't really be able to grasp how valuable this is until you give it a try for yourself. This primer aims to encourage you to do just that!

In this article I'll show you how to use it with your favorite tools. Grunt, Gulp, `npm run` and `nodemon`.

[1]: http://www.browsersync.io/ "Time-saving synchronised browser testing."
[2]: http://livereload.com/ "A happy land where browsers don't need a Refresh button"
[3]: https://twitter.com/BrowserSync/status/498161493418840064
[4]: https://cloud.githubusercontent.com/assets/934293/3879487/18b2f866-2179-11e4-8098-3467f24e5061.png

Modern web development is burdened with a few pauses, such as saving files, running build tasks, refreshing the browser, besides the usual **thinking and refactoring** that you were used to. [Continuous development helps you][8] bridge the gap by shortening those pauses, or at least automating them in such a way that you don't have to **manually take action** in between those pauses.

It doesn't matter what tools you use, as long as those tools get you one step closer to true continuous development, eliminating the friction in between your changes and the result displayed on the browser.

- Using a reasonable text editor _([Sublime][9], [Atom][10], [vim][11], etc)_ you can **auto-save** when your tabs lose focus
- Using [watch tasks][5] you can avoid restarting your build flow whenever **source code changes**
- Using [Nodemon][7] you can avoid running `node app` whenever **server-side code changes**
- Using [BrowserSync][3] you can avoid hitting <kbd>F5</kbd> on the browser whenever **client-side code changes**

The one _unfortunate drawback_ to [BrowserSync][3] is that currently its documentation is quite lacking. Rather than explain and document what the API does and what properties, options and methods there are, you just get [a few use cases][4] and the example code that goes with them.

Hopefully, their documentation will improve in the future. For now, you can rely on the guides in this article.

## Using Grunt

If you're using Grunt in your builds, you can use the [`grunt-browser-sync`][1] package. I'll go ahead and assume you're [using `load-grunt-tasks`][2] to load your tasks automatically.

```shell
npm install grunt-browser-sync --save-dev
```

Typically you'll want the proxy behavior. In this case you can mount BrowserSync to listen on a port, but proxying an existing app. This will use your own server to respond to requests, but it'll patch it with the appropriate script injection code, which is used on the client-side to reload CSS and JS through BrowserSync.

Besides configuring the server, BrowserSync needs to understand what files changes are going to trigger updates, and you can use a globbing pattern for that. Refer to the code snippet below for an example on how to configure `grunt-browser-sync`.

```js
browserSync: {
  dev: {
    bsFiles: {
      src: 'public/**/*.{js,css}'
    },
    options: {
      proxy: 'localhost:3000'
    }
  }
}
```

When you are [using watch tasks][5], you are expected to set the `watchTask` option to `true`.

## Using Gulp

When using Gulp, the folks at BrowserSync recommend [using the `browser-sync` API directly][6], rather than a plugin, and for good reason. The code is pretty self-explanatory, simplicity beats over-engineering.

```js
var gulp = require('gulp');
var browserSync = require('browser-sync');

gulp.task('browser-sync', function () {
  browserSync({
    proxy: 'localhost:3000',
    files: ['public/**/*.{js,css}']
  });
});
```

Doing the same with `npm run` is even easier.

```shell
browser-sync start --proxy localhost:3000 --files "public/*"
```

In my experience, though, at this point you are better off using the API within your Node application.

## Using in a Node application

The upside in running [BrowserSync][3] directly in your app is that you won't bash your head against the wall when attempting to make it play nicely with tools such as [`nodemon`][7], which constantly restart the server that BrowserSync is proxying.

To ensure that the server is always up when BrowserSync attempts to uplink, I find that the best approach is to configure it as follows.

```js
var express = require('express');
var browserSync = require('browser-sync');
var app = express();
var port = process.env.PORT || 3000;

app.listen(port, listening);

function listening () {
  browserSync({
    proxy: 'localhost:' + port,
    files: ['public/**/*.{js,css}']
  });
}
```

Now you can run `nodemon` and BrowserSync will work even across restarts. Yay!

```shell
nodemon app.js
```

_It's time to focus on what really matters._

[1]: https://github.com/shakyshane/grunt-browser-sync "shakyshane/grunt-browser-sync on GitHub"
[2]: /2013/11/13/grunt-tips-and-tricks "Grunt Tips and Tricks"
[3]: http://www.browsersync.io/ "Time-saving synchronised browser testing."
[4]: http://www.browsersync.io/docs/grunt/ "BrowserSync + Grunt.js"
[5]: https://github.com/gruntjs/grunt-contrib-watch "grunt-contrib-watch on GitHub"
[6]: http://www.browsersync.io/docs/gulp/ "BrowserSync + Gulp.js"
[7]: https://github.com/remy/nodemon "Monitor for any changes in your application and automatically restart the server"
[8]: /2013/09/26/continuous-development-in-nodejs "Continuous Development in Node.js"
[9]: http://www.sublimetext.com/ "The text editor you'll fall in love with"
[10]: https://atom.io/ "A hackable text editor for the 21st Century"
[11]: http://www.vim.org/ "Highly configurable text editor"

[browsersync grunt gulp npm nodemon]
