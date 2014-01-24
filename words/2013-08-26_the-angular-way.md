# The Angular Way

For the past few months I've been sailing around the world of Angular. Today I can hardly imagine myself doing day to day coding on a large front-end web application without _some kind_ of data binding framework, such as [Angular.js](http://angularjs.org/ "Angular.js MVW Framework"), [Backbone.js](http://backbonejs.org/ "Backbone.js"), and [friends](http://underscorejs.org/ "Underscore.js utility belt"); and I can't believe I've done so in the past.

I might however be _a bit biased_, considering the application I'm working on is a **PhotoShop**esque editor in the browser, which presents the same data in _radically different_ ways.

- Layers are presented graphically, taking up large portions of the screen. They are also listed in a panel where you can delete them.
- When you select a layer it gets the typical dashed line around its edges, and it also gets highlighted in the list view.
- Similarly, properties like the dimensions of a layer show up both in a panel and define their size upon the canvas.
- The panels I've mentioned can be dragged around, collapsed, and closed.

This kind of interaction, data binding, and view synchronization would be easily be a **maintenance nightmare** if it wasn't for a framework such as Angular. Being able to update a model in one place, and have Angular update all relevant views almost feels like cheating. Adding, removing, or moving a layer is just a matter of changing an object. `layer.x += 10`, done. There's no need to invalidate the view by hand, or to manually update each instance of the layer in the DOM. Or to even interact with the DOM, for that matter. 

Angular enabled us to go places we wouldn't ever have dreamt of, such as setting up a bunch of keyboard shortcuts that are enabled based on the current context of the application. For example, text editing shortcuts, such as <kbd>⌘</kbd> <kbd>B</kbd> to toggle **bold** text, are just enabled when we're editing a text layer.

[![app-press-shortcuts.png][2]](http://i.imgur.com/I1ZDYeO.png "Click to enlarge")

Similarly, we tacked a description onto these shortcuts (which are registered through a _service_ we created), and we are then able to show a list of shortcuts, along with their descriptions, in a handy cheat sheet. Furthermore, we wrote a _directive_ which enables us to bind individual DOM elements with their keyboard shortcut counterparts, showing a tooltip when you hover over them for a little while, letting you know a keyboard shortcut is available, too.

> Angular enabled us to go places we wouldn't ever have dreamt of.

Honestly, it's as if we weren't writing a web application anymore. The web is just the medium. As we improve our understanding of Angular, the code gets more modular, more self-contained, and yet... more inter-connected. It is simply becoming **more Angular**.

And by Angular I mean the highly interactive rich application development philosophy behind Angular.js, the same one that's enabled us to develop a piece of software that I wouldn't have thought possible a while back.

[![app-press-canvas.png][1]](http://i.imgur.com/RSqiAkS.png "Click to enlarge")

We were even able to develop a full-fledged history panel that updates the DOM to the currently selected point in history, and _it performs well, too_! Seeing the data binding capabilities of Angular update every small detail in your view work flawlessly as you go back and forth in the history panel is inspiring, to say the least.

> It wasn't always so easy, the code-base used to be an uncontrollable mess.

Indeed, in the last few weeks we've been updating and re-writing the overall architecture of our front-end. Before we took up this re-write, looking to update Angular to [edge](https://github.com/angular/angular.js/tree/v1.2.0rc1 "Angular.js v1.2.0 RC 1"), all the way from [0.10.6](https://github.com/angular/angular.js/tree/v0.10.6 "Angular.js v0.10.6"). That's a pretty long way to go, if you look at the [change log](https://github.com/angular/angular.js/blob/v1.2.0rc1/CHANGELOG.md#0106-bubblewrap-cape-2012-01-17 "Angular.js change log, starting at v0.10.6").

Going through this refactoring, we went from doing Angular _the wrong way_, to doing Angular _the Angular way_.

The wrong way, in our case, encompassed quite a few issues we had to work through before getting to the lovable state our code-base is in at the moment.

### Controllers declared on the global scope

This one is, sadly, pretty common amongst folks who've been using Angular since the early days. If you're familiar with Angular, you might be familiar with this pattern, too.

```js
// winds up on window.LoginCtrl ...
var LoginCtrl = function ($scope, dep1, dep2) {
    // scope defaults
};

LoginCtrl.prototype.resetPassword = function () {
    // reset password button click handler  
};

// more on this one later
LoginCtrl.$inject = ['$scope', dep1', 'dep2'];
```

That kind of file isn't wrapped in a closure, either, meaning anything declared on the root scope goes to the global `window` object, yuck. The Angular way to do that is to use the [module API](http://docs.angularjs.org/guide/module "Angular.js Modules") they provide. But, as you can see, even in the documentation the _recommended setup_ is still outdated and suggests you **use the global scope mercilessly**:

> Do this, and **wonderful things** will happen to you!
>
>     // A Controller for your app
>     var XmplController = function($scope, greeter, user) {
>       $scope.greeting = greeter.greet(user.name);
>     }
>
> _-- the Angular.js documentation_


Using modules allows us to rewrite controllers in the following way:

```js
angular.module('myApp').controller('loginCtrl', [
    '$scope', 'dep1', 'dep2',
    function ($scope, dep1, dep2) {
        'use strict';

        // scope defaults

        $scope.resetPassword = function () {
            // reset password button click handler 
        };
    }
]);
```

The beauty I find in the way Angular approaches controllers, is that you you need the controller `function` anyways, because that's used to inject the dependencies required by the controller, and it provides a new scope, strapping us from the need to wrap all of our script files in _self-invoking function expressions_ like `(function(){})()`.

### Dependency `$inject`ion

You might've noticed that in the earliest example, dependencies are injected using `$inject`. Most of the module API, on the other hand, allows you to pass either a `function`, or an `Array`, containing the list of dependencies, followed by the `function` that depends on those. This is _the one thing_ that I don't like in Angular, and it's probably the documentation's fault. Most of the examples in the documentation are treated as if you don't really need the `Array` form; but the thing is, you do. If you minify your code using a minifier, without running [ngmin](https://github.com/btford/ngmin "ngmin application pre-minifier") first, you're going to have a bad time.

Since you didn't explicitly declare your dependencies using the array form `['$scope',...]`, your clean-looking function arguments are going to get minified to something like `b,c,d,e`, effectively killing Angular's dependency injection capabilities. I consider this to be a gross mistake in the way they built the framework, in a similar way to my reasoning behind [strongly disliking Require.js](/2013/05/13/the-web-wars "The Web Wars") and their troubling AMD modules.

> If it's not going to work in production, what good is it for?

My fundamental problem with this kind of behavior is that they have code in their framework that is dead as soon as you go in production. That's fine for utilities like `console`, and error reporting, which are _useful_ during development, and have uses in production. It doesn't make any sense in syntactic sugar that just works during development.

These things infuriate me, but _enough ranting_. Talking of dollar signs...

### Cutting down on the jQuery proliferation

Going in, the application was "kind of Angular", in that it was just wrapped in Angular, but most DOM interaction happened through jQuery, rendering Angular pretty much moot.

> If I were to write an Angular.js application from scratch today, I wouldn't include jQuery right away. Forcing myself to use [angular.element](http://docs.angularjs.org/api/angular.element "Angular.js angular.element API") instead.

The `angular.element` API wraps `jQuery` if it's present, and it alternatively provides the Angular team's implementation of jQuery's API, called [jqLite](https://github.com/angular/angular.js/blob/master/src/jqLite.js "Angular.js jqLite source"). It's not that jQuery is evil, or that we need yet another implementation that somewhat reflects their API. It's just that using jQuery isn't very Angular.

Lets look at a concrete, and dumb, example. This uses jQuery to do class manipulation on the element where the controller has been declared.

```jade
div.foo(ng-controller='fooCtrl')
```

```js
angular.module('foo').controller('fooCtrl', function ($scope) {
    $('.foo').addClass('foo-init');

    $scope.$watch('something', function () {
        $('.foo').toggleClass('foo-something-else');
    });
});
```

However, we could be using Angular the way we're supposed to, instead.

```js
angular.module('foo').controller('fooCtrl', function ($scope, $element) {
    $element.addClass('foo-init');

    $scope.$watch('something', function () {
        $element.toggleClass('foo-something-else');
    });
});
```

The bottom line is you shouldn't be manipulating the DOM (changing attributes, adding event listeners) directly, or through jQuery. You should be [using directives](http://amitgharat.wordpress.com/2013/06/08/the-hitchhikers-guide-to-the-directive/ "The Hitchhiker's Guide to the Directive"), instead. That's an excellent article, go read it.

If you're still jQuery-lized, there's lots of articles you could read, such as this [migration guide](http://amitgharat.wordpress.com/2013/06/22/migration-guide-for-jquery-developers/ "Migration Guide for jQuery Developers"), and my article on [critically thinking](http://blog.ponyfoo.com/2013/07/09/getting-over-jquery "Getting Over jQuery") about whether to use jQuery.

I won't sit here and claim we've managed to remove jQuery altogether. We have other, more important goals in place, such as releasing the product. It was still worthwhile to remove as much jQuery spam as possible. Doing so simplified every controller we went through, we created directives that manipulate the DOM, and use `angular.element`, even if it just maps to jQuery today.

We have a dependency on the _oh-so-hideous_ [jQuery UI](http://jqueryui.com/ "jQuery UI") which makes me sick. We're clearly not using it just for the sake of dialogs, we have directives for that. But dragging, drag and drop, and in particular: being able to drag something and drop it in a sorted list, is just something that involves a lot of work if you're not using jQuery UI, there is no real alternative. The drag and drop problem has been solved, we could _(and probably should)_ be using [angular-dragon-drop](https://github.com/btford/angular-dragon-drop "btford/angular-dragon-drop on GitHub"), which is a really simple drag and drop implementation. [Sortable](https://github.com/angular-ui/ui-sortable "angular-ui-sortable on GitHub") on the other hand, just depends on jQuery UI.

### Organizing a code base

Another illness we had to deal with during our migration, was that **the entire code-base was crammed together in a single large file**. This file contained all the controllers, all services, directives, and code specific to each controller. I made it a point to break it down so that we had exactly one file per component. Right now, we have very few files with more than one component, and most of those happened because a directive used a service to share its data with the outside world.

Although unrelated to Angular, we also modularized our stylesheets. We added a two letter prefix to every class name we use in our code. This prefix represents the component the class belonged to. `pn-`, for example, represents classes that style the panels. `ly-` for layers, and so on. The immediate benefit this provides is that you don't have to think about class names anymore. Because you're namespacing them, it becomes much harder for you to accidentally re-use a class name. Another benefit is reduced nesting, we used to have selectors such as `#layoutEditor div.layer .handle div`, which now might be `.ly-handle-content`. The deepest "nesting" we have now only occurs on overloaded selectors such as `.fo-bar[disabled]:hover`, or at worst, something like `.fo-bar .br-baz`.

A few rules we laid out for this CSS class naming style were:

- Two characters to describe the component name. `ly-`, `dd-`, `dg-`, etc.
- Instead of nesting classes such as `.ly-foo .bar`, we gave `.bar` a more appropriate `.ly-foo-bar` name
- Avoid styling tags directly, use classes for everything. This reduces confusion and improves your ability to use semantic markup.
- Never use an ID in CSS.

> After implementing this component-oriented CSS declaration approach, I have a hard time thinking of going back to doing it "the class soup way".

Angular forces you to write good code, but on a deeper level than that, it forces you to think. A while later, it will either feel like a server-side implementation, or it'll become an unbearable hack-fest that you won't be able to stand on. _The choice is up to you._ 

### A Piece of Heaven

Let's decompose one of the pieces of our application, the layers.

```jade
div.cv-layer(
    ng-repeat="layer in page.layers | reverse",
    ap-layer,
    ng-mousedown="selectLayer(layer.id)",
    ng-mouseup="selectLayer(layer.id)",
    ng-dblclick="doubleClickLayer(layer)",
    ng-hide="layer.invisible"
)
```

Here, we're using the `cv-layer` class, given that the element is part of the _canvas_ component (the canvas is where our layers are drawn to, not to be confused with an HTML5 `canvas`). We're then using the [ngRepeat](http://docs.angularjs.org/api/ng.directive:ngRepeat "Angular.js ngRepeat directive") directive to create one of these elements per layer, in a `foreach` kind of fashion. It is passed through a `reverse` [filter](http://docs.angularjs.org/api/ng.filter:filter "Angular.js filters") we wrote, so that the last layer is visually on top. The **apLayer** directive is tasked with actually rendering the layer, whether it's an image, some text, HTML, or something else. The event directives (`ng-mousedown`, `ng-mouseup`, `ng-dblclick`) simply delegate the event to our _layer selection service_, which handles it from there. Lastly, `ngHide` doesn't really need a lot of explaining.

That's a **huge amount of functionality**, and Angular manages to make it look simple with readable HTML that sort of tells you what's going on. Furthermore, it allows you to separate the different concerns allowing you to write concise pieces of code that don't try to do everything at once. In summary, it reduces complexity, making the complex, simple. And the "hard to even fathom", possible.

Expect more posts about writing Angular code in the future. Particularly, I'm interested in discussing the corner-cases I ran into while upgrading our codebase, how we solved some, and how we worked around the rest.

  [1]: http://i.imgur.com/RSqiAkS.png
  [2]: http://i.imgur.com/I1ZDYeO.png