# Angle Brackets, Swift Directives

In [the previous part of this article][4], I discussed scope events and the behavior of the digest cycle. This time around, I'll talk about directives. Just as promised, this article will cover **isolate scopes, transclusion, linking functions, compilers, directive controllers, and more**.

If the following figure [_(source)_][2] looks unreasonably mind bending, then this article might be for you.

[![scope.png][1]][2]

_Disclaimer: article based on [Angular v1.2.10 tree @ `caed2dfe4f`][3]._

  [1]: http://i.stack.imgur.com/fkWHA.png
  [2]: http://docs.angularjs.org/guide/concepts
  [3]: https://github.com/angular/angular.js/tree/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d "Angular on GitHub"
  [4]: /2014/02/14/angle-brackets-rifle-scopes "Angle Brackets, Rifle Scopes"

[![angular.png][2]][3]

[Earlier][7], I listed each property Angular makes available on a scope, in order to explain how scopes work, and the digest cycle. I'll do the same for directives, but this time I'll be going through the properties of the object returned by a directive's factory function, and how each of those properties influences the directive we're defining.

The first thing of note is the name of the directive. Let's look at a brief example.

```js
angular.module('PonyDeli').directive('pieceOfFood', function () {
  var definition = {}; // <- these are the options we'll be discussing
  return definition;
});
```

Even though in the snippet above I'm defining a directive named `'pieceOfFood'`, I'll be using a dashed version of that name in the markup, instead. That is, if this directive was implemented as an attribute, I might need to reference it like below, in my HTML.

```html
<span piece-of-food></span>
```

By default, directives can only be triggered as attributes, but what if you want to change this behavior? You can use the `restrict` option.

> 1. [restrict][6] Defines how a directive may be applied in markup

```js
angular.module('PonyDeli').directive('pieceOfFood', function () {
  return {
    restrict: 'E'
  };
});
```

For some reason I can not fathom, they've somehow decided to obfuscate what's otherwise a verbose framework, and we ended up with single capital letters to define how a directive is restricted. Here's a list of [available `restrict` choices][5].

- `'A'` [Silent default][4, attributes are allowed, `<span piece-of-food></span>`
- `'E'` Elements are allowed, `<piece-of-food></piece-of-food>`
- `'C'` As a class name, `<span class='piece-of-food'></span>`
- `'M'` As a comment, `<!-- directive: piece-of-food -->`
- `'AE'` You can combine any of these to loosen up the restriction a bit

Don't ever use `'C'` or `'M'` to restrict your directives. Using `'C'` doesn't stand out in markup, and using `'M'` was meant for backwards compatibility. If you feel like being funny, though, you could make a case for setting `restrict` to `'ACME'`.

> Remember how the last time around I said **take advice with a pinch of salt**? Don't do that with mine, my advice is _awesome!_

Unfortunately, the rest of the properties in a directive definition object are much more obscure. Since we've already discussed scopes at length in the previous part

1. [scope]




1. [require]

1. [controller]
1. [controllerAs]


1. [template]
1. [templateUrl]

1. [replace]
1. [transclude]

1. [compile]
1. [link]

1. [priority]
1. [terminal]

https://github.com/angular/angular.js/wiki/Understanding-Scopes

scopes
child



isolate.. directives!
http://stackoverflow.com/a/14914798/389745
directive
pre
post(default)

transclusion

Please comment on any issues regarding this article, so _everyone can benefit_ from your feedback. Also, you should [follow me on Twitter][1]!

  [1]: https://twitter.com/nzgb "@nzgb on Twitter"
  [2]: http://i.imgur.com/LSVpcm1.png
  [3]: http://angularjs.org/ "Angular.js"
  [4]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L549 "`'A'` is the default `restrict` value - Angular on GitHub"
  [5]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L980 "Where can I add a directive? - Angular on GitHub"
  [6]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1590 "Directives restricted to a location - Angular on GitHub"
  [7]: /2014/02/14/angle-brackets-rifle-scopes "Angle Brackets, Rifle Scopes"







[angle-brackets angularjs directives front-end mvc]
