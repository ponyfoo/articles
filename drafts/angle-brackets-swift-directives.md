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

## What the hell is a directive?

Directives are _typically small_ components which are meant to interact with the DOM, in Angular. They are used as an abstraction layer on top of the DOM, and most manipulation can be achieved without touching DOM elements, wrapped in jQuery, jqLite, or otherwise. This is accomplished by using expressions, and other directives, to achieve the results you want.

Directives in Angular core can bind an element property **(such as visibility, class list, inner text, inner HTML, or value)** to a scope property or expression. Most notably, these bindings will be updated whenever changes in the scope are digested, using watches. Similarly, and in the opposite direction, DOM attributes can we "watched" using an `$observe` function, which will trigger a callback whenever the watched property changes.

## Creating a directive

Earlier, I [listed each property available on a scope][7] in Angular, and I used that to explain  the digest mechanism, and how scopes operate. I'll do the same for directives, but this time I'll be going through the properties of the object returned by a directive's factory function, and how each of those properties influences the directive we're defining.

The first thing of note is the name of the directive. Let's look at a brief example.

```js
angular.module('PonyDeli').directive('pieceOfFood', function () {
  var definition = { // <- these are the options we'll be discussing
    template: // ...
  };
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
    restrict: 'E',
    template: // ...
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

Unfortunately, the rest of the properties in a directive definition object are much more obscure.

1. [scope][8] Sets how a directive interacts with **the `$parent` scope**

Since we've already discussed scopes at length in the previous part, learning how to use the `scope` property **properly** shouldn't be _all that excruciating_. Let's start with the default value, `scope: false`, where the scope chain remains unaffected: you get **whatever scope is found** on the associated element, following the rules I've [outlined in the previous part][7].

Leaving the scope chain untouched is obviously useful when your directive doesn't interact with the scope at all, but that rarely happens. A much more common scenario where not touching the scope is useful, is when creating a directive that has no reason to be instanced more than once on any given scope, and which just interacts with a single scope property, _the directive name_. This is most declarative when combined with `restrict: 'A'`, the default `restrict` value.

```js
angular.module('PonyDeli').directive('pieceOfFood', function () {
  return {
    template: '{{pieceOfFood}}',
    link: function (scope, element, attrs) {
      attrs.$observe('pieceOfFood', function (value) {
        scope.pieceOfFood = value;
      });
    }
  };
});
```

```html
<body ng-app='PonyDeli'>
  <span piece-of-food='Fish & Chips'></span>
</body>
```

[Grab the pen.][9] There's a few things to note here, which we haven't discussed yet. You'll learn more about the `link` property later in the article. For the time being you can think of it as _a controller that runs for each instance of the directive_.

In the directive linking function we can access `attrs`, which is a collection of attributes present on `element`. This collection has a special method, called `$observe()`, which will fire a callback [whenever a property changes][11]. Without watching the attribute for changes, the property wouldn't ever make it to the scope, and we wouldn't be able to bind to it in our template.

We can twist the code above, making it much more useful, by adding `scope.$eval` into the mix. Remember how it could be used to evaluate an expression against a scope? Look at the code below to get a better idea of how that could help us.

```js
var deli = angular.module('PonyDeli', []);

deli.controller('foodCtrl', function ($scope) {
  $scope.piece = 'Fish & Chips';
});

deli.directive('pieceOfFood', function () {
  return {
    template: '{{pieceOfFood}}',
    link: function (scope, element, attrs) {
      attrs.$observe('pieceOfFood', function (value) {
        scope.pieceOfFood = scope.$eval(value);
      });
    }
  };
});
```

```html
<body ng-app='PonyDeli' ng-controller='foodCtrl'>
  <span piece-of-food='piece'></span>
</body>
```

[In this case][10], I'm evaluating the attribute value, `piece`, against the scope, which defined `$scope.piece` at the controller. Of course, you could use a template like `{{piece}}` directly, but that would require specific knowledge about which property in the scope you want to track. This pattern provides _a little more flexibility_, although you're still going to be sharing the scope **across all directives**, which can lead to _unexpected behavior_ if you were to try adding more than one directive in the same scope.

## Playful Child Scopes

You could solve that issue by creating a child scope, which inherits prototypically from its parent. In order to create a child scope, you merely need to declare `scope: true`.

```js
var deli = angular.module('PonyDeli', []);

deli.controller('foodCtrl', function ($scope) {
  $scope.pieces = ['Fish & Chips', 'Potato Salad'];
});

deli.directive('pieceOfFood', function () {
  return {
    template: '{{pieceOfFood}}',
    scope: true,
    link: function (scope, element, attrs) {
      attrs.$observe('pieceOfFood', function (value) {
        scope.pieceOfFood = scope.$eval(value);
      });
    }
  };
});
```

```html
<body ng-app='PonyDeli' ng-controller='foodCtrl'>
  <p piece-of-food='pieces[0]'></p>
  <p piece-of-food='pieces[1]'></p>
</body>
```

As you can see, we're now [able to use multiple instances][12] of the directive, and get the desired behavior, because each directive is creating its own scope. However, there's a limitation: multiple directives on an element all get the same scope.

>  If multiple directives on the same element request a new scope, only one new scope is created.

## Lonely, Isolate Scope

One last option is creating a local, or isolate scope. The difference between an isolate scope and a child scope, is that local scopes don't inherit from their parent _(but it's still accessible on `scope.$parent`)_. You can declare an isolate scope like this: `scope: {}`. You can add properties to the object, which get data-bound to the parent scope, but accessible on the local scope. Much like `restrict`, isolate scope properties have a terse but confusing syntax, where you can use symbols like `&`, `@`, and `=` to define how the property is bound.

## Choose Your Weapon. `@`, `&`, or `=`?

What do those symbols mean, then? The examples I coded, enumerated below, might help you decode them.

#### Attribute Observer, `@`

Using `@` binds to the result of [observing an attribute][14] against the parent scope.

```html
<body ng-app='PonyDeli' ng-controller='foodCtrl'>
  <p note='You just bought some {{type}}'></p>
</body>
```

```js
deli.directive('note', function () {
  return {
    template: '{{note}}',
    scope: {
      note: '@'
    }
  };
});
```

This is [equivalent to observing the attribute][17] for changes, and updating our local scope.

```js
deli.directive('note', function () {
  return {
    template: '{{note}}',
    scope: {},
    link: function (scope, element, attrs) {
      attrs.$observe('note', function (value) {
        scope.note = value;
      });
    }
  };
});
```

Of course, using the `@` notation is much more "Angular".

#### Expression Builder, `&`

Using `&` gives you an [expression evaluating function][16], in the context of the parent scope.

```html
<body ng-app='PonyDeli' ng-controller='foodCtrl'>
  <p note='"You just bought some " + type'></p>
</body>
```

```js
deli.directive('note', function () {
  return {
    template: '{{note()}}',
    scope: {
      note: '&'
    }
  };
});
```

This one is a bit more uncommon, as [it parses the expression in the attribute][18] once, building a reusable function.

```js
deli.directive('note', function ($parse) {
  return {
    template: '{{note()}}',
    scope: {},
    link: function (scope, element, attrs) {
      var parentGet = $parse(attrs.note);

      scope.note = function (locals) {
        return parentGet(scope.$parent, locals);
      };
    }
  };
});
```

### Bi-directional Binding, `=`

Using `=` sets up [bi-directional binding][15] between the local and parent scopes.

```html
<body ng-app='PonyDeli' ng-controller='foodCtrl'>
  <button countable='clicks'></button>
  <span>Got {{clicks}} clicks!</span>
</body>
```

```js
deli.directive('countable', function () {
  return {
    template:
      '<button ng-disabled="!remaining">' +
        'Click me {{remaining}} more times! ({{count}})' +
      '</button>',
    replace: true,
    scope: {
      count: '=countable'
    },
    link: function (scope, element, attrs) {
      scope.remaining = 10;

      element.bind('click', function () {
        scope.remaining--;
        scope.count++;
        scope.$apply();
      });
    }
  };
});
```

Bi-directional binding is [_quite a bit_ more complicated][19] than `&` or `@`.

```
deli.directive('countable', function ($parse) {
  return {
    template:
      '<button ng-disabled="!remaining">' +
        'Click me {{remaining}} more times! ({{count}})' +
      '</button>',
    replace: true,
    scope: {},
    link: function (scope, element, attrs) {

      // you're definitely better off using '&'
      var compare;
      var parentGet = $parse(attrs.countable);
      if (parentGet.literal) {
        compare = angular.equals;
      } else {
        compare = function(a,b) { return a === b; };
      }
      var parentSet = parentGet.assign; // or throw
      var lastValue = scope.count = parentGet(scope.$parent);
      scope.$watch(function () {
        var value = parentGet(scope.$parent);
        if (!compare(value, scope.count)) {
          if (!compare(value, lastValue)) {
            scope.count = value;
          } else {
            parentSet(scope.$parent, value = scope.count);
          }
        }
        return lastValue = value;
      }, null, parentGet.literal);

      scope.remaining = 10;

      element.bind('click', function () {
        scope.remaining--;
        scope.count++;
        scope.$apply();
      });
    }
  };
});
```

You may omit the property name if you're going to use that as the key in your local scope. That is to say, `pieceOfFood: '='` is a short-hand form for `pieceOfFood: '=pieceOfFood'`, they are equivalent.



....

...

usage on each of the above. then proceed!






1. [template]
1. [templateUrl]

1. [link]

1. [require]

1. [controller]
1. [controllerAs]

1. [replace]
1. [transclude]

1. [compile]

1. [priority]
1. [terminal]

https://github.com/angular/angular.js/wiki/Understanding-Scopes

pre
post(default)
transclusion





## Further Reading

Here's some additional resources you can read to further extend your comprehension of Angular.

- [Angle Brackets _(Part I)_, Rifle Scopes][7]
- [How to choose between no new scope, child scope, or isolate scope?][20]


http://stackoverflow.com/a/14914798/389745
Please comment on any issues regarding this article, so _everyone can benefit_ from your feedback. Also, you should [follow me on Twitter][1]!

  [1]: https://twitter.com/nzgb "@nzgb on Twitter"
  [2]: http://i.imgur.com/LSVpcm1.png
  [3]: http://angularjs.org/ "Angular.js"
  [4]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L549 "`'A'` is the default `restrict` value - Angular on GitHub"
  [5]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L980 "Where can I add a directive? - Angular on GitHub"
  [6]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1590 "Directives restricted to a location - Angular on GitHub"
  [7]: /2014/02/14/angle-brackets-rifle-scopes "Angle Brackets, Rifle Scopes"
  [8]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L931-L936 "Determining the scope of a directive - Angular on GitHub"
  [9]: http://codepen.io/bevacqua/pen/iexmJ "A piece of food on CodePen"
  [10]: http://codepen.io/bevacqua/pen/ilgtv "Interpreting food-scopes on CodePen"
  [11]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L747-L755 "Observing attributes - Angular on GitHub"
  [12]: http://codepen.io/bevacqua/pen/JrLev "Directive child scopes on CodePen"
  [13]: http://codepen.io/bevacqua/pen/HAnba "Isolate scope in directives on CodePen"
  [14]: http://codepen.io/bevacqua/pen/IxvBc "Buying vegetables on CodePen"
  [15]: http://codepen.io/bevacqua/pen/sDmAo "Bi-directional binding on CodePen"
  [16]: http://codepen.io/bevacqua/pen/glhso "Expression evaluation on CodePen"
  [17]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1417-L1419 "Observing an attribute value - Angular on GitHub"
  [18]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1463-L1466 "Parsing a getter expression - Angular on GitHub"
  [19]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1429-L1459 "Bi-directional binding implementation - Angular on GitHub"
  [20]: http://stackoverflow.com/a/14914798/389745 "When writing a directive, how do I decide if a need no new scope, a new child scope, or a new isolate scope? on StackOverflow"




[angle-brackets angularjs directives front-end mvc]




// TODO links to this article in the previous part
