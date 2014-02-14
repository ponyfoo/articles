# Angle Brackets, Rifle Scopes

**Angular.js** presents a remarkable number of interesting design choices in its code-base. Two particularly interesting cases are the way in which scopes work, and how directives behave.

The first thing anyone is taught when approaching Angular for the first time is that directives are meant to interact with the DOM, or whatever does DOM manipulation for you, such as jQuery [_(Get over it!)_][1]. What immediately becomes **(and stays)** confusing for most, though, is the interaction between scopes, directives, and controllers. Particularly when we focus on scopes, and start factoring in the advanced concepts: **the digest cycle, isolate scopes, transclusion, and the different linking functions in directives.**

This _(two-part)_ article aims to navigate the salt marsh that are Angular scopes and directives, while providing an amusingly informative, in-depth read. In the first part, this one, I'll focus on scopes, and the life-cycle of an Angular application. The second part is focused on directives

> The bar is high, but scopes are _sufficiently hard_ to explain. If I'm going to fail miserably at it, at least I'll throw in a few more promises I can't keep!

If the following figure [_(source)_][4] looks unreasonably mind bending, then this article might be for you.

[![mindbender.png][3]][4]

_Disclaimer: article based on [Angular v1.2.10 tree @ `caed2dfe4f`][2]._

  [1]: /2013/07/09/getting-over-jquery "Getting Over jQuery"
  [2]: https://github.com/angular/angular.js/tree/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d "Angular on GitHub"
  [3]: http://i.stack.imgur.com/O1iSG.png
  [4]: https://github.com/angular/angular.js/wiki/Understanding-Scopes "'Understanding' Scopes - Angular wiki on GitHub"

[![angularjs.png][1]][2]

Angular uses scopes to abstract communication between directives and the DOM. Scopes also exist in the controller level. Scopes are plain old JavaScript objects _(POJO)_, which is fancy talk explaining that Angular does not heavily manipulate scopes, other than adding a bunch of properties, prefixed with one or two `$` symbols. The ones prefixed with `$$` aren't necessary as frequently, and using them is often a code smell, which can be avoided by having _a deeper understanding of the digest cycle_.

# What kind of scopes are we talking about?

In Angular slang, a _"scope"_ is not what you might be used to, when thinking about JavaScript code, or even programming in general. Usually, scopes are used to refer to the bag in a piece of code which holds the _context_, variables, and so on.

> In most languages variables are held in imaginary bags, which are defined by curly braces `{}`, or code blocks. This is known as [_block scoping_][35]. JavaScript, in contrast, deals in [_lexical scoping_][36], which pretty much means the bags are defined by functions, or the global object, rather than code blocks.
>
> Bags can contain any number of smaller bags. Each bag can access the candy _(sweet, sweet variables)_ inside its parent bag (and its parent's parent, and so on), but they can't poke holes in smaller, or child bags.

As a quick and dirty example, let's examine the function below.

```js
function eat (thing) {
  console.log('Eating a ' + thing);
}

function nuts (peanut) {
  var hazelnut = 'hazelnut';

  function seeds () {
    var almond = 'almond';
    eat(hazelnut); // I can reach into the nuts bag!
  }

  // Inaccessible almond is inaccessible.
  // Almonds are not nuts.
}
```

I won't dwell on [`this` matter][37] any longer, as these are not the scopes people refer to, when talking about Angular.

## Scope inheritance in Angular.js

Scopes in Angular are also context, but _on Angular terms_. In Angular, a scope is associated to an element, while an element is not necessarily _directly associated_ with a scope. Elements are assigned a scope is one of the following ways.

A scope is created on an element by a controller, or a directive (directives don't always introduce new scopes).

```html
<nav ng-controller='menuCtrl'>
```

If a scope isn't present on the element, then it's inherited from its parent.

```html
<nav ng-controller='menuCtrl'>
  <a ng-click='navigate()'>Click Me!</a> <!-- also <nav>'s scope -->
</nav>
```

If the element isn't part of an `ng-app`, then **it doesn't belong to an scope at all**.

```html
<head>
  <h1>Pony Deli App</h1>
</head>
<main ng-app='PonyDeli'>
  <nav ng-controller='menuCtrl'>
    <a ng-click='navigate()'>Click Me!</a>
  </nav>
</main>
```

To figure out an element's scope, try to think of elements **recursively inside-out** following the three rules I've just outlined. Does it create a new scope? That's its scope. Does it have a parent? Check the parent, then. Is it not part of an `ng-app`? Tough luck, no scope.

You can **(and most definitely should)** use developer tools magic to easily figure out the scope for an element.

## Examination of a telescopic sight

I'll walk through a few properties in a typical scope as a way to introduce concepts, before moving on to explaining how digests work and behave internally. I'll also let you in on how I'm getting to these properties. First, I'll open Chrome and navigate to the application I'm working on, which is written in Angular. Then I'll inspect on an element, and open the developer tools.

> Did you know that [`$0` gives you _access to the last selected element_][3] in the Elements pane? `$1` gives you access to the previously selected element, and so on.
>
> I prognosticate **you'll use `$0` the most**, particularly when working with Angular.

For any given DOM element, `angular.element` wraps that in either [jQuery][4] or _jqLite_, their [little own mini-jQuery][5]. Once wrapped, you get access to a `scope()` function which returns, you guessed it, the Angular scope associated with that element. Combining that with `$0`, I find myself using the following command quite often.

```js
angular.element($0).scope()
```

_Of course, if you just know you're using jQuery, `$($0).scope()` will work just the same. `angular.element` works every time, **regardless of jQuery**._

Then I'm able to inspect the scope, assert that it's the scope I expected, and whether the property values match what I was expecting, as well. Super useful. Let's see what special properties are available on a typical scope.

```js
for(o in $($0).scope())o[0]=='$'&&console.log(o)
```

That's good enough, I'll go over each property, clustering them by functionality, and going over each portion of Angular's scoping philosophy.

# Don't buy a rifle scope without reading this

Here I've listed the properies yielded by that command, grouped by area of functionality. Let's start with the basic ones, which merely provide scope navigation.

> 1. [`$id`][7] Uniquely identifies the scope
> 1. [`$root`][8] Root scope
> 1. [`$parent`][9] Parent scope, or `null` if `scope == scope.$root`
> 1. [`$$childHead`][10] First child scope, if any; or `null`
> 1. [`$$childTail`][11] Last child scope, if any; or `null`
> 1. [`$$prevSibling`][12] Previous sibling scope, if any; or `null`
> 1. [`$$nextSibling`][13] Next sibling scope, if any; or `null`

No surprises there. Navigating scopes like this would be utter non-sense. Sometimes accessing the `$parent` scope might seem appropriate, but there are always better, _less coupled_, ways to deal with parental communication than **tightly binding people-scopes together**. One such way is using event listeners, our next batch of scope properties!

## Events and partying: spreading the word

The properties described below let us publish events and subscribe to them. This is a pattern known as [PubSub][14], or just events.

> 1. [`$$listeners`][15] Event listeners registered on the scope
> 1. [`$on(evt, fn)`][16] Attaches an event listener `fn` named `evt`
> 1. [`$emit(evt, args)`][17] Fires event `evt`, roaring upward on the scope chain, triggering on the current scope and all `$parent`s, including the `$rootScope`
> 1. [`$broadcast(evt, args)`][18] Fires event `evt`, triggering on the current scope and all its children

When triggered, event listeners are passed an `event` object, and any arguments passed to the `$emit` or `$broadcast` function. There are many ways in which scope events can provide value.

A directive might use events to announce something important happened. Check out this sample directive, where a button can be clicked to announce you feel like eating food of some type.

```js
angular.module('PonyDeli').directive('food', function () {
  return {
    scope: { // I'll come back to directive scopes later
      type: '=type'
    },
    template: '<button ng-click="eat()">I want to eat some {{type}}!</button>',
    link: function (scope, element, attrs) {
      scope.eat = function () {
        letThemHaveIt();
        scope.$emit('food.click', scope.type, element);
      };

      function letThemHaveIt () {
        // do some fancy UI things
      }
    }
  };
});
```

I like namespacing my events, and so should you. It avoids name collisions, and it's clear where events originate from, or what event you're subscribing to. Imagine you have an interest in analytics, and want to track clicks on `food` elements using [Mixpanel][38]. That would actually be a reasonable need, and there's no reason why that should be polluting your directive, or your controller. You could put together a directive which does _the food-clicking analytics-tracking_ for you, in a nicely self-contained manner.

```js
angular.module('PonyDeli').directive('foodTracker', function (mixpanelService) {
  return {
    link: function (scope, element, attrs) {
      scope.$on('food.click', function (e, type) {
        mixpanelService.track('food-eater', type);
      });
    }
  };
});
```

The service implementation is not relevant here, as it would merely wrap Mixpanel's client-side API. The HTML would look like below, and I threw in a controller, to hold all of the food types I want to serve in my deli. The [`ng-app`][39] directive helps Angular to auto-bootstrap my application, as well. Rounding the example up, I added an `ng-repeat` directive so I can render all of my food without repeating myself, it'll just loop through `foodTypes`, available on `foodCtrl`'s scope.

```html
<ul ng-app='PonyDeli' ng-controller='foodCtrl' food-tracker>
  <li food type='type' ng-repeat='type in foodTypes'></li>
</ul>
```

```js
angular.module('PonyDeli').controller('foodCtrl', function ($scope) {
  $scope.foodTypes = ['onion', 'cucumber', 'hazelnut'];
});
```

The fully working example is [hosted on CodePen][40].

That's a good example on paper, but you need to think about whether you need an event anyone can subscribe to. Maybe a service will do? In this case, it could go either way. You could argue that you need events because you don't know who else is going to subscribe to `food.click`, and that means it'd be more _"future-proof"_ to use events. You could also say that the `food-tracker` directive doesn't have a reason to be, as it doesn't interact with the DOM or even the scope at all, other than to listen to an event which you could replace with a service.

Both thoughts would be correct, in the given context. As more components need to be `food.click`-aware, it may feel clearer that _events are the way to go_. In reality, though, events are most useful when you actually need to bridge the gap between two scopes (or more), and other factors aren't as important.

As we'll see when we inspect directives more closely in the upcoming second part of this article, events aren't even necessary for scopes to communicate. A child scope may read from its parent by binding to it, and it can also update those values.

> There's rarely a good reason to host events to help children communicate better with their parent.

Siblings often have a harder time communicating with each other, and they often do so through a parent they have in common. That generally translates into broadcasting from `$rootScope`, and listening on the interested siblings, like below.

```js
angular.module('PonyDeli').controller('foodCtrl', function ($rootScope) {
  $scope.foodTypes = ['onion', 'cucumber', 'hazelnut'];
  $scope.deliver = function (req) {
    $rootScope.$broadcast('delivery.request', req);
  };
});

angular.module('PonyDeli').controller('deliveryCtrl', function ($scope) {
  $scope.$on('delivery.request', function (e, req) {
    $scope.received = true; // deal with the request
  });
});
```

```html
<body ng-app='PonyDeli'>
  <div ng-controller='foodCtrl'>
    <ul food-tracker>
      <li food type='type' ng-repeat='type in foodTypes'></li>
    </ul>
    <button ng-click='deliver()'>I want to eat that!</button>
  </div>
  <div ng-controller='deliveryCtrl'>
    <span ng-show='received'>
      A monkey has been dispatched. You shall eat soon.
    </span>
  </div>
</body>
```

This one is [also on CodePen][41].

Over time you'll learn to lean towards events or services accordingly. I could say that you should use events when you expect view models to change in response to `event`, and you ought to use services otherwise, when you don't expect view model changes. Sometimes the response is a mixture of both, where an action triggers an event which calls a service, or a service which broadcasts an event on `$rootScope`. It depends on each situation, and you should analyze it as such, rather than attempting to nail down the **elusive one-size-fits-all solution**.

If you have two components which communicate through `$rootScope`, you might prefer to use **`$rootScope.$emit`** _(rather than `$broadcast`)_ and `$rootScope.$on`. That way, the event will only spread among `$rootScope.$$listeners`, and it won't waste time looping through every children of `$rootScope`, which _you just know_ won't have any listeners for that event. Here's an example service using `$rootScope` to provide events without limiting itself to a particular scope. It provides a subscribe method which allows consumers to register event listeners, and it might do things internally, which trigger that event.

```js
angular.module('PonyDeli').factory("notificationService", function ($rootScope) {
  function notify (data) {
    $rootScope.$emit("notificationService.update", data);
  }

  function listen (fn) {
    $rootScope.$on("notificationService.update", function (e, data) {
      fn(data);
    });
  }

  // anything that might have a reason
  // to emit events at later points in time
  function load () {
    setInterval(notify.bind(null, 'Something happened!'), 1000);
  }

  return {
    subscribe: listen,
    load: load
  };
});
```

You guessed right! This one is [also on CodePen][42].

Enough events versus services banter, shall we move on to some other properties?

## Digesting change-sets

> **Understanding this intimidating process is the cornerstone to understanding Angular.**

Angular bases its data-binding features in [a dirty-checking loop which tracks changes][34], and fires events when these change. This is simpler than it sounds. No, really. It is! Let me quickly go over each of the core components of the `$digest` cycle. Firstly, there's the `scope.$digest` method. This method recursively digests changes in a scope and its children.

> 1. [`$digest()`][20] Executes the digest dirty-checking loop
> 1. [`$$phase`][19] Current phase in the digest cycle. One of `[null, '$apply', '$digest']`

It should be noted that you need to be careful about triggering digests, because attempting to do so when you're already in a digest phase will cause Angular to blow up in a mysterious haze of unexplainable phenomena. In other words, it'll be pretty hard to pinpoint the root cause of the issue.

Let's take a look at what [the documentation][43] has to say, regarding `$digest`.

> #### `$digest()`
>
> Processes all of the [watchers][44] of the current scope and its children. Because a [watcher][44]'s listener can change the model, the [$digest()][43] keeps calling the [watchers][44] until no more listeners are firing. This means that it is possible to get into an infinite loop. This function will throw `'Maximum iteration limit exceeded.'` if the number of iterations exceeds 10.
>
> Usually, you don't call [$digest()][43] directly in [controllers][45] or in [directives][46]. Instead, you should call [$apply()][47] (typically from within a [directives][46]), which will force a [$digest()][43].

So, a `$digest` processes all watchers, and then the watchers those watchers trigger, until nothing else triggers a watch. There's two questions left to understand this loop.

- What the hell is _a watcher_?
- Who triggers a `$digest`!?

Answering both of these questions can be made out to be **as simple or as complicated** as the person explaining them to you feels like. I'll begin talking about watchers, and I'll let you draw _your own conclusions_.

If you've read this far, you probably _already know_ what a watcher is. You've probably used [`scope.$watch`][22], and maybe even used [`scope.$watchCollection`][23]. The `$$watchers` property has all the watchers on a scope.

> 1. [`$watch(watchExp, listener, objectEquality)`][22] Adds a watch listener to the scope
> 1. [`$watchCollection`][23] Watches array items or object map properties
> 1. [`$$watchers`][24] Contains all the watches associated with the scope

Watchers are the single most important aspect of an Angular application's data-binding capabilities, but Angular needs our help in order to trigger those watchers, because otherwise it can't effectively update data-bound variables appropriately. Consider the following example.

```js
angular.module('PonyDeli').controller('foodCtrl', function ($scope) {
  $scope.prop = 'initial value';
  $scope.dependency = 'nothing yet!';

  $scope.$watch('prop', function (value) {
    $scope.dependency = 'prop is "' + value + '"! such amaze';
  });

  setTimeout(function () {
    $scope.prop = 'something else';
  }, 1000);
});
```

```html
<body ng-app='PonyDeli'>
  <ul ng-controller='foodCtrl'>
    <li ng-bind='prop'></li>
    <li ng-bind='dependency'></li>
  </ul>
</body>
```

So you have `'initial value'`, and expect the second HTML line to change to `'prop is "something else"! such amaze'` after a second. Right? Even more interesting, you'd at the very least expect the first line to change to `'something else'`! Why doesn't it? That's not a watcher... or is it?

Actually, a lot of what you do in the HTML markup ends up _creating a watcher_. In this case, [each `ng-bind` directive created a watcher][54] on the property. It will update the HTML of the `<li>`, whenever `prop` and `dependency` change, similarly to how our watch will change the property itself.

That way, you can now think of your code as having three watches, one for each `ng-bind` directive, and the one in the controller. How is Angular supposed to know the property is updated, after the timeout? You could tell it, just by adding a manual digest to the timeout callback.

```js
setTimeout(function () {
  $scope.prop = 'something else';
  $scope.$digest();
}, 1000);
```

Here's a CodePen [without the `$digest`][55], and one that [does `$digest`][56], after the timeout. The more [Angular way][50] to do it, however, would be using the [`$timeout` service][57] instead of `setTimeout`. It provides some error handling, and executes `$apply()`.

> 1. [`$apply(expr)`][21] Parses and evaluates an expression, then executes the digest loop **on `$rootScope`**

In addition to executing the digest on every scope, `$apply` provides error handling functionality, as well. If you're trying to tune your performance, then using `$digest` may be warranted, but I'd stay away from it until you feel really comfortable with how Angular works internally.

We're back to the second question, now.

> - Who triggers a `$digest`!?

Digests are triggered internally in strategic places all over the Angular code-base. They are triggered either directly or by calls to `$apply()`, like we've observed in the `$timeout` service. Most directives, both those found in Angular core and those out in the wild, trigger digests. Digests fire your watchers, and watchers update your UI. That's the basic idea, anyways.

There's a pretty good resource with best practices in the Angular Wiki, which you can find linked at the bottom of this article.

> #### A word of advice, regarding advice
>
> Ever since I've come aboard the Angular boat, I've read lots of advice on how to structure your code, what to do; and what not to do, when working with Angular. The truth is that you need to take advice regarding Angular with a pinch of salt. There's lots of bad advice clinging to the web, from back when Angular wasn't the mature framework that it is today, or written by people who don't have a clue what they're talking about.
>
> Even [the good advice][58] is **one guy's opinion**, and you shouldn't stick to whatever worked for someone else, just because they've blogged about them. I do believe that you should read about what other people to be best practices, and embrace them, if you feel they're adequate. But don't turn them into your unbreakable mantra, because they'll break you.

I've explained how watches and the digest loop interact with each other. Below, I listed properties related to the digest loop, which you can find on a scope. These help you parse text expressions through Angular's compiler, or execute pieces of code at different points of the digest cycle.

> 1. [`$eval(expression, locals)`][25] Parse and evaluate an scope expression immediately
> 1. [`$evalAsync(expression)`][26] Parse and evaluate an expression at a later point in time
> 1. [`$$asyncQueue`][27] Async task queue, consumed on every digest
> 1. [`$$postDigest(fn)`][28] Executes `fn` after the next digest cycle
> 1. [`$$postDigestQueue`][29] Methods registered with `$$postDigest(fn)`

Phew, that's it. It wasn't that bad, was it?

## The `Scope` is dead, long live the `Scope`!

These are the last few, rather dull-looking, properties in a scope. They deal with the scope life cycle, and are mostly used for internal purposes, although there are cases where you may want to `$new` scopes by yourself.

> 1. [$$isolateBindings][30] Isolate scope bindings, e.g `{ options: '@megaOptions' }`. Very internal
> 1. [$new(isolate)][31] Creates a child scope, or an isolate scope, which won't inherit from its parent
> 1. [$destroy][32] Removes the scope from the scope chain. Scope and children won't receive events, and watches won't fire anymore
> 1. [$$destroyed][33] Has the scope been destroyed?

Isolate scopes? What is this madness? The second part of this article will be dedicated to directives, and it'll cover **isolate scopes, transclusion, linking functions, compilers, directive controllers, and more**. Can't wait? _You'll just have to._

## Further Reading

Here's some additional resources you can read to further extend your comprehension of Angular.

- [The Angular Way][50]
- [Anti Patterns][48]
- [Best Practices][49]
- [TodoMVC Angular.js Example][51]
- [Training Videos from John Lindquist][52]
- [ng-newsletter][59]
- [Using scope.$watch and scope.$apply][60]

Please comment on any issues regarding this article, so _everyone can benefit_ from your feedback. Also, you should [follow me on Twitter][53]!

  [1]: http://i.imgur.com/LSVpcm1.png
  [2]: http://angularjs.org/ "Angular.js"
  [3]: https://developers.google.com/chrome-developer-tools/docs/commandline-api#0_-_4 "Chrome Developer Tools CLI API Reference"
  [4]: http://jquery.com/ "jQuery - write less, do more."
  [5]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/jqLite.js "jqLite - Angular on GitHub"
  [6]: http://docs.angularjs.org/api/ng.$rootScope.Scope "ng.$rootScope.Scope"
  [7]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L127 "Assigning a unique identifier to a scope - Angular on GitHub"
  [8]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L131 "Setting scope.$root - Angular on GitHub"
  [9]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L197 "Setting scope.$parent - Angular on GitHub"
  [10]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#204 "Setting scope.$childHead - Angular on GitHub"
  [11]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#202 "Setting scope.$childTail - Angular on GitHub"
  [12]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L199 "Setting scope.$prevSibling - Angular on GitHub"
  [13]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L201 "Setting scope.$nextSibling - Angular on GitHub"
  [14]: http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern "Publish – Subscribe pattern on Wikipedia"
  [15]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L889 "Adding event $$listeners - Angular on GitHub"
  [16]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L886-L906 "Listening $on events - Angular on GitHub"
  [17]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L932-L975 "Spreading events upward with $emit - Angular on GitHub"
  [18]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L1000-L1048 "Spreading events downward with $broadcast - Angular on GitHub"
  [19]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L1056 "Digest phases - Angular on GitHub"
  [20]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L549 "scope.$digest method - Angular on GitHub"
  [21]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L842-L857 "scope.$apply method - Angular on GitHub"
  [22]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L319 "scope.$watch method - Angular on GitHub"
  [23]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L415 "scope.$watchCollection method - Angular on GitHub"
  [24]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L352 "scope.$$watchers collection - Angular on GitHub"
  [25]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L744 "scope.$eval method - Angular on GitHub"
  [26]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L778 "scope.$evalAsync method - Angular on GitHub"
  [27]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L568-L577 "Digest async queue - Angular on GitHub"
  [28]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L792 "$$postDigest method - Angular on GitHub"
  [29]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L793 "The $$postDigest queue - Angular on GitHub"
  [30]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1412 "Assigning isolate bindings - Angular on GitHub"
  [31]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L176 "scope.$new method - Angular on GitHub"
  [32]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L693 "scope.$destroy method - Angular on GitHub"
  [33]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L699 "scope.$$destroyed - Angular on GitHub"
  [34]: http://stackoverflow.com/a/9693933/389745 "Data binding in Angular.js, Misko on StackOverflow"
  [35]: http://en.wikipedia.org/wiki/Scope_(computer_science)#Block_scope "Block scoping in Computer Science - Wikipedia"
  [36]: http://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scoping "Lexical scoping - Wikipedia"
  [37]: /2013/12/04/where-does-this-keyword-come-from "Where does this keyword come from?"
  [38]: https://mixpanel.com/ "Mixpanel Analytics Platform"
  [39]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/Angular.js#L1144 "The ng-app directive - Angular on GitHub"
  [40]: http://codepen.io/bevacqua/pen/qmBGd "Separation of concerns using scope events"
  [41]: http://codepen.io/bevacqua/pen/CzGla "Siblings talking to each other"
  [42]: http://codepen.io/bevacqua/pen/HsBCa "Service events, using $rootScope"
  [43]: http://docs.angularjs.org/api/ng.$rootScope.Scope#methods_$digest "$digest on Angular.js documentation"
  [44]: http://docs.angularjs.org/api/ng.$rootScope.Scope#methods_$watch "$watch on Angular.js documentation"
  [45]: http://docs.angularjs.org/api/ng.directive:ngController "ng-controller on Angular.js documentation"
  [46]: http://docs.angularjs.org/api/ng.$compileProvider#methods_directive "Directives on Angular.js documentation"
  [47]: http://docs.angularjs.org/api/ng.$rootScope.Scope#methods_$apply "$apply on Angular.js documentation"
  [48]: https://github.com/angular/angular.js/wiki/Anti-Patterns "Angular.js Anti Patterns"
  [49]: https://github.com/angular/angular.js/wiki/Best-Practices "Angular.js Best Practices"
  [50]: /2013/08/27/the-angular-way "The Angular Way"
  [51]: http://todomvc.com/architecture-examples/angularjs "TodoMVC Angular Example"
  [52]: https://egghead.io/ "Training Videos on egghead.io"
  [53]: https://twitter.com/nzgb "@nzgb on Twitter"
  [54]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/directive/ngBind.js#L54-L62 "ng-bind directive creating a watch - Angular on GitHub"
  [55]: http://codepen.io/bevacqua/pen/lLbtI "No digest means no watchers will fire"
  [56]: http://codepen.io/bevacqua/pen/vwDoz "Using scope.$digest fixes the issue"
  [57]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/timeout.js#L36 "$timeout service - Angular on GitHub"
  [58]: http://www.benlesh.com/2013/08/angularjs-watch-digest-and-apply-oh-my.html "AngularJS: $watch, $digest and $apply"
  [59]: http://www.ng-newsletter.com/ "The free, weekly newsletter of the best AngularJS links created by the AngularJS community"
  [60]: http://stackoverflow.com/a/15113029/389745 "Using scope.$watch and scope.$apply, StackOverflow"

[angle-brackets angularjs scope internals front-end mvc]
