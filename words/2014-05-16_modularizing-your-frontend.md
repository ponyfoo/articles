# Modularizing Your Front-End

In the past [I've wrote][1] about a small alternative to [`async`][5], named [`contra`][2], which is _barely over 2kb_, and has the browser at its heart. It comes with the usual suspects, allowing you to craft asynchronous flows in series or concurrently, as well as a few functional methods, an event emitter implementation, and a queue which powers most of the library.

[![contra.png][3]][2]

I've also discussed [campaign][8], where I modularized [email sending and templating][6]. That was really mostly a port of what was already in Pony Foo's code base, and thus [open-source][7], but written in a more portable way that could be used across other projects or efforts. In that occasion I explained the motives for developing the library as well, which mostly revolved around having a reusable email sender that was easier to bring into a Node application than the existing alternatives.

In this article I'll **describe the development process, choices I've made, and contents of my latest open-source** projects. In lieu with Pony Foo's humble, [_humble beginnings_][9], I've decided to continue on the path set forth back when I wrote [campaign][8], and [re-do Pony Foo from scratch][10]. The difference is that this time I've been putting more of a **focus on modularity**, basing off of what I've learned over the last year _(and from the glaring mistakes I made implementing the original blog engine)_.

- [`flexarea`][12] is a tiny library that helps humans resize `<textarea>` elements
- [`hint`][11] is a pure CSS implementation of the tooltips you can see today on **Pony Foo**
- [`taunus`][13] is a _micro MVC_ framework **I feel extremely excited about**

Caution: This article contains words about [Browserify][14], [`npm run`][15], [Gulp][16], framework-less JavaScript, and ponies. **Reader discretion is advised.** Also, not all of it might make sense.

  [1]: /2014/03/07/a-less-convoluted-event-emitter-implementation "A Less Convoluted Event Emitter Implementation"
  [2]: https://github.com/bevacqua/contra "Asynchronous flow control with a functional taste to it"
  [3]: https://raw.github.com/bevacqua/contra/master/resources/contra.png
  [5]: https://github.com/caolan/async "async utilities for node and the browser"
  [6]: /2014/01/07/email-sending-done-right "Email Sending Done Right"
  [7]: https://github.com/bevacqua/ponyfoo "Pony Foo on GitHub"
  [8]: https://github.com/bevacqua/campaign "Compose responsive email templates easily, fill them with models, and send them out"
  [9]: /2012/12/25/pony-foo-begins "Pony Foo Begins"
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

[![flexarea.png][3]][4]

The API is fairly simple. `flexarea` is a function, it takes a DOM node, surrounds it with a wrapper element, and appends the "grip" element to the wrapper. The grip is what's on the bottom of the `<textarea>` shown in the picture above. As you can [see in the repository][5], I used a Common.JS module for the component, meaning that I need Browserify in a compile step to try the component out, as well as during releases. For local development I deeply recommend you to familiarize yourself with [`npm link`][6] if you're a package author, as it'll **considerably speed up** your development productivity.

## Gulp, `npm run`, Whatever

There's a very similar [Gulpfile][7] to the one I detailed in [My First Gulp Adventure][8]. The "big" difference in the Gulpfile is that I run Browserify, given I'm using Common.JS in my source code. Browserify takes a little dancing in Gulp. Here's how it ends up looking like, after removing some of the cruft, getting us just the core idea.

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

The code above would compile my source code into [a UMD module][9] (also read [Writing Modular JavaScript][10] by [Addy Osmani][11] while you're at it). That is pretty awesome. You write CJS code, with all the benefits that that implies, such as no implicit globals and no [IIFE] hell, and you get a module that can be run and included pretty much everywhere you can run JavaScript! Except there's no way in hell compiling a single file into two distributions should involve around 15 lines of code. Aren't you using a library to simplify things? Compare that to using `npm run` and the [`browserify` CLI][12].

```js
browserify src/flexarea.js -s flexarea -o dist/flexarea.js -d
browserify src/flexarea.js -s flexarea -o dist/flexarea.min.js -t uglifyify
```

_Anyways._ Lately I've been leaning more and more into [the "whatever" camp][13]. I find that it's much leaner to just use `npm run`, and [putting together a build file][14] is incredibly faster if you aren't dealing with a formal build system such as Grunt, or Gulp. The pretty awesome part of `npm run` some people are surprised about is that you get access to `node_modules` for free. Meaning that rather than having to do something weird like `node_modules/browserify/bin/browserify {args}`, `npm` figures all of that out for you.

## You Were Saying

Right. So. The `flexarea` thing. The code itself is irrelevant to this article, but I want to focus on two aspects of how I put it together.

## "nico-style" CSS

I've come up with a **CSS naming convention** similar to [BEM][15], but without all of the weird dashes (for the most part). Rather than splitting your CSS declarations in blocks, elements, and modifiers, I've come up with something arguably more pragmatic. You should start with a unique namespace for your component, preferably 2 to 4 characters long, such as `fa` in the case of _**f**lex**a**rea_. Then you can use as many words as you'd like to name classes associated with that component. You might decide to have `fa-container`, `fa-items`, `fa-item`, and `fa-item-selected`, or just `fa-selected`. In practice it won't matter all that much as long as the class names are reasonable and you don't use a single namespace for various components.

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

If you are interested in this pattern, you might find it useful to check out the [Stylus files in ponyfoo@redo][16]. Note how I also keep components one per file, and modularize the style rules as aggressively as possible. Files are cheap, but a 2000 line stylesheet is every front-end developer's nightmare. Raise your hand if you've seen one of those written in LESS, SASS, or Stylus, containing **rules over six nesting levels deep**. I have. It's **not pretty**.

_Disclaimer: I didn't dub it _**"nico-style"**_, but I didn't bother to give the convention a better name either._

## Framework-less Development

The other aspect of `flexarea` that I think is worth mentioning is that it's framework-less. It has every reason to be. Why would you make a commitment to [a large library like jQuery][17] just to provide a little bit of functionality such as a resizer? Would you add **jQuery UI** to that? Where does it end? How much did you really gain by adding all those extra constraints? It must be a lot, because on the flip side, you are getting bigger, and more importantly, you are limiting your project to **"nerds who use jQuery in their application"**, where you could just be writing code for **"any nerd with a browser"**. If your code didn't really depend on the DOM API, like in the case of [`contra`][18], then you could be targeting just about anyone with a JavaScript interpreter.

Okay sure, so you need promises, and jQuery provides an implementation, oh -- how convenient! Well, you certainly didn't have to include all of jQuery for that. You could _at least try_ and [use a custom build][19]. Or you could also find yourself a Promise library which only does promises (and is closer to the ES6 Promise spec), such as [rsvp][20]. That sounds great! At least, you won't be turning down users who don't have jQuery in their projects.

I believe jQuery is becoming less relevant, and that **a polyfill-oriented approach should be preferred over using jQuery**. This is not just insano-purist talk, jQuery damages mobile web performance. jQuery UI annihilates it. Before you go ahead and post a comment, I'm not just talking about the size of these libraries. **jQuery takes a long time to be interpreted in mobile devices.** This happens on every request, regardless of how aggressively jQuery gets cached. That'll bite your page load times, making them slower every time. 

> A **polyfill-oriented** approach should be preferred over using jQuery

[Polyfills][25]. Indeed, my mention of [rsvp][21] was just auxiliary to introducing [es6-promises][22], a Promise polyfill _based_ on **rsvp**. The awesomeness in _ponyfills_ is that they **aren't opinionated** about the feature's API. If you want to use a feature, and there's a native implementation coming to most browsers, it makes sense using a polyfill because it's a drop-in fix for browsers where it's _not yet implemented_. You are also theoretically able to remove the polyfill cleanly once you deem it no longer necessary, based on the analysis of user agents coming your way. You just remove the reference and you are done. There's no large commitment, which would result in anxious keyboard pounding and long afternoons tediously refactoring your way out of an opinionated library's API.

![time.png][23]

**TL;DR** If there's a polyfill that already does what you need, then **always prefer that over a library** that forces you to adopt their own API. [Here's a few you can use][26] to bring browsers up to speed by enabling features like `dataset`, `classList`, and others.

# Hint

Hint made an even happier Pony. Remember the little tooltips that you get when hovering something in **Pony Foo**? Come on. [They're everywhere!][2]. _Here's another._

[![hint.png][27]][2]

The hint in the figure shown above, though, isn't in production but rather using the `hint` micro-library. I feel particularly good about `hint` because I've removed all JS from it, making it into a pure-CSS solution. All you have to do is add the stylesheet to your page, and then you can create hints with a very reasonable piece of markup. It's quite semantic, and you don't have to waste time telling a JavaScript library **when** it should be turning your attributes into the thing you want.

```html
<a data-hint='The draft will be deleted'>Discard Draft</a>
```

The cute thing about `hint` is that I wrote it in [Stylus][28], but I still provide a CSS distribution that gets _rebuilt during releases_. Certainly, it involves [a bit of Gulp][30] work, but it's not as bad as with the [Browserify][29] example we saw earlier.

```js
gulp.task('build', ['clean'], function () {
  return gulp.src('./src/hint.styl')
    .pipe(stylus())
    .pipe(gulp.dest('./dist'))
    .pipe(rename('hint.min.css'))
    .pipe(minifyCSS())
    .pipe(size())
    .pipe(gulp.dest('./dist'));
});
```

That being said, `npm run stylus` would definitely be **leaner and clearer**. I guess that _copying and pasting_ did win this battle on behalf of Gulp, though.

```shell
stylus src/hint.styl -o dist/hint.css
stylus src/hint.styl -o dist/hint.min.css --compress
wc -c dist/hint.min.css
```

That's as far as builds go, but **what about the component?** I've managed to turn the solution into a pure CSS one by getting rid of assumption that `hint` cared about whether or not the attribute was empty. It'd check that the hint wasn't empty and provide an API to either enable or disable a `hint` tooltip. Oh, also, [that code was written in jQuery][31]. I decided that that validation was better left to the consumer, who can decide exactly when and how to apply `data-hint` attributes on their DOM elements, and got rid of the JavaScript code.

I mentioned Stylus, so I think it's also worth mentioning how easy it is to include `hint` in your project provided that you are already using Stylus yourself. [This is how I do it][32] on **ponyfoo@redo**. There's **no extra build step** involved, because Stylus takes care of dependency resolution, much like Browserify does.

```css
@import 'node_modules/hint/src/hint'
```

##### That's it!

I'll leave [Taunus][24] for next week! I think it deserves its own article. _(and a little more polish!)_

  [1]: https://github.com/bevacqua/flexarea "Pretty flexible textareas"
  [2]: https://github.com/bevacqua/hint "Awesome tooltips at your fingertips"
  [3]: https://camo.githubusercontent.com/57839a85a2dbc4571da5243a30f51db24426ce2f/687474703a2f2f692e696d6775722e636f6d2f5a5770414875312e706e67
  [4]: https://github.com/bevacqua/flexarea "Pretty flexible textareas"
  [5]: https://github.com/bevacqua/flexarea/blob/master/src/flexarea.js "flexarea.js on GitHub"
  [6]: https://www.npmjs.org/doc/cli/npm-link.html "npm link documentation"
  [7]: https://github.com/bevacqua/flexarea/blob/master/gulpfile.js "Gulpfile for bevacqua/flexarea"
  [8]: /2014/01/27/my-first-gulp-adventure "My First Gulp Adventure"
  [9]: https://github.com/umdjs/umd "Universal Module Definition"
  [10]: http://addyosmani.com/writing-modular-js/ "Writing Modular JavaScript"
  [11]: https://twitter.com/addyosmani "@addyosmani on Twitter"
  [12]: https://github.com/substack/node-browserify#usage "Browserify CLI Usage"
  [13]: /2014/01/09/gulp-grunt-whatever "Gulp, Grunt, Whatever"
  [14]: https://github.com/bevacqua/ponyfoo/blob/redo/build/release "Release build file on ponyfoo@redo"
  [15]: http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/ "MindBEMding – getting your head ’round BEM syntax"
  [16]: https://github.com/bevacqua/ponyfoo/tree/redo/client/css "CSS at ponyfoo@redo on GitHub"
  [17]: /2013/07/09/getting-over-jquery "Getting Over jQuery"
  [18]: https://github.com/bevacqua/contra "Asynchronous flow control with a functional taste to it"
  [19]: https://github.com/jquery/jquery#how-to-build-your-own-jquery "How to Build Your Own jQuery"
  [20]: https://github.com/tildeio/rsvp.js "A lightweight library that provides tools for organizing asynchronous code"
  [21]: https://github.com/tildeio/rsvp.js "A lightweight library that provides tools for organizing asynchronous code"
  [22]: https://github.com/jakearchibald/es6-promise "A polyfill for ES6-style Promises"
  [23]: http://i.imgur.com/LkMiobQ.jpg "I'd have to change all these files, ain't nobody got time fo dat!"
  [24]: https://github.com/bevacqua/taunus "Taunus on GitHub"
  [25]: http://remysharp.com/2010/10/08/what-is-a-polyfill/ "What is a Polyfill?"
  [26]: https://github.com/remy/polyfills "Collection of Polyfills by Remy Sharp"
  [27]: https://camo.githubusercontent.com/e7fef05529a194b8238efd6a7df9f0a16c65daef/687474703a2f2f692e696d6775722e636f6d2f454650356a34452e706e67
  [28]: http://learnboost.github.io/stylus/ "Expressive, dynamic, robust CSS"
  [29]: https://github.com/substack/node-browserify "browser-side require() the node.js way"
  [30]: https://github.com/bevacqua/hint/blob/master/gulpfile.js "Gulpfile for hint on GitHub"
  [31]: https://github.com/bevacqua/ponyfoo/blob/3a1bfc51a48d06db5868ab638df5c922e02e4afc/src/frontend/js/ext/jquery.ui.js#L4-L29 "$.hints in ponyfoo@3a1bfc51"
  [32]: https://github.com/bevacqua/ponyfoo/blob/redo/client/css/components/data-hint.styl "data-hint on ponyfoo@redo"

[ponyfoo front-end modules browserify npm-run gulp]
