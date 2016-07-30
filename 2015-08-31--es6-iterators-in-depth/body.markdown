## Iterator Protocol and Iterable Protocol

> _There's a lot of new, intertwined terminology here. Please bear with me as I get some of these explanations out of the way!_

JavaScript gets two new protocols in ES6, _Iterators_ and _Iterables_. In plain terms, you can think of protocols as _conventions_. As long as you follow a determined convention in the language, you get a side-effect. The _iterable_ protocol allows you to define the behavior when JavaScript objects are being iterated. Under the hood, deep in the world of JavaScript interpreters and language specification _keyboard-smashers_, we have the [`@@iterator`][1] method. This method underlies the _iterable_ protocol and, in the real world, you can assign to it using something called "the _"well-known"_ `Symbol.iterator` Symbol".

We'll get back to [what Symbols are][13] _later in the series_. Before losing focus, you should know that the `@@iterator` method is called **once, whenever an object needs to be iterated**. For example, at the beginning of a [`for..of`][2] loop _(which we'll also get back to in a few minutes)_, the `@@iterator` will be asked for an _iterator_. The returned _iterator_ will be used to obtain values out of the object.

Let's use the snippet of code found below as a crutch to understand the concepts behind iteration. The first thing you'll notice is that I'm making my object an iterable by assigning to it's mystical `@@iterator` property through the `Symbol.iterator` property. I can't use the symbol as a property name directly. Instead, I have to wrap in square brackets, meaning it's a computed property name that evaluates to the `Symbol.iterator` _expression_ -- as you might recall from the [article on object literals][3]. The object returned by the method assigned to the `[Symbol.iterator]` property must adhere to the _iterator_ protocol. The _iterator_ protocol defines how to get values out of an object, and we must return an `@@iterator` that adheres to _iterator_ protocol. The protocol indicates we must have an object with a `next` method. The `next` method takes no arguments and it should return an object with these two properties.

- `done` signals that the sequence has ended when `true`, and `false` means there may be more values
- `value` is the current item in the sequence

In my example, the iterator method returns an object that has a finite list of items and which emits those items until there aren't any more left. The code below is an iterable object in ES6.

```js
var foo = {
  <mark>[Symbol.iterator]</mark>: () => ({
    items: ['p', 'o', 'n', 'y', 'f', 'o', 'o'],
    <mark>next</mark>: function next () {
      return {
        done: this.items.length === 0,
        value: this.items.shift()
      }
    }
  })
}
```

To actually iterate over the object, we could use `for..of`. How would that look like? See below. The `for..of` iteration method is also new in ES6, and it settles the everlasting war against looping over JavaScript collections and randomly finding things that didn't belong in the result-set you were expecting.

```js
for (<mark>let pony of foo</mark>) {
  console.log(pony)
  // <- 'p'
  // <- 'o'
  // <- 'n'
  // <- 'y'
  // <- 'f'
  // <- 'o'
  // <- 'o'
}
```

You can use `for..of` to iterate over any object that adheres to the _iterable_ protocol. In ES6, that includes arrays, any objects with an user-defined `[Symbol.iterator]` method, [_generators_][12], DOM node collections from [`.querySelectorAll`][4] and friends, etc. If you just want to _"cast"_ any iterable into an array, a couple of terse alternatives would be using the [spread operator][5] and `Array.from`.

```js
console.log(<mark>[...foo]</mark>)
// <- ['p', 'o', 'n', 'y', 'f', 'o', 'o']
console.log(<mark>Array.from(foo)</mark>)
// <- ['p', 'o', 'n', 'y', 'f', 'o', 'o']
```

To recap, our `foo` object adheres to the _iterable_ protocol by assigning a method to `[Symbol.iterator]` _-- anywhere in the prototype chain for `foo` would work_. This means that the object is _iterable_: it can be iterated. Said method returns an object that adheres to the _iterator_ protocol. The iterator method is called once whenever we want to start iterating over the object, and the returned _iterator_ is used to pull values out of `foo`. To iterate over iterables, we can use `for..of`, the [spread operator][6], or `Array.from`.

## What Does This All Mean?

In essence, the selling point about iteration protocols, `for..of`, `Array.from`, and the spread operator is that they provide expressive ways to effortlessly iterate over collections and array-likes _(such as `arguments`)_. Having the ability to define how any object may be iterated is huge, because it enables any libraries like [lo-dash][7] to converge under a protocol the language natively understands _-- iterables._ This is **huge**.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Lodash&#39;s chaining wrapper is now an iterator and iterable:&#10;var w = _({ a: 1, b: 2 });&#10;Array.from(w);&#10;// =&gt; [1, 2]</p>&mdash; John-David Dalton (@jdalton) <a href="https://twitter.com/jdalton/status/638238228869283841">August 31, 2015</a></blockquote>

Just to give you another example, remember how I always complain about jQuery wrapper objects not being [true arrays][8], or how `document.querySelectorAll` doesn't return a true array either? If jQuery implemented the iterator protocol on their collection's prototype, then you could do something like below.

```js
for (let item of $('li')) {
  console.log(item)
  // <- the <li> wrapped in a jQuery object
}
```

Why wrapped? Because it's more expressive. You could easily iterate as deep as you need to.

```js
for (let list of $('ul')) {
  for (let item of list.find('li')) {
    console.log(item)
    // <- the <li> wrapped in a jQuery object
  }
}
```

This brings me to an important aspect of iterables and iterators.

## Lazy in Nature

Iterators are _lazy in nature_. This is fancy-speak for saying that the sequence is accessed one item at a time. It can even be an infinite sequence -- a legitimate scenario with many use cases. Given that iterators are lazy, having jQuery wrap every result in the sequence with their wrapper object wouldn't have a big upfront cost. Instead, a wrapper is created each time a value is pulled from the _iterator_.

How would an infinite iterator look? The example below shows an iterator with a `1..Infinity` range. Note how it will never yields `done: true`, signaling that the sequence is over. Attempting to cast the iterable `foo` object into an array using either `Array.from(foo)` or `[...foo]` would crash our program, since the sequence _never ends_. We must be very careful with these types of sequences as they can crash and burn our Node process, or the human's browser tab.

```js
var foo = {
  [Symbol.iterator]: () => {
    var i = 0
    return { next: () => (<mark>{ value: ++i }</mark>) }
  }
}
```

The correct way of working with such an iterator is with an escape condition that prevents the loop from going infinite. The example below loops over our infinite sequence using `for..of`, but it breaks the loop as soon as the value goes over `10`.

```js
for (let pony of foo) {
  if (pony > 10) {
    <mark>break</mark>
  }
  console.log(pony)
}
```

> The iterator doesn't really _know_ that the sequence is infinite. In that regard, this is similar to the [halting problem][11] -- there is no way of knowing whether the sequence is infinite or not in code.
>
> [![The halting problem depicted by XKCD][9]][10]
>
> We **usually have a good idea** about whether a sequence is _finite or infinite_, since we construct those sequences. Whenever we have an infinite sequence it's up to us to add an escape condition that ensures our program won't crash in an attempt to loop over every single value in the sequence.

_Come back tomorrow for [a discussion about generators!][12]_

  [1]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Iteration_protocols "Iteration Protocols on MDN"
  [2]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/for...of "for..of is on MDN"
  [3]: /articles/es6-object-literal-features-in-depth "ES6 Object Literal Features in Depth on Pony Foo"
  [4]: https://developer.mozilla.org/en-US/docs/Web/API/Element/querySelectorAll "Element.querySelectorAll() on MDN"
  [5]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
  [6]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
  [7]: http://lodash.com/docs "Lodash documentation"
  [8]: http://ponyfoo.com/articles/how-to-avoid-objectprototype-pollution "How To Avoid Object.prototype Pollution on Pony Foo"
  [9]: https://imgs.xkcd.com/comics/halting_problem.png
  [10]: https://xkcd.com/1266/ "Halting Problem on XKCD"
  [11]: https://en.wikipedia.org/wiki/Halting_problem "Halting Problem on Wikipedia"
  [12]: /articles/es6-generators-in-depth "ES6 Generators in Depth on Pony Foo"
  [13]: /articles/es6-symbols-in-depth "ES6 Symbols in Depth on Pony Foo"
