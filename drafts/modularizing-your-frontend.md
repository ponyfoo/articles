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

If you've glanced over the projects, you'll realize that both [`flexarea`][1] and [`hint`][2] are quite simple. However, they are also highly reusable, and it'd be interesting to share 

# Flexible `<textarea>` elements

Flexarea just provides a little grip you can drag to resize a `<textarea>`, and nothing else. This is how it currently looks like.

[![flexarea.png][5]][1]

The API is fairly simple. `flexarea` is a function, it takes a DOM node, surrounds it with a wrapper element, and appends the "grip" element to the wrapper. The grip is what's on the bottom of the `<textarea>` shown in the picture above. As you can [see in the repository][6], I used a Common.JS module for the component, meaning that I need Browserify in a compile step to try the component out, as well as during releases. For local development I deeply recommend you to familiarize yourself with [`npm link`][16] if you're a package author, as it'll **considerably speed up** your development productivity.

## Gulp, `npm run`, Whatever

There's a very similar [Gulpfile][8] to the one I detailed in [My First Gulp Adventure][7]. The "big" difference in the Gulpfile is that I run Browserify, given I'm using Common.JS in my source code. Browserify takes a little dancing in Gulp. Here's how it ends up looking like, after removing some of the cruft, getting us just the core idea.

```js
var gulp = require('gulp');
var streamify = require('gulp-streamify');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');
var source = require('vinyl-source-stream');
var browserify = require('browserify');

gulp.task('build', function () {
  return browserify('./src/flexarea.js')
    .bundle({ debug: true, standalone: 'flexarea' })
    .pipe(source('flexarea.js'))
    .pipe(gulp.dest('./dist'))
    .pipe(streamify(rename('flexarea.min.js')))
    .pipe(streamify(uglify()))
    .pipe(gulp.dest('./dist'));
});
```

The code above would compile my source code into [a UMD module][9] (also read [Writing Modular JavaScript][10] by [Addy Osmani][11] while you're at it). That is pretty awesome. You write CJS code, with all the benefits that that implies, such as no implicit globals and no [IIFE] hell, and you get a module that can be run and included pretty much everywhere you can run JavaScript! Except there's no way in hell compiling a single file into two distributions should involve around 15 lines of code. Aren't you using a library to simplify things? Compare that to using `npm run` and the [`browserify` CLI][13].

```js
browserify src/flexarea.js -s flexarea -o dist/flexarea.js -d
browserify src/flexarea.js -s flexarea -o dist/flexarea.min.js -t uglifyify
```

_Anyways._ Lately I've been leaning more and more into [the "whatever" camp][14]. I find that it's much leaner to just use `npm run`, and [putting together a build file][15] is incredibly faster if you aren't dealing with a formal build system such as Grunt, or Gulp. The pretty awesome part of `npm run` some people are surprised about is that you get access to `node_modules` for free. Meaning that rather than having to do something weird like `node_modules/browserify/bin/browserify {args}`, `npm` figures all of that out for you.

## You Were Saying

Right. So. The `flexarea` thing. The code itself is irrelevant to this article, but I want to focus on two aspects of how I put it together.

## "nico-style" CSS

I've come up with a **CSS naming convention** similar to [BEM][17], but without all of the weird dashes (for the most part). Rather than splitting your CSS declarations in blocks, elements, and modifiers, I've come up with something arguably more pragmatic. You should start with a unique namespace for your component, preferably 2 to 4 characters long, such as `fa` in the case of _**f**lex**a**rea_. Then you can use as many words as you'd like to name classes associated with that component. You might decide to have `fa-container`, `fa-items`, `fa-item`, and `fa-item-selected`, or just `fa-selected`. In practice it won't matter all that much as long as the class names are reasonable and you don't use a single namespace for various components.

Indeed, sometimes you'll find yourself in need of slightly changing a component in a particular view or maybe when it's inside of another component. The preferred approach here is to **duplicate the class name but keep the parent namespace**. Suppose you have a `fa` component in the `at` (accounting) view, and you want to change the bottom margin for `fa-item` in the accounting view. Where possible, the second approach is preferred, but it's also decent to have one or two levels of selector nesting.

```css
.at-container .fa-item {
  margin-bottom: 5px;
}
```

```css
.at-item {
  margin-bottom: 5px;
}
```

If you are interested in this pattern, you might find it useful to check out the [Stylus files in ponyfoo@redo][18]. Note how I also keep components one per file, and modularize the style rules as aggressively as possible. Files are cheap, but a 2000 line stylesheet is every front-end developer's nightmare. Raise your hand if you've seen one of those written in LESS, SASS, or Stylus, containing **rules over six nesting levels deep**. I have. It's **not pretty**.

_Disclaimer: I didn't dub it **"nico-style"**, but I didn't bother to give the convention a better name either._

## Frameworkless Development

....


# Hint

.....


## Next Up!

Next up I should put together a markdown editor component for the front-end, and then I'll move on to the tedious process of migrating the views in [ponyfoo@master][4] to [ponyfoo@redo][3].


[1]: https://github.com/bevacqua/flexarea "Pretty flexible textareas"
[2]: https://github.com/bevacqua/hint "Awesome tooltips at your fingertips"
[3]: https://github.com/bevacqua/ponyfoo/tree/redo "ponyfoo@redo on GitHub"
[4]: https://github.com/bevacqua/ponyfoo/tree/master "ponyfoo@master on GitHub"
[5]: https://camo.githubusercontent.com/57839a85a2dbc4571da5243a30f51db24426ce2f/687474703a2f2f692e696d6775722e636f6d2f5a5770414875312e706e67
[6]: https://github.com/bevacqua/flexarea/blob/master/src/flexarea.js "flexarea.js on GitHub"
[7]: http://blog.ponyfoo.com/2014/01/27/my-first-gulp-adventure "My First Gulp Adventure"
[8]: https://github.com/bevacqua/flexarea/blob/master/gulpfile.js "Gulpfile for bevacqua/flexarea"
[9]: https://github.com/umdjs/umd "Universal Module Definition"
[10]: http://addyosmani.com/writing-modular-js/ "Writing Modular JavaScript"
[11]: https://twitter.com/addyosmani "@addyosmani on Twitter"
[12]: http://benalman.com/news/2010/11/immediately-invoked-function-expression/ "Immediately Invoked Function Expression"
[13]: https://github.com/substack/node-browserify#usage "Browserify CLI Usage"
[14]: http://blog.ponyfoo.com/2014/01/09/gulp-grunt-whatever "Gulp, Grunt, Whatever"
[15]: https://github.com/bevacqua/ponyfoo/blob/redo/build/release "Release build file on ponyfoo@redo"
[16]: https://www.npmjs.org/doc/cli/npm-link.html "npm link documentation"
[17]: http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/ "MindBEMding – getting your head ’round BEM syntax"
[18]: https://github.com/bevacqua/ponyfoo/tree/redo/client/css "CSS at ponyfoo@redo on GitHub"

[isomorphic ponyfoo taunus browserify npm-run gulp]
