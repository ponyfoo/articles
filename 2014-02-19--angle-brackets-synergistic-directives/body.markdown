# Creating a directive

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

> 1. [`restrict`][6] Defines how a directive may be applied in markup

```js
angular.module('PonyDeli').directive('pieceOfFood', function () {
  return {
    restrict: 'E',
    template: // ...
  };
});
```

For some reason I can not fathom, they've somehow decided to obfuscate what's otherwise a verbose framework, and we ended up with single capital letters to define how a directive is restricted. Here's a list of [available `restrict` choices][5].

- `'A'` [Silent default][4], attributes are allowed, `<span piece-of-food></span>`
- `'E'` Elements are allowed, `<piece-of-food></piece-of-food>`
- `'C'` As a class name, `<span class='piece-of-food'></span>`
- `'M'` As a comment, `<!-- directive: piece-of-food -->`
- `'AE'` You can combine any of these to loosen up the restriction a bit

Don't ever use `'C'` or `'M'` to restrict your directives. Using `'C'` doesn't stand out in markup, and using `'M'` was meant for backwards compatibility. If you feel like being funny, though, you could make a case for setting `restrict` to `'ACME'`.

> Remember how the last time around I said **take advice with a pinch of salt**? Don't do that with mine, my advice is _awesome!_

Unfortunately, the rest of the properties in a directive definition object are much more obscure.

1. [`scope`][8] Sets how a directive interacts with **the `$parent` scope**

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

# Playful Child Scopes

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

You may omit the property name if you're going to use that as the key in your local scope. That is to say, `pieceOfFood: '='` is a short-hand form for `pieceOfFood: '=pieceOfFood'`, they are equivalent.

### Choose Your Weapon. `@`, `&`, or `=`?

What do those symbols mean, then? The examples I coded, enumerated below, might help you decode them.

##### Attribute Observer, `@`

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

This is [equivalent to observing the attribute][17] for changes, and updating our local scope. Of course, using the `@` notation is much more "Angular".

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

Attribute observers are most useful when **consuming options for a directive**. If we want to change the directive's behavior based on changing options, though, it _might make more sense_ to write the `attrs.$observe` line ourselves, rather than have Angular [_do that internally_][17], and creating a watch on our end, which would be slower.

In those cases, merely replacing `scope.note = value`, in the `$observe` handler shown above, into whatever you would've put on the `$watch` listener, should do.

> It's important to keep in mind that, when dealing with `@`, we're **talking about observing and attribute**, instead of _binding to the parent scope_.

##### Expression Builder, `&`

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

Below I outlined how you might implement that same functionality inside the linking function, if you weren't aware of `&`. This one is a tad more lengthy than `@`, as [it parses the expression in the attribute][18] once, building a reusable function.

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

Expression builders are, as we can see, generate a method which queries the parent scope. You can execute the method whenever you'd like, and even watch it for output changes. This method should be treated as a read-only query on a parent expression, and as such would be most useful in two scenarios. The first one is if you need to watch for changes on the parent scope, in which case you would set up a watch on the function expression, `note()`, which is in essence, what we did in the example above.

The other situation in which this might come in handy is when you need access to a method on the parent scope. Suppose the parent scope has a method which refreshes a table, while your local scope represents a table row. When the table row is deleted, you might want to refresh the table. If the button is in the child scope, then it would make sense using a `&` binding to access the refresh functionality on the parent scope. That's just a contrived example, as you might prefer to use events for that kind of thing, or maybe even structure your application in some way where complicating things like that could be avoided.

##### Bi-directional Binding, `=`

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

      // you're definitely better off just using '&'

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

      // I told you!

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

This form of data-binding is _arguably the most useful of all three_. In this case, the parent scope property is kept in sync with the local scope. Whenever the local scope value is updated, it gets set on the parent scope. Likewise, whenever the parent scope value changes, the local scope gets an update. The most straightforward scenario I've got for you as to when this could be useful, would be whenever you have a child scope which is used to represent a sub-model of the parent scope. Think of your typical [CRUD _(Create, Read, Update, Delete)_][23] table. The table as a whole would be the parent scope, whereas each row would be contained in an isolate directive, which binds to the row's data model through a two-way `=` binding. This would allow for modularity while still being able to effectively communicate between the master table and its children.

That took a lot of words, but I think I've managed to sum up how the `scope` property works when declaring directives, and what the _most common use cases_ are. Time to move on to other properties in the directive definition object, shall we?

# Sensible View Templates

Directives are most effective when they contain small, reusable snippets of HTML. That's where the true power of directives comes from. These templates can be provided in plain text, or as a resource Angular will query when bootstrapping the directive.

1. [`template`][25] Is how you would provide the view template as plain text. `template: '<span ng-bind="message" />'`
1. [`templateUrl`][26] Allows you to provide the url to an HTML template. `templateUrl: /partials/message.html`

Using a `templateUrl` to separate the HTML from your linking function _is awesome_. Making an AJAX request whenever you want to initialize a directive for the first time, **not so much**. However, you can circumvent the AJAX request if you pre-fill the `$templateCache` with a build task, such as [grunt-angular-templates][24]. That would be the **"best of both worlds"**.Separation of concerns without the extra overhead of AJAX calls.

You could also provide a `function (tElement, tAttrs)` as the `template`, but this is **neither necessary nor useful.**

1. [`replace`][28] Should the template be inserted as a child element, or inlined?

The documentation for this property is _woefully confusing_.

> ##### `replace`
>
> Specify where the template should be inserted. Defaults to `false`.
>
> - `true` The template will replace the current element
> - `false` The template will replace the contents of the current element

So when replace is `false` the directive actually replaces the element? That doesn't sound right. If you [check out this pen][28], then you'll find out that the element simply gets appended if `replace: false`, and it gets [sort of replaced][27], if `replace: true`.

As a rule of thumb, try and keep replacements to a minimum. Directives should strive to keep interferance with the DOM as close as possible to none, whenever possible, of course.

Directives are compiled, which results in a pre-linking function, and a post-linking function. You can define the code which returns these functions, or just provide them. Here are the different ways in which you can provide linking functions. I warn you, this is yet another one of those _"features"_ in Angular which I feel is more of a drawback, because **it confuses the hell out of new-comers for little to no gain.** Behold.

```js
compile: function (templateElement, templateAttrs) {
  return {
    pre: function (scope, instanceElement, instanceAttrs, controller) {
      // pre-linking function
    },
    post: function (scope, instanceElement, instanceAttrs, controller) {
      // post-linking function
    }
  }
}
```

```js
compile: function (templateElement, templateAttrs) {
  return function (scope, instanceElement, instanceAttrs, controller) {
    // post-linking function
  };
}
```

```js
link: {
  pre: function (scope, instanceElement, instanceAttrs, controller) {
    // pre-linking function
  },
  post: function (scope, instanceElement, instanceAttrs, controller) {
    // post-linking function
  }
}
```

```js
link: function (scope, instanceElement, instanceAttrs, controller) {
  // post-linking function
}
```

Actually, you could even forget about the directive definition object we've been discussing thus far, and merely return a post-linking function. However, this isn't recommended even by Angular peeps, so you better stay away from it.

```js
deli.directive('food', function () {
  return function (scope, element, attrs) {
    // post-linking function
  };
});
```

Before proceeding, here's an important note from the Angular documentation I'd like you to take a look at.

> **Note:** The template instance and the link instance may be different objects if the template has been cloned. For this reason it is not safe to do anything other than DOM transformations that apply to all cloned DOM nodes within the compile function. Specifically, DOM listener registration should be done in a linking function rather than in a compile function.

Compile functions currently take in a third parameter, a _transclude linking function_, but it's deprecated. Also, you shouldn't be altering the DOM during compile functions (on `templateElement`). Just do yourself a favor and avoid `compile` entirely, provide pre-linking and post-linking functions directly. Most often, a post-linking function is just enough, which is what you're using when you assign a `link` function to the definition object.

I have a rule for you here. Always use a post-linking function. If a scope absolutely needs to be pre-populated before the DOM is linked, then do _just that_ in the pre-linking function, but bind the functionality in the post-linking function, like you normally would have. You'll rarely need to do this, but I think it's still worth mentioning.

```js
link: {
  pre: function (scope, element, attrs, controller) {
    scope.requiredThing = [1, 2, 3];
  },
  post: function (scope, element, attrs, controller) {
    scope.squeal = function () {
      scope.$emit("squeal");
    };
  }
}
```

1. [`controller`][30] A controller instance on the directive

Directives can have controllers, which makes sense, because directives can create a scope. The controller is shared among all directives on the scope, and it is accessible as the fourth argument in linking functions. These controllers are a useful communication channel across directives on the same scoping level, which can be contained in the directive itself.

1. [`controllerAs`][29] Controller alias to reference it in the template

Using a controller alias allows for using the controller within the template itself, as it'll be made available in the scope.

1. [`require`][31] _I'll throw if you don't link some other directive(s) on this element!_

The documentation for require is surprisingly straightforward, so I'll just cheat and paste that here.

> Require another directive and inject its controller as the fourth argument to the linking function. The `require` takes a string name (or array of strings) of the directive(s) to pass in. If an array is used, the injected argument will be an array in corresponding order. If no such directive can be found, or if the directive does not have a controller, then an error is raised. The name can be prefixed with:
>
> - `(no prefix)` Locate the required controller on the current element. Throw an error if not found
> - `?` Attempt to locate the required controller or pass `null` to the `link` fn if not found
> - `^` Locate the required controller by searching the element's parents. Throw an error if not found
> - `?^` Attempt to locate the required controller by searching the element's parents or pass `null` to the `link` fn if not found

1. [`priority`][32] Defines the order in which directives are applied.

Cheating time!

> When there are multiple directives defined on a single DOM element, sometimes it is necessary to specify the order in which the directives are applied. The `priority` is used to sort the directives before their `compile` functions get called. Priority is defined as a number. Directives with greater numerical `priority` are compiled first. Pre-link functions are also run in priority order, but post-link functions are run in reverse order. The order of directives with the same priority is _undefined_. The default priority is `0`.

1. [`terminal`][33] Prevents further processing of directives

> If set to true then the current `priority` will be the last set of directives which will execute (any directives at the current priority will still execute as the order of execution on same `priority` is _undefined_).

# Transcluding for much win

1. [`transclude`][34] Compiles the content of the element and makes it available to the directive.

I saved the best _(worst?)_ for last. This property allows two values, for more fun and less profits. You can either set it to `true`, which enables transclusion, or to `'element'`, in which case the whole element, including any directives defined at lower priority, get transcluded.

At a high level, transclusion allows the consumer of a directive to define a snippet of HTML which can then be included into some part of the directive, using an `ng-transclude` directive. This sounds way too complicated, and it's only _kind of complicated_. An example might make things clearer for you.

```js
angular.module('PonyDeli').directive('transclusion', function () {
  return {
    restrict: 'E',
    template:
      '<div ng-hide="hidden" class="transcluded">' +
        '<span ng-transclude></span>' +
        '<span ng-click="hidden=true" class="close">Close</span>' +
      '</div>',
    transclude: true
  };
});
```

```html
<body ng-app='PonyDeli'>
  <transclusion>
    <span>The plot thickens!</span>
  </transclusion>
</body>
```

You can [check it out on CodePen][38], of course. What happens when you try to get scopes into the mix? We'll, the content which gets transcluded inside the directive will still respond to the parent content, correctly, even though it's placed inside the directive, and even if the directive presents an isolate scope. This is what you'd expect, because the transcluded content is defined in the consuming code, which belongs to the parent scope, and not the directive's scope. The directive still binds to it's local scope, as usual.

```js
var deli = angular.module('PonyDeli', []);

deli.controller('foodCtrl', function ($scope) {
  $scope.message = 'The plot thickens!';
});

deli.directive('transclusion', function () {
  return {
    restrict: 'E',
    template:
      '<div ng-hide="hidden" class="transcluded">' +
        '<span ng-transclude></span>' +
        '<span ng-click="hidden=true" class="close" ng-bind="close"></span>' +
      '</div>',
    transclude: true,
    scope: {},
    link: function (scope) {
      scope.close = 'Close';
    }
  };
});
```

```html
<body ng-app='PonyDeli' ng-controller='foodCtrl'>
  <transclusion>
    <span ng-bind='message'></span>
  </transclusion>
</body>
```

You can [find that one on CodePen][39] as well. There you have it, transclusion, demystified.

## Further Reading

Here's some additional resources you can read to further extend your comprehension of Angular.

- [Angle Brackets _(Part I)_, Rifle Scopes][7]
- [How to choose between no new scope, child scope, or isolate scope?][20]
- [Transclusion Basics (screencast)][36]
- [`transclude: true` vs `transclude: 'element'`][35]
- [Understanding Directives, `ng-repeat` and `compile`][37]

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
  [23]: http://en.wikipedia.org/wiki/Create,_read,_update_and_delete "CRUD on Wikipedia"
  [24]: https://github.com/ericclemmons/grunt-angular-templates "grunt-angular-templates on GitHub"
  [25]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1236 "Setting a view template - Angular on GitHub"
  [26]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1663 "Using templateUrl to fetch a template using AJAX - Angular on GitHub"
  [27]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1244-L1279 "'Replacing' an element with a directive - Angular on GitHub"
  [28]: http://codepen.io/bevacqua/pen/iteGj "Replaced versus inlined directives"
  [29]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1503-L1505 "Assigning a controllerAs alias - Angular on GitHub"
  [30]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1492 "Controller instances can be shared on the scope - Angular on GitHub"
  [31]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1362-L1365 "Require or whine - Angular on GitHub"
  [32]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1748-L1753 "Priority Sort - Angular on GitHub"
  [33]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1167-L1169 "Enforcing terminal priority - Angular on GitHub"
  [34]: https://github.com/angular/angular.js/blob/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d/src/ng/compile.js#L1195-L1232 "Transcluded directives - Angular on GitHub"
  [35]: http://stackoverflow.com/a/18457319/389745 "When to use transclude 'true' and transclude 'element' - StackOverflow"
  [36]: https://egghead.io/lessons/angularjs-transclusion-basics "Angular.js Transclusion Basics - Egghead.io"
  [37]: http://liamkaufman.com/blog/2013/05/13/understanding-angularjs-directives-part1-ng-repeat-and-compile/ "Understanding AngularJS Directives Part 1: Ng-repeat and Compile"
  [38]: http://codepen.io/bevacqua/pen/rlmwB "Basic Transclusion on CodePen"
  [39]: http://codepen.io/bevacqua/pen/lFHoE "Transcluded Scopes on CodePen"
