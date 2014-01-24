# The Web Wars #

There have always been wars in browser-land. Browsers, specs, politics, _lots_ of politics. Even _libraries_ had theirs.

What once was the browser _utility library_ war, has now been settled, [jQuery](http://jquery.com/ "jQuery library") won that. We are now in the midst of another one, the _framework_ war, with [AngularJS](http://angularjs.org/ "Angular Model-View-Whatever Framework") leading the competition. There are lots to pick from though, and it isn't anywhere near settled. Popular frameworks include [BackboneJS](http://backbonejs.org/ "Backbone.js library"), [EmberJS](http://emberjs.com/ "Ember.js framework"), [SproutCore](http://sproutcore.com/ "SproutCore MVC"), [KnockoutJS](http://knockoutjs.com/ "Knockout.js Model-View-ViewModel Framework"), just to name a few.

Before **Node**, and frameworks like **Angular**, it wasn't all that common opening a web project and finding front-end code _organized in a way that scaled_. This war will probably drag on for at least a couple more years. Maybe even _indefinitely_.

As we can see on [TodoMVC](http://todomvc.com/ "TodoMVC MV* Comparison"), there are _a boatload_ of different **MV*** frameworks out there. I expect that many of those won't make it very far, while a few will gain more traction.

I'll also be covering [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1 "CommonJS Modules Spec") vs [RequireJS](http://requirejs.org/ "RequireJS script loader") vs [LazyJS](http://bevacqua.github.io/lazyjs/ "LazyJS: The minimalist JS loader"), an alternative I developed.

![angularjs.png][1]

I believe [AngularJS](http://angularjs.org/ "Angular Model-View-Whatever Framework") has a _good chance_ of winning. It's a great combination of **MV*** patterns, great modularization. It's _debatable_ whether IoC will captivate developers at large, but they certainly implemented it in a _clean and coherent_ way. Their solution is highly _extensible_, higly _comprehensive_, and, _considering its extensive feature-set_, extremely _intuitive and easy to use_. So, kudos **Google**!

However, it won't enjoy the _widespread adoption_ that **jQuery** has. **jQuery** is, ultimately, a utility library that's meant to be _auxiliary_ to your application, **AngularJS** is an integral solution, and it restricts how you can organize your front end architecture.

Lets get into [RequireJS](http://requirejs.org/ "RequireJS script loader"), a script loader that resembles the **AngularJS** [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection") mechanism.

## A little background history ##

There was a time when Node was coming up, when they had to decide on a spec to organize the complex code architecture that comes with a web application's back end. The [CommonJS Modules/1.1](http://wiki.commonjs.org/wiki/Modules/1.1 "CommonJS Modules Spec") spec was conceived. But it didn't _quite work_ in browser-land.

It wasn't that long ago [CommonJS and RequireJS](http://blog.millermedeiros.com/amd-is-better-for-the-web-than-commonjs-modules/ "AMD is better for the web than CommonJS modules, by Miller Medeiros") took sides in a little skirmish. Fortunately, that's [more or less](http://tomdale.net/2012/01/amd-is-not-the-answer/ "AMD is Not the Answer, by Tom Dale") settled. To be fair, I created [LazyJS](http://bevacqua.github.io/lazyjs/ "LazyJS: The minimalist JS loader") in hopes to _reignite_ that argument. I'll expand on that later.

## [CommonJS Modules/1.1](http://wiki.commonjs.org/wiki/Modules/1.1 "CommonJS Modules Spec") ##

Modules, as defined by the CommonJS spec, the so-called _Node way_, are directly related to the files that contain them, and can be included using `require`. A module will contain all properties published in the public interface defined in `module.exports`.

```js
// util.js

module.exports = {
	public_api: true,
	log_message: function(message){
		console.log(message);
	}
};

// main.js

var util = require('./util.js');
expect(util.public_api).toBeTruthy();
util.log_message('foo');

// > foo
```

This pattern is _very good for Node_.

- Everything that's not exposed with `module.exports` is considered private to the module
- Modules are interpreted once, and their results are stored for subsequent calls to `require`
- It's similar to what we see in other, _non-prototypal_, server side languages
- Modules just execute and return a value. You are not tied to a particular pattern

## [RequireJS](http://requirejs.org/ "RequireJS script loader") ##

![requirejs.jpg][2]

[AMD](http://requirejs.org/docs/whyamd.html "Why AMD? - RequireJS") is a _concession_ to the fact that browsers are just terminals and, unlike Node, assets are fetched **over the network**. This introduces a host of problems, such as latency, uncertainty (that the file will _ever_ load), asynchronicity, and such.

**AMD** attempts to correct these issues by providing an _asynchronous module loading pattern_.

	define('module', function(dep1, dep2){
		return function(){};
	});

Benefits of using **RequireJS** include the following.

- Modular code that isn't limited to one module per file, like CommonJS modules are
- **AMD** modules work even if they aren't resolved immediatly, due to dependencies such a script in an external [CDN](https://en.wikipedia.org/wiki/Content_delivery_network "Content Delivery Network")
- [DI](http://en.wikipedia.org/wiki/Dependency_injection "Dependency Injection") pattern. Dependencies are _inferred from the arguments_, resolved, and injected to the module function, all of which is done by **RequireJS**

## Behind enemy lines ##

If you are like me, you don't care about the _politics_ behind different specs and solutions. All that matters is getting a clean solution that _just works_.

> It's a fact that **CommonJS** was not aimed at the browser, and although I prefer it for Node, I wouldn't try and force it into browser-land. However, **RequireJS** attempts to do too much when it comes to the browser, and _kind of fails at it_.

You need a lot of boilerplate code in order to get **AMD** working, and _it shouldn't have to be that way_. 

**RequireJS** set out to be a _simple and easy to use_ script loading solution, but that's not quite what you find here.

> **RequireJS** allows modules on the `Object` level, but  I'm _not so hot_ on the idea that my script loader should _also_ take on the task of providing [inversion of control](http://en.wikipedia.org/wiki/Inversion_of_control "Inversion of Control technique") and dependency injection mechanisms

Besides that, there are some _unwanted complications_. The example above won't work when the JS is _minified_, because the arguments on the anonymous function will get renamed to something like `a, b`, meaning they won't be able to infer the names of the modules anymore. The solution, is **even more verbose**.

```js
define('module', ['dep1', 'dep2'], function(dep1, dep2){
	return function(){};
});
```

This leaves you wondering why they try so hard to provide **something that just won't work in production environments**. And why does everyone have to modify their code to comply with yours? That's _just not right_.

# How is **LazyJS** any different? #

- Modules don't need to adhere to silly conventions, you can you require them providing the `/path/to/the/source.js`
- _Faster_, very little JS is parsed on page load, after that, necessary JS is parsed on demand
- _Less tightly coupled_. Directives are comments that don't define the way you should style your code
- _Less ceremony_. Use the same tools and concepts in development and production
- _Succint_. Give it a hint of which modules you depend on, _that's it_
- [KISS](http://en.wikipedia.org/wiki/KISS_principle "Keep it simple stupid")

[LazyJS](http://bevacqua.github.io/lazyjs/ "LazyJS: The minimalist JS loader") is different in that **it can become whatever you want it to be**. It's different in that it lets you organize blocks of code (entire files or chunks), and specify their dependencies.

For now it doesn't even have an API, it's just _an idea_. And I want to spend some time figuring out the best "out of the box" feature-set, without staining everyone's code with extra code.

Suppose you want it to "become" **CJS**, then you should reference a bundle, disable AJAX calls, and let it resolve everything on its own before, synchronously, giving you a result back.

Similarly, you can leave it pretty much on its [current state](https://github.com/bevacqua/lazyjs/tree/9d3c3173ec067a83f5e4afafc29b9e195ef05798 "LazyJS on GitHub"), expose the `.lookup(url, done)` [function](https://github.com/bevacqua/lazyjs/blob/9d3c3173ec067a83f5e4afafc29b9e195ef05798/src/lazy-loader.js#L112 "LazyJS on GitHub"), and voilá, you've got RequireJS. Sort of.

Feedback regarding [LazyJS](http://bevacqua.github.io/lazyjs/ "LazyJS: The minimalist JS loader") is welcome, and I promise to try my best to leave it _as agnostic and unopinionated as possible_.

  [1]: http://i.imgur.com/hYmljo5.png "AngularJS application framework"
  [2]: http://i.imgur.com/tkY5UGR.jpg "RequireJS script loader"