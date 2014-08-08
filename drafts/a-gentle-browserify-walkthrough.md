# A Gentle Browserify Walkthrough

Building modules for the front-end has become an increasingly easy problem to solve. Back in the nineties we had our Java applets, our `<MARQUEE>` and `<BLINK>` tag combinations, and those beloved `<CENTER>` tags. Oh and we were mostly developing on Front Page. Anyways, time to **wean off the nostalgia**. Let's focus.

This time around I want to focus on [Browserify][2], a lean build step you can take to obtain [CommonJS][3] modules in your browser **today**. You have no idea what CommonJS modules are or why you need them? _Keep on reading!_

[![browserify.png][1]][2]

CommonJS is a module format for JavaScript, widely adopted by the [Node.js][5] community and popularized by [npm][4], the package manager that's bundled with Node. In this article you'll learn about CommonJS and how you can bring those same modules to your client-side code, using [Browserify][2] and maybe a build tool to automate your processes.

Browserify is one of those tools that the open-source community almost takes for granted, whereas it's overlooked by the vast majority of companies, hardly making its way into projects in a professional setting. Not all is lost, sometimes big companies _do the right thing_ and use it to develop [their open-source projects][7].

If you're still stuck with [AMD modules][6] and their sheet-piles of documentation at work, then maybe you should consider Browserify for a change.

  [1]: http://i.imgur.com/tYvbZwb.png
  [2]: http://browserify.org/ "Browserify lets you require('modules') in the browser by bundling up all of your dependencies"
  [3]: http://wiki.commonjs.org/wiki/CommonJS "CommonJS Module Spec"
  [4]: https://www.npmjs.org/ "Node Packaged Modules"
  [5]: http://nodejs.org/ "Node.js Platform"
  [6]: https://github.com/facebook/react/ "Facebook React on GitHub"
  [7]: http://requirejs.org/ "RequireJS Module Loader"

I could name quite a few reasons why **you should be using Browserify today**. First, let's talk about [Common.JS][1].

[![commonjs.png][2]][1]

CommonJS is the open module format specification behind the Node.js platform. That sounds overly complicated, but its not. CommonJS revolves around a few very straightforward rules.

> Every file you use is a "module". Modules are a 1-to-1 mapping to files

The above just means that in CommonJS you have a single entry point, which is a CommonJS module file, and everything is loaded from there. Implicitly, this signifies that module loading happens synchronously, but you could still use multiple entry points, placing each on its own `<script>` tag, and even have them interact with each other. This means that while CommonJS is easily bundled together you can also _"partition"_ your bits into several modules and serve them progressively rather than offload everything to the browser at once.

As a quick example, here's a module that prints numbers from `1` to `30` in your `console`. Let's say the file is named `app.js`.

```js
var i = 0;

for (; i++ < 30; ) {
  console.log(i);
}
```

If you wanted to try this module out, you could download and install [Node.js][3], fire up a terminal, and type in the following command.

```js
node app
```

One of the most important benefits of using CommonJS lies in how scoping behaves in its modules. Let's compare it with typical behavior in browsers.

# Differences in Scoping

In the browser, you might be used to an implicit global `window` scope. This typically forces us to wrap our code with `functions` on every JavaScript file, creating closures so that the implicit scope is no longer the global object.

For example, in the browser, you might be used to the following behavior.

```html
<script src='foo.js'></script>
<script src='bar.js'></script>
```

```js
// foo.js

var i = 0;

for (; i++ < 30; ) {
  console.log(i);
}
```

```js
// bar.js

console.log(i);
// <- 31
```

In order to work around that, you'd go ahead and create a closure in the `foo` module, preventing `i` to leak into the global object and spill information to the other camp_, _...err,_ file.

```js
// foo.js

void function () {
  var i = 0;

  for (; i++ < 30; ) {
    console.log(i);
  }
}();
```

```js
// bar.js

console.log(i);
// <- ReferenceError: i is not defined
```

CommonJS has **implicit local scopes**, meaning that variables declared at the top of a module's scope won't leak into the global object, but rather remain contained within the module where they're declared. Of course, you could always publish properties on the global object, by being explicit about it.

```js
global.warming = true;
```

While _possible_, it's **not recommended**. CommonJS encourages us to establish inter-module communications through API declarations.

> Modules can export an API by setting `module.exports`

Modules can publish a public API to `module.exports`. If they do so, other modules will be able to access this API. Consider it the public interface of the module.

```js
var i = 1;
var max = 30;

module.exports = function () {
  for (i -= 1; i++ < max; ) {
    console.log(i);
  }
  max *= 1.1;
};
```

In the example shown above, both `i` and `max` are only accessible internally by our module, while consumers will be able to access the `function` that we're exporting, and invoke it as often as they please. You aren't limited to exposing `function`s but you can actually expose any API you want, which is typically an `object` or an [object factory][4] method.

> ###### Trivia
>
> Note that the CommonJS `exports` specification didn't originally allow exporting `function`s, but Node has allowed it since its early days with the [`module.exports`][5] alternative, for convenience.

The counterpart to exposing an API via `module.exports` is consuming another module's API. Let's look into that one.

> Modules can consume the API exposed by other modules using the `require(module)` method, where `module` is the relative path from the file where the require statement is executed, to the location of the module we want to import is located

Again, it's simpler than it sounds. Suppose you have a `print.js` file and a `printer.js` file in the same directory. You'd have `printer.js` expose the API we've been fiddling with.

```js
var i = 1;
var max = 30;

module.exports = {
  print: function () {
    for (i -= 1; i++ < max; ) {
      console.log(i);
    }
    max *= 1.1;
  }
};
```

Then there's `print.js` which does the actual printing. Loading other modules happens synchronously and directly in your JavaScript code. You can omit the `.js` extension for brevity. You can also assign the results of requiring a module to a variable, or you can choose not to do that.

```js
var printer = require('./printer');

printer.print();
printer.print();
printer.print();
printer.print();
```

Again, you could run this example with Node by entering the command below into a terminal.

```shell
node print
```

A `require`d module will only be interpreted once. This means that calls to `require('./printer')` won't reset `i` after the module has been interpreted once. In fact, the interface exported by [the module is cached][6], and that is returned every time. You could fiddle with the caching behavior by removing entries from the cache, but it's considered a bad practice unless you have a legitimate reason to go there.

### But what about the browser?

So far, the CommonJS modules you've been looking at in this article work perfectly fine under a Node.js execution environment, but this article started out talking about the front-end. I wanted to first introduce you to the type of modules you'd be earning by making the switch to [Browserify][21], so first I had to explain how CommonJS works.

Now that that's out of the way, let's throw some questions at the discussion.

- Why is CommonJS "better" than _["whatever"][7]_, or than using AMD?
- Doesn't this cripple my development productivity?
- How do I bring these modules to the browser?
- What other benefits can I get from using CommonJS modules?
- What are some advanced use cases?

I'll go through these in turn, hoping to answer your concerns when it comes to using CommonJS in the browser through Browserify.

# Why CommonJS?

As I covered earlier, CommonJS gives you **true modules** contained in by _individual files_, but first let's look at the **reasons why you might want to ditch the alternatives**.

#### Why not _"whatever"_?

The idea of using nothing, while appealing, poses no benefit other than feeling nostalgic about those Java Applets you used to adore.

- You are required to create a closure in every piece of code that you write, otherwise you risk leaks to the global object
- You need to figure out your own system to develop modular components, and be consistent about it too
- You need to ["compute the dependency graph"][9] by yourself, which is a nice way of saying you have to stack your `<script>` tag deck in order to make sure that parts of your application that depend on other pieces of code come after the pieces they depend upon

![dependency-graph.png][8]

Note that, in this section, I'm not speaking about ES6 modules because they need a lot of work until we can use it natively in [A-grade browsers][11] and we can consider them _"using nothing"_. Everyone pretty much agrees on this one, though. Let's move to RequireJS.

#### Why not [RequireJS][10]?

RequireJS is a module loader based on the AMD specification. While I personally dislike it, this way of approaching modularity is a popular alternative to CommonJS. It does provide a solution to modularity, dependency injection, and a built-in solution to the dependency graph issue, which is always a great thing.

That being said, there are _some issues_ with RequireJS.

- Too **verbose**. It basically needs us to wrap our code in closures using `define` and `require` statements
- Overly **complex**. Have you seen the [documentation on the RequireJS site][12]? Jesus! Every little thing means you need to adjust and tweak your configuration to be just so for that specific corner-case. That's not very [high-quality][13] of them
- _Compatibility issues_ with libraries that don't conform to their standards
- Wildly different development and production distributions

During development RequireJS provides [a super-asynchronous API][14] that enables modules to be loaded through AJAX as dependencies are resolved. For release, the recommended approach is to compile the modules into one or many bundles. As usual, RequireJS provides [a boatload of documentation on the subject][15]. _Way more_ than you'd ever want to know about a module system.

> To me, these are all just indicators of **poor design**.

Fine, I **don't like** RequireJS. For the record, I would **still prefer it over not doing anything**. Currently, the reasons why I prefer CommonJS over ES6 has nothing to do with ES6 and everything to do with the features that CommonJS boasts.

- Straightforward, familiar syntax
- Code sharing with Node.js
- Access to [all of `npm`][16]

The syntax familiarity point is a minor one, but I'll expand on these items later in the article.

# Doesn't this cripple my productivity?

You probably think that a build step which needs you to **bundle your code together even during development** is a nightmare from a day-to-day development productivity point of view. Thanks to the [myriad of tools][7] that are available to us, this is _not that big_ of a deal when it comes to **Continuous Development** (the ability to rebuild the compiled bundle whenever your code changes). When it comes to productivity we take a **speed hit** because we have to wait for the build step to complete after we make changes to our code.

Then there's also the fact that **bundled-up pieces of code make debugging hard**, because stack traces and logging statements end up referencing line numbers that don't match your source code. Luckily Browserify has great support for [source maps][17], which can point the browser to the sources of compiled assets. It's just a matter of setting the `--debug` option on the command-line, or `debug` when using the programmatic API.

# How do I bring these modules to the browser?

Using [Browserify][21]! It's quite easy. Here's _what you need_ to know.

First off, you can run Browserify using its [CLI][26], its [API][25], the [Grunt plugin][22], [piping into `gulp.dest`][23], the [Broccoli plugin][24], etc. Instead of documenting each and every one of these ways to run Browserify I'll keep my examples to the CLI, which translates quite nicely to what the API and the plugins expect you to do.

In the `printer` example shown earlier, for example, compiling the code so that it works in the browser would just be a matter of using the command shown below.

```shell
browserify print.js > dist/app.js
```

You could also use the `-o` flag, indicating the destination file.

```shell
browserify print.js -o dist/app.js
```

This is most useful if you need to run the task whenever your code changes. To that end, you'll use the specialized [`watchify` tool][27]. Then, when your code changes, `watchify` will make sure to recompile the bundle. Watchify has the same API as the regular `browserify` CLI does, which is super convenient for us.

```shell
npm install -g watchify
```

```shell
watchify print.js -o dist/app.js
```

If you wanted to generate **a source map**, you'd only have to add the `-d` option.

```shell
browserify print.js -do dist/app.js
```

```shell
watchify print.js -do dist/app.js
```

There's also an extensive list of "transforms" that allow you to do neat things during compilation. For instance, if you [love CoffeeScript like I do][28], you could use the [`coffeeify`][30] transform as shown below.

```shell
browserify print.coffee -t coffeeify -o dist/app.js
```

There's _tons_ of other transforms you could use. Make sure to [consult the wiki page][29], you'll probably find what you need in there. Another transform that you may find super helpful is [`brfs`][31]. That transform works by inlining calls to `fs.readFileSync` into your client-side bundles. This means that we could have a `cool-sites.txt` file and a piece of code like the one shown below, and compile it into something that works on the browser!

```js
var fs = require('fs');
var sites = fs.readFileSync(__dirname + '/cool-sites.txt', { encoding: 'utf8' });

sites.split('\n').forEach(function (site, i) {
  console.log('%s: %s is pretty cool', i, site);
});
```

```
ponyfoo.com
github.com
codepen.com
newsblur.com
twitter.com
```

Oh, `__dirname` is a special variable that's local to every CommonJS module, and it contains the path to the directory for that module. Similarly, `__filename` is the module's file name. The compiler will be smart enough to figure out the result of `__dirname + '/cool-sites.txt'` statically. The compiler **isn't able to handle most cases** where you try to dynamically calculate the file path, so using `path.join` and the like won't be possible. This is unfortunate because its a common scenario in Node.js, however, it rarely comes up in modules that are meant for the browser. You could compile the code above using the command below.

```shell
browserify sites.js -t brfs -o dist/app.js
```

Fortunately, there's a way you could work around the static file path problem, but it takes a little bit of work.

### Working around static file paths

When the **static file-naming** situation gets the best of you, you could work around it by doing something similar to what I did in [Taunus][33], where I created a CLI that creates a [static file containing route definitions and then requires that into the bundle][32], _effectively annuling_ the limitation. I was able to do that because I had to `require` a ton of files defined in a server-side route definition file, but I didn't want to have the consumer repeat that information in a very similar format for the client-side. Due to the limitation in Browserify, I couldn't just create a for loop either, so I worked around it with a generator. This generator ends up unrolling the `for` loop, as shown in the example below, taken from Pony Foo's [`redo` branch][34].

```js
var controllers = {
  'author/compose': require('../../client/js/controllers/author/compose.js'),
  'home/index': require('../../client/js/controllers/home/index.js')
};
 
var routes = {
  '/': 'home/index',
  '/account/login': 'account/login',
  '/author/compose': 'author/compose'
};
 
module.exports = {
  templates: templates,
  controllers: controllers,
  routes: routes
};
```

Problem solved! That's a file Browserify **does understand**.

# What other benefits can I get from using CommonJS modules?

Besides being _"not as bad as the alternatives"_, CommonJS has a few things going for it on its own. Earlier on, I mentioned a few benefits of using CommonJS. Time to go through them.

> - Straightforward, familiar syntax
> - Code sharing with Node.js

If you're a Node developer, [Browserify][21] will feel like [the Architect][35]'s gift to you. Being able to leverage the CommonJS syntax in the client-side too opens your applications up to a world of opportunities. A big one of those is the ability to share an identical code base between the server and the client side. Quite a few libraries out there have some degree of cross-platform_ity_ built into them, perhaps using [the UMD pattern][36]. The twist in using Browserify is that you get to use CommonJS just like in Node, and you can avoid the UMD boilerplate, as well as any other boilerplate.

> Access to [all of `npm`][16]

Perhaps the most significant of the benefits in using CommonJS modules is that there are **so, so many** modules written in CommonJS and published to `npm`. In my experience it has almost always been easy to find a module that did what I needed to do, and most of them work as expected. While there's also [Bower][38], [Component][39] and others out there, the module-mentality in Node.js camps is much stronger than in other areas.

> Note that **Component seems like it's dying**. The site has been down [for a month now][40], and [duo][41] seems to be going to be its successor.

With a little work, you could also use modules that are written under other formats such as AMD or by publishing a global object. In order to make such modules work for us, we could use [`browserify-shim`][37], which makes it quite easy.

There's [a StackOverflow answer][42] that details the process. I've copied part of the answer below.

> 1. Install `browserify-shim`:
>
> ```shell
> npm install browserify-shim --save-dev
> ```
>
> 2. In `package.json` file, tell `browserify` to use `browserify-shim` as a transform:
>
> ```json
> {
>   "browserify": {
>     "transform": [ "browserify-shim" ]
>   }
> }
> ```
>
> 3. In `package.json` file, tell `browserify-shim` to map jQuery to the jQuery in the global scope:
>
> ```json
> {
>   "browserify-shim": {
>     "jQuery": "global:jQuery"
>    }
> }
> ```js
>
> 4. Run `browserify`:
>
> ```shell
> browserify mymodule.js > bundle.js
> ```

Let's move on to our last topic of discussion!

# What are some advanced use cases?

...

> **Browserify your front-end away!**

Here's a few related articles for you to chew on.

- [Building High-Quality Front-End Modules][13]
- [Gulp, Grunt, Whatever][7]
- [Browserify Handbook][18] (written by Substack)
- [Introduction to Browserify][19]
- [Cross-platform JavaScript with Browserify][20]


  [1]: http://wiki.commonjs.org/wiki/CommonJS "CommonJS specification"
  [2]: https://cloud.githubusercontent.com/assets/934293/3831513/6fd5d336-1d94-11e4-868b-4f1165e6600e.jpg
  [3]: http://nodejs.org/ "Node.js application development platform"
  [4]: http://addyosmani.com/resources/essentialjsdesignpatterns/book/#factorypatternjavascript "The Factory Pattern explained by Addy Osmani"
  [5]: http://nodejs.org/api/modules.html#modules_module_exports "Documentation for module.exports"
  [6]: http://nodejs.org/api/modules.html#modules_caching "Module caching documentation by Node.js"
  [7]: /2014/01/09/gulp-grunt-whatever "Gulp, Grunt, Whatever"
  [8]: https://cloud.githubusercontent.com/assets/934293/3849717/2f96edc6-1e7c-11e4-9de5-0a53f93fcd82.png
  [9]: http://en.wikipedia.org/wiki/Dependency_graph "Dependency Graph as defined by Wikipedia"
  [10]: http://requirejs.org/ "RequireJS Module Loader"
  [11]: https://github.com/yui/yui3/wiki/Graded-Browser-Support#a-grade "A-grade browsers according to Yahoo"
  [12]: http://requirejs.org/docs/api.html "RequireJS API Documentation"
  [13]: /2014/08/05/building-high-quality-front-end-modules "Building High-Quality Front-End Modules"
  [14]: /2013/05/13/the-web-wars "The Web Wars"
  [15]: http://requirejs.org/docs/optimization.html "RequireJS Optimization Documentation"
  [16]: http://npmjs.org/ "Node Packaged Modules"
  [17]: http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/ "Introduction to JavaScript Source Maps"
  [18]: https://github.com/substack/browserify-handbook "Browserify Handbook by Substack"
  [19]: http://blakeembrey.com/articles/introduction-to-browserify/ "require(‘modules’) in the browser."
  [20]: https://blog.codecentric.de/en/2014/02/cross-platform-javascript/ "Sharing Code Between Node.js and the Browser"
  [21]: http://browserify.org/ "Browserify lets you require('modules') in the browser by bundling up all of your dependencies"
  [22]: https://github.com/jmreidy/grunt-browserify "grunt-browserify on GitHub"
  [23]: https://github.com/gulpjs/plugins/issues/47 "gulp-browserify is blacklisted"
  [24]: https://github.com/gingerhendrix/broccoli-browserify "broccoli-browserify on GitHub"
  [25]: https://github.com/substack/node-browserify#api-example "Browserify API Example Documentation on GitHub"
  [26]: https://github.com/substack/node-browserify#usage "Browserify CLI Usage Documentation on GitHub"
  [27]: https://github.com/substack/watchify "substack/watchify on GitHub"
  [28]: /2013/09/28/we-dont-want-your-coffee "We don't want your Coffee"
  [29]: https://github.com/substack/node-browserify/wiki/list-of-transforms "List of Browserify transforms on GitHub"
  [30]: https://github.com/jnordberg/coffeeify "coffeeify on GitHub"
  [31]: https://github.com/substack/brfs "substack/brfs on GitHub"
  [32]: https://github.com/bevacqua/taunus/blob/b4524b1d7d74a620e40de38fad54d2f33d998f76/bin/taunus#L15-L34 "Relevant lines in the Taunus CLI"
  [33]: https://github.com/bevacqua/taunus "bevacqua/taunus on GitHub"
  [34]: https://github.com/ponyfoo/ponyfoo/tree/redo "bevacqua/ponyfoo/redo on GitHub"
  [35]: http://en.wikipedia.org/wiki/Architect_(The_Matrix) "You know the one."
  [36]: https://github.com/umdjs/umd "Universal Module Definition"
  [37]: https://github.com/thlorenz/browserify-shim "browserify-shim on GitHub"
  [38]: http://bower.io/ "bower.io"
  [39]: https://github.com/component/component "component/component on GitHub"
  [40]: https://github.com/component/component/issues/587 "component.io site down"
  [41]: https://www.npmjs.org/package/duo "duo on npmjs.org"
  [42]: http://stackoverflow.com/a/23129051/389745 "How do I use Browserify with external dependencies?"

[browserify modules front-end tutorial]
