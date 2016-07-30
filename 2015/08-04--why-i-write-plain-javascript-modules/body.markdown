While `dragula` needs to be integrated into each one of those frameworks _(React, Angular, Backbone)_, doing so usually takes few lines of code. Here's one such example using `react`, `dragula`, and plain JavaScript someone posted on [a GitHub issue for dragula][1].

```js
dragula([...], {
  direction: 'horizontal',
}).on('cloned', function (clone) {
  clone.removeAttribute('data-reactid');
  var descendents = clone.getElementsByTagName('*');
  Array.prototype.slice.call(descendents).forEach(function (child) {
    child.removeAttribute('data-reactid');
  });
});
```

Could `dragula` get rid of `data-reactid` attributes when it clones things? _Allow me to answer that question using a meme._

![This is JavaScript!][2]

That being said, there's **no good reason** for `dragula` to do so. Why should `dragula` _know_ about React? Instead, we introduced a feature where whenever `dragula` clones a DOM element, it emits an event. In practical terms, this is about the same as getting rid of `data-reactid` ourselves, but we've moved the responsibility of knowing about that particular attribute to whoever uses React.

As I wrote this post, I made [`react-dragula`][4] into a library. It's just a wrapper -- _a mighty thin wrapper_ -- around `dragula`.

```js
function reactDragula () {
  return dragula.apply(this, <mark>atoa(arguments)</mark>).on('cloned', cloned);
  function cloned (clone) {
    rm(clone);
    atoa(clone.getElementsByTagName('*')).forEach(rm);
  }
  function rm (el) {
    el.removeAttribute('data-reactid');
  }
}
```

_<sub><mark>*</mark> `atoa` casts array-like objects into true arrays.</sub>_

Going through the trouble is worth it because if somebody wants to write an Angular directive for `dragula` it's also [easy for them to do so][5], and they seldom have to do anything to integrate it with Backbone -- Backbone isn't that "smart", so we don't have to rewrite our code to fit its awkward architecture.

# Portability across Frameworks

Without lower level libraries like `dragula` or [`xhr`][6] we'll end up reinventing the wheel for an entire afterlife of eternity in hell. Don't get me wrong -- I'm a big fan of reinventing the wheel. I've reinvented my fair share of wheels, Twitter reinvented RSS, etc. _But_, reinventing the wheel as a pointless exercise in porting a library from a framework to another is just wasteful.

When I wrote [`react-dragula`][4], I didn't have to fork `dragula` and repurpose it for React. When I wrote [`angular-dragula`][5], I didn't have to fork `dragula` either. I guess at this point you might argue that _"nobody seems to be forking things and repurposing them for other frameworks"_, but that's beside the point.

The point in question is that <mark>**developing a library that specifically targets a framework is a waste of your time**</mark>, because when you eventually move on to the next framework _(`this` is JavaScript, you will)_ you'll kick yourself over tightly coupling the library to the framework.

Sure, it involves a bit more of work and design thinking. You have to figure out how the library would work without your framework or choice _(or any framework for that matter)_ first -- and then wrap that in another module that molds them into the framework's paradigm.

The [`react-dragula`][4] example was too easy, right? All I did was add an event listener, getting rid of `data-reactid` attributes. Even though it **"looks easy" in hindsight**, the naïve approach would've been to just write it to conform to React right off the bat -- skipping the vanilla implementation entirely. Thus, we'd be ignoring the opportunity to provide hooks where we could later adjust the library to play nice with React, _saving ourselves from the painful experience_ of maintaining multiple libraries that do **essentially the same thing**.

In the case of [`angular-dragula`][5], I could've come up with a directive that just passed the options onto `dragula`, but that wouldn't have been very "Angular way"_ish_. Thus, I came up with the idea of trying to replicate the simple API in `dragula` in an Angular way. Instead of defining the containers as an array passed to `dragula`, you could also use directives to group different containers under the same `angular-dragula` instance.

The example below would use `angular-dragula` to create two instances of `drake`, identified as `'foo'`, and `'bar'`.

```html
<div ng-controller='ExampleCtrl'>
  <div dragula='"foo"'></div>
  <div dragula='"foo"'></div>
  <div dragula='"foo"'></div>
  <div dragula='"bar"'></div>
  <div dragula='"bar"'></div>
</div>
```

If you wanted to listen for events emitted by one of these `drake` instances, you could do so on the `$scope`, prefixing the event with the _"bag name"_ and a dot. Here again, I conformed to the Angular style by propagating `drake` events across the `$scope` chain, allowing the consumer to leverage Angular event engine. While events in [`dragula`][8] are raised using raw DOM elements, the events emitted across the `$scope` chain wrap them in `angular.element` calls, staying consistent with what you've come to expect of Angular components.

```js
app.controller('ExampleCtrl', ['$scope',
  function ($scope) {
    $scope.$on(<mark>'foo.over'</mark>, function (e, el, container) {
      <mark>container.addClass</mark>('dragging');
    });
    $scope.$on(<mark>'foo.out'</mark>, function (e, el, container) {
      container.removeClass('dragging');
    });
  }
]);
```

To configure the instances, you'd use the `dragulaService` in the controller for these containers. The example below makes it so that items in `foo` containers are copied instead of moved.

```js
app.controller('ExampleCtrl', ['$scope', 'dragulaService',
  function ($scope, dragulaService) {
    dragulaService.options($scope, 'foo', {
      copy: true
    });
  }
]);
```

In the future, I might **add more directives**, moving away from the _native [`dragula`][8] implementation_ and towards a more Angular way of handling things. For example, one such directive could be `dragula-accepts='method'`, and it could configure the `accepts` callback in such a way that the container where the directive is added to only accepts elements that return `true` when `method(item, source)` is invoked. A similar `dragula-moves='method'` directive could determine whether an item can be dragged away from a container, based on the result of calling `method(item)`.

A few more aspects of `dragula` can be _"molded into Angular"_ in this way.

While `dragula` doesn't have a native way of treating containers individually -- even when they take part of the same logical unit in the underlying implementation _(a `drake` can have as many containers as needed)_, we can build the functionality into `angular-dragula`. That helps us achieve [the _"Angular way"_][7] of writing directives that affect containers individually, rather than writing directives on a container that have knowledge of a series of unrelated DOM elements. Or, _even worse_, creating a directive where every immediate child element is a [`dragula`][8] container, constraining the use cases for the consumer.

> It might involve _some extra work_, but being able to reuse the code in any future projects makes **plain JavaScript modules** well worth your time.

# Portability across Platforms

Portability isn't just a matter of writing _vanilla client-side JavaScript libraries_. An equivalent case may be made for writing libraries that work well in both Node.js and the browser. Consider [`async`][9]: an amazing piece of software in Node.js, that's just <mark>garbage in the client-side.</mark> Granted, it was written well before ES6 modules _(or even Browserify)_ became a thing. A similar story can be told about `fast-url-parser`, a URL parser which underlies many server-side routers but is insanely large for the client-side. Talking about insane, I've used [`sanitize-html`][12] in countless opportunities to sanitize HTML on the server-side, but again -- repeat with me: **freaking _huge_ for the client-side** _(depends on [`htmlparser2`][11])_.

I've worked on reimplementing a few of those to work well on the client-side. Naturally, their server-side counterparts are more comprehensive, as they should be. Use cases for server-side JavaScript far outnumber what you need to do on a given site on the client-side for a single visitor. On the client-side, we can get away _(should get away)_ with much smaller libraries and modules.

Here are some examples.

- [`contra`][13] _(2k)_ is like `async` for the browser -- It's _modular_, too. You can just `require` individual methods _(ala `lodash`)_
- [`omnibox`][15] _(1.6k)_ is like `fast-url-parser` for the browser
- [`insane`][14] _(2k)_ is like `sanitize-html` _(100k)_ for the browser

Then again -- huge JavaScript libraries are only worrisome if we actually care about [performance when it comes to serving images][10] in the first place -- _right?_

> We're a far way from the _"universal JavaScript"_ fairytale we keep telling ourselves.

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][3]._ Remember to **subscribe** if you got this far!

[1]: https://github.com/bevacqua/dragula/issues/56 "Invariant Violation when using React.js #56"
[2]: https://i.imgur.com/8yKeaLf.jpg
[3]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
[4]: https://github.com/bevacqua/react-dragula "bevacqua/react-dragula on GitHub"
[5]: https://github.com/bevacqua/angular-dragula "bevacqua/angular-dragula on GitHub"
[6]: https://github.com/Raynos/xhr "Raynos/xhr on GitHub"
[7]: /articles/the-angular-way "The Angular Way on Pony Foo"
[8]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
[9]: https://github.com/caolan/async "caolan/async on GitHub"
[10]: /articles/fixing-web-performance "Fixing Web Performance on Pony Foo"
[11]: http://github.com/fb55/htmlparser2 "fb55/htmlparser2 on GitHub"
[12]: https://github.com/punkave/sanitize-html "punkave/sanitize-html on GitHub"
[13]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
[14]: https://github.com/bevacqua/insane "bevacqua/insane on GitHub"
[15]: https://github.com/petkaantonov/urlparser "petkaantonov/urlparser on GitHub"
[16]: https://github.com/bevacqua/omnibox "bevacqua/omnibox on GitHub"
