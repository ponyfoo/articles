# Angle brackets, rifle scopes

**Angular.js** presents a remarkable number of interesting design choices in its code-base. Two particularly interesting cases are the way in which scopes work; and how directives get compiled, and behave.

The first thing anyone is taught when approaching Angular for the first time is that directives are meant to interact with the DOM, or whatever does DOM manipulation for you, such as jQuery [_(Get over it!)_][1]. What immediately becomes **(and stays)** confusing for most, though, is the interaction between scopes, directives, and controllers. Particularly when we focus on scopes, and start factoring in the advanced concepts: **the digest cycle, isolate scopes, transclusion, and the different linking functions in directives.**

This article aims to navigate the salt marsh that are Angular scopes and directives, while providing an amusingly informative, in-depth read.

> The bar is high, but scopes are _sufficiently hard_ to explain. If I'm going to fail miserably at it, at least I'll throw in a few more promises I can't keep!

_Disclaimer: article based on [Angular v1.2.10 tree @ `caed2dfe4f`][2]._

  [1]: /2013/07/09/getting-over-jquery "Getting Over jQuery"
  [2]: https://github.com/angular/angular.js/tree/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d "Angular on GitHub"

[![angularjs.png][1]][2]

Angular uses scopes to abstract communication between directives and the DOM. Scopes also exist in the controller level. Scopes are plain old JavaScript objects _(POJO)_, which is fancy talk explaining that Angular does not heavily manipulate scopes, other than adding a bunch of properties, prefixed with one or two `$` symbols. The ones prefixed with `$$` aren't necessary as frequently, and using them is often a code smell, which can be avoided by having _a deeper understanding of the digest cycle_.

### Examination of a telescopic sight

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

# Don't buy a rifle scope without reading this

Here I've listed the properies yielded by that command, grouped by area of functionality. Let's start with the basics.

> 1. [**`$id`**][7] Uniquely identifies the scope
> 1. [**`$root`**][8] Root scope
> 1. [**`$parent`**][9] Parent scope, or `null` if `scope == scope.$root`
> 1. [**`$$childHead`**][10] First child scope, if any; or `null`
> 1. [**`$$childTail`**][11] Last child scope, if any; or `null`
> 1. [**`$$prevSibling`**][12] Previous sibling scope, if any; or `null`
> 1. [**`$$nextSibling`**][13] Next sibling scope, if any; or `null`

No surprises there. Navigating scopes like this would be utter non-sense. Sometimes accessing the `$parent` scope might seem right, but there are better, _less coupled_ ways to deal with parental communication than **tightly binding people-scopes together**. One such way is using event listeners, our next batch of scope properties!

## Events and partying: spreading the word

The properties described below let us publish events and subscribe to them. This is a pattern known as [PubSub][14], or just events.

> 1. [**`$$listeners`**][15] Event listeners [registered on the scope][70]
> 1. [**`$on(evt, fn)`**][16] Attaches an event listener `fn` named `evt`
> 1. [**`$emit(evt, args)`**][17] Fires event `evt`, roaring upward on the scope chain, triggering on the current scope and all `$parent`s, including the `$rootScope`
> 1. [**`$broadcast(evt, args)`**][18] Fires event `evt`, triggering on the current scope and all its children

When triggered, event listeners are passed an `event` object, and any arguments passed to the `$emit` or `$broadcast` function. There are many ways in which scope events can provide value.
....
.....


## Digesting change-sets

Angular bases its data-binding features in [a dirty-checking loop which tracks changes][34] and fires events when these change.

> 1. [**`$$phase`**][19] Current phase in the digest cycle. One of `[null, '$apply', '$digest']`
> 1. [**`$digest()`**][20] Executes the digest dirty-checking loop
> 1. [**`$apply(expr)`**][21] Parses and evaluates an expression, and then executes the digest loop

> 1. [**`$watch(watchExp, listener, objectEquality)`**][22] Adds a watch listener to the scope
> 1. [**`$watchCollection`**][23] Watches array items or object map properties
> 1. [**`$$watchers`**][24] Contains all the watches associated with the scope

> 1. [**`$eval`**][25] Parse and evaluate an scope expression immediately
> 1. [**`$evalAsync`**][26] Parse and evaluate an expression in the next digest
> 1. [**`$$asyncQueue`**][27] Async task queue, consumed on every digest
> 1. [**`$$postDigest(fn)`**][28] Executes `fn` after the next digest cycle
> 1. [**`$$postDigestQueue`**][29] Methods registered with `$$postDigest(fn)`


scopes
child


## Scope half-life

These are the last few, rather dull-looking, properties in a scope. They deal with the scope life cycle, and are mostly used for internal purposes, although there are cases where you may want to `$new` scopes by yourself.

> 1. [**$$isolateBindings**][30] Isolate scope bindings, e.g `{ options: '@megaOptions' }`. Very internal
> 1. [**$new(isolate)**][31] Creates a child scope, or an isolate scope, which won't inherit from its parent
> 1. [**$destroy**][32] Removes the scope from the scope chain. Scope and children won't receive events, and watches won't fire anymore
> 1. [**$$destroyed**][33] Has the scope been destroyed?

isolate.. directives!

directive
pre
post(default)

transclusion





Please comment on any issues regarding this article so _everyone can benefit_ from your feedback!

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
  [24]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L352 "scope.$$watchers method - Angular on GitHub"
  [25]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L744 "scope.$eval method - Angular on GitHub"
  [26]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L778 "scope.$evalAsync method - Angular on GitHub"
  [27]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L568-L577 "Digest async queue - Angular on GitHub"
  [28]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L792 "$$postDigest method - Angular on GitHub"
  [29]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L793 "The $$postDigest queue - Angular on GitHub"
  [30]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1412 "Assigning isolate bindings - Angular on GitHub"
  [31]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L176 "scope.$new - Angular on GitHub"
  [32]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L693 "scope.$destroy - Angular on GitHub"
  [33]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/rootScope.js#L699 "scope.$$destroyed - Angular on GitHub"
  [34]: http://stackoverflow.com/a/9693933/389745 "Data binding in Angular.js, Misko on StackOverflow"

