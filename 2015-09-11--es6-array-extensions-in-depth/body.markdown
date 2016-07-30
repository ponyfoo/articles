# Upcoming `Array` Methods

There's plenty to choose from. Over the years, libraries like Underscore and Lodash spoke loudly about features we were missing in the language, and now we have a ton more tools in the [functional array][1] arsenal at our disposal.

First off, there's a couple of static methods being added.

- [`Array.from`](#arrayfrom) -- create `Array` instances from arraylike objects like `arguments` or iterables
- [`Array.of`](#arrayof)

Then there's a few methods that help you manipulate, fill, and filter arrays.

- [`Array.prototype.copyWithin`](#arrayprototypecopywithin)
- [`Array.prototype.fill`](#arrayprototypefill)
- [`Array.prototype.find`](#arrayprototypefind)
- [`Array.prototype.findIndex`](#arrayprototypefindindex)

There's also the methods [related to the iterator protocol][2].

- [`Array.prototype.keys`](#arrayprototypekeys)
- [`Array.prototype.values`](#arrayprototypevalues)
- [`Array.prototype.entries`](#arrayprototypeentries)
- [`Array.prototype[Symbol.iterator]`](#arrayprototype-symboliterator)

There's a few more methods coming in ES2016 _(ES7)_ as well, but we won't be covering those today.

- `Array.prototype.includes`
- `Array.observe`
- `Array.unobserve`

Let's get to work!

# `Array.from`

This method has been long overdue. Remember the quintessential example of converting an arraylike into an actual array?

```js
function cast ()
  return <mark>Array.prototype.slice.call</mark>(arguments)
}
cast('a', 'b')
// <- ['a', 'b']
```

Or, a shorter form perhaps?

```js
function cast ()
  return <mark>[].slice.call</mark>(arguments)
}
```

To be fair, we've already explored even more terse ways of doing this at some point during the [ES6 in depth series][3]. For instance you could use the [spread operator][4]. As you probably remember, the spread operator leverages the [iterator protocol][2] to produce a sequence of values in arbitrary objects. The downside is that the objects we want to cast with spread **must implement** `@@iterator` through `Symbol.iterator`. Luckily for us, `arguments` does implement the iterator protocol in ES6.

```js
function cast ()
  return [<mark>...</mark>arguments]
}
```

Another thing you could be casting through the spread operator is DOM element collections like those returned from `document.querySelectorAll`. Once again, this is made possible thanks to ES6 adding conformance to the iterator protocol to `NodeList`.

```js
[<mark>...</mark>document.querySelectorAll('div')]
// <- [<div>, <div>, <div>, ...]
```

What happens when we try to cast a jQuery collection through the spread operator? Actually, you'll **get an exception** because they haven't implemented [`Symbol.iterator`][2] quite yet. You can try this one on [jquery.com][5] in Firefox.

```js
[...$('div')]
<mark>TypeError: $(...)[Symbol.iterator] is not a function</mark>
```

The new `Array.from` method is different, though. It doesn't _only_ rely on iterator protocol to figure out how to pull values from an object. It also has support for arraylikes out the box.

```js
Array.from(<mark>$('div')</mark>)
// <- [<div>, <div>, <div>, ...]
```

The one thing you cannot do with either `Array.from` nor the spread operator is to pick a start index. Suppose you wanted to pull every `<div>` after the first one. With `.slice.call`, you could do it like so:

```js
[].slice.call(document.querySelectorAll('div')<mark>, 1</mark>)
```

Of course, there's nothing stopping you from using `.slice` _after_ casting. This is probably way easier to read, and looks more like functional programming, so there's that.

```js
Array.from(document.querySelectorAll('div'))<mark>.slice(1)</mark>
```

`Array.from` actually has three arguments, _but only the `input` is required_. To wit:

- `input` -- the arraylike or iterable object you want to cast
- `map` -- a mapping function that's executed on every item of `input`
- `context` -- the `this` binding to use when calling `map`

With `Array.from` we cannot slice, but we can dice!

```js
function typesOf () {
  return Array.from(arguments, value => typeof value)
}
typesOf(null, [], NaN)
// <- ['object', 'object', 'number']
```

Do note that you could also just combine [rest parameters][4] and `.map` if you were just dealing with `arguments`. In this case in particular, we may be better off just doing something like the snippet of code found below.

```js
function typesOf (<mark>...</mark>all) {
  return all.map(value => typeof value)
}
typesOf(null, [], NaN)
// <- ['object', 'object', 'number']
```

In some cases, like the case of jQuery we saw earlier, it makes sense to use `Array.from`.

```js
Array.from($('div'))
// <- [<div>, <div>, <div>, ...]
Array.from($('div'), el => el.id)
// <- ['', 'container', 'logo-events', 'broadcast', ...]
```

I guess you get the idea.

# `Array.of`

This method is exactly like the first incarnation of `cast` we played with in our analysis of [`Array.from`](#arrayfrom).

```js
Array.of = function of () {
  return Array.prototype.slice.call(arguments)
}
```

You can't just replace `Array.prototype.slice.call` with `Array.of`. They're different animals.

```js
Array.prototype.slice.call(<mark>[1, 2, 3]</mark>)
// <- [1, 2, 3]
Array.of(1, 2, 3)
// <- [1, 2, 3]
```

You can think of `Array.of` as an alternative for `new Array` that doesn't have the `new Array(length)` overload. Below you'll find some of the strange ways in which `new Array` behaves thanks to its single-argument `length` overloaded constructor. If you're confused about the `undefined x ${number}` notation in the browser console, that's indicating there are [array holes][7] in those positions.

```js
new Array()
// <- []
new Array(undefined)
// <- [undefined]
new Array(1)
// <- <mark>[undefined x 1]</mark>
new Array(3)
// <- <mark>[undefined x 3]</mark>
new Array(1, 2)
// <- [1, 2]
new Array(-1)
// <- <mark>RangeError: Invalid array length</mark>
```

In contrast, `Array.of` has more consistent behavior because it doesn't have the special `length` case.

```js
Array.of()
// <- []
Array.of(undefined)
// <- [undefined]
Array.of(1)
// <- [1]
Array.of(3)
// <- [3]
Array.of(1, 2)
// <- [1, 2]
Array.of(-1)
// <- [-1]
```

There's not a lot to add here -- let's move on.

# `Array.prototype.copyWithin`

This is the most obscure method that got added to `Array.prototype`. I suspect use cases lie around [buffers and typed arrays][8] _-- which we'll cover at some point, later in the series._ The method copies a sequence of array elements _within the array_ to the _"paste position"_ starting at `target`. The elements that should be copied are taken from the `[start, end)` range.

Here's the signature of the `copyWithin` method. The `target` _"paste position"_ **is required**. The `start` index where to take elements from defaults to `0`. The `end` position defaults to the length of the array.

```js
Array.prototype.copyWithin(target, start = 0, end = this.length)
```

Let's start with a simple example. Consider the `items` array in the snippet below.

```js
var items = [1, 2, 3, ,,,,,,,]
// <- [1, 2, 3, undefined x 7]
```

The method below takes the `items` array and determines that it'll start _"pasting"_ items in the **sixth position**. It further determines that the items to be copied will be taken starting in the **second position** _(zero-based)_, until the **third position** _(also zero-based)_.

```js
items.copyWithin(6, 1, 3)
// <- [1, 2, 3, <mark>undefined × 3</mark>, 2, 3, <mark>undefined × 2</mark>]
```

> Reasoning about this method can be pretty hard. _Let's break it down._

If we consider that the items to be copied were taken from the `[start, end)` range, then we can express that using the `.slice` operation. These are the items that were _"pasted"_ at the `target` position. We can use `.slice` to _"copy"_ them.

```js
items.slice(1, 3)
// <- [<mark>2, 3</mark>]
```

We could then consider the "pasting" part of the operation as an advanced usage of [`.splice`][9] -- one of those lovely methods that can do just about anything. The method below does just that, and then returns `items`, because `.splice` returns the items that were spliced from an Array, and in our case this is no good. Note that we also had to use the [spread operator][4] so that elements are inserted individually through `.splice`, and not as an array.

```js
function copyWithin (items, target, start = 0, end = items.length) {
  items.splice(target, end - start, <mark>...</mark>items.slice(start, end))
  return items
}
```

Our example would still work the same with this method.

```js
copyWithin([1, 2, 3, ,,,,,,,], 6, 1, 3)
// <- [1, 2, 3, undefined × 3, 2, 3, undefined × 2]
```

The `copyWithin` method accepts negative `start` indices, negative `end` indices, and negative `target` indices. Let's try something using that.

```js
[1, 2, 3, ,,,,,,,].copyWithin(-3)
// <- [1, 2, 3, undefined x 4, 1, 2, 3]
copyWithin([1, 2, 3, ,,,,,,,], -3)
// <- [1, 2, 3, undefined x 4, 1, 2, 3, <mark>undefined x 7</mark>]
```

Turns out, that thought exercise was useful for understanding `Array.prototype.copyWithin`, but it wasn't actually correct. Why are we seeing `undefined x 7` at the end? Why the discrepancy? The problem is that we are seeing the array holes at the end of `items` when we do `...items.slice(start, end)`.

```js
[1, 2, 3, ,,,,,,,]
// <- [1, 2, 3, undefined x 7]
[1, 2, 3, ,,,,,,,].slice(0, 10)
// <- [1, 2, 3, undefined x 7]
console.log(<mark>...[1, 2, 3, ,,,,,,,].slice(0, 10)</mark>)
// <- 1, 2, 3, undefined, undefined, undefined, undefined, undefined, undefined, undefined
```

Thus, we _end up splicing the holes_ onto `items`, while the original solution is not. We could get rid of the holes using `.filter`, which conveniently discards array holes.

```js

[1, 2, 3, ,,,,,,,].slice(0, 10)
// <- [1, 2, 3, undefined x 7]
[1, 2, 3, ,,,,,,,].slice(0, 10)<mark>.filter(el => true)</mark>
// <- [1, 2, 3]
```

With that, we can update our `copyWithin` method. We'll stop using `end - start` as the splice position and instead use the amount of `replacements` that we have, as those numbers may be different now that we're discarding array holes.

```js
function copyWithin (items, target, start = 0, end = items.length) {
  var replacements = items.slice(start, end)<mark>.filter(el => true)</mark>
  items.splice(target, <mark>replacements.length</mark>, ...replacements)
  return items
}
```

The case were we previously added extra holes now works as expected. Woo!

```js
[1, 2, 3, ,,,,,,,].copyWithin(-3)
// <- [1, 2, 3, undefined x 4, 1, 2, 3]
copyWithin([1, 2, 3, ,,,,,,,], -3)
// <- [1, 2, 3, undefined x 4, 1, 2, 3]
```

Furthermore, our polyfill seems to work correctly *across the board* now. I wouldn't rely on it for anything other than educational purposes, though.

```js
[1, 2, 3, ,,,,,,,].copyWithin(-3, 1)
// <- [1, 2, 3, undefined x 4, 2, 3, undefined x 1]
copyWithin([1, 2, 3, ,,,,,,,], -3, 1)
// <- [1, 2, 3, undefined x 4, 2, 3, undefined x 1]
[1, 2, 3, ,,,,,,,].copyWithin(-6, -8)
// <- [1, 2, 3, undefined x 1, 3, undefined x 5]
copyWithin([1, 2, 3, ,,,,,,,], -6, -8)
// <- [1, 2, 3, undefined x 1, 3, undefined x 5]
[1, 2, 3, ,,,,,,,].copyWithin(-3, 1, 2)
// <- [1, 2, 3, undefined x 4, 2, undefined x 2]
copyWithin([1, 2, 3, ,,,,,,,], -3, 1, 2)
// <- [1, 2, 3, undefined x 4, 2, undefined x 2]
```

It's decidedly better to just use the actual implementation, but at least now we have **a better idea of how the hell it works!**

# `Array.prototype.fill`

Convenient utility method to fill all places in an `Array` with the provided `value`. Note that array holes will be filled as well.

```js
['a', 'b', 'c'].fill(0)
// <- [0, 0, 0]
new Array(3).fill(0)
// <- [0, 0, 0]
```

You could also determine a start index and an end index in the second and third parameters respectively.

```js
['a', 'b', 'c',,,].fill(0, 2)
// <- ['a', 'b', 0, 0, 0]
new Array(5).fill(0, 0, 3)
// <- [0, 0, 0, <mark>undefined x 2</mark>]
```

The provided value can be arbitrary, and not necessarily a number or even a primitive type.

```js
new Array(3).fill({})
// <- [{}, {}, {}]
```

Unfortunately, you can't fill arrays using a mapping method that takes an `index` parameter or anything like that.

```js
new Array(3).fill(function foo () {})
// <- [function foo () {}, function foo () {}, function foo () {}]
```

_Moving along..._

# `Array.prototype.find`

Ah. One of those methods that JavaScript desperately wanted but didn't get in ES5. The `.find` method returns the _first_ `item` that matches `callback(item, i, array)` for an `array` Array. You can also optionally pass in a `context` binding for `this`. You can think of it as an equivalent of [`.some`][10] that returns the matching element _(or `undefined`)_ instead of merely `true` or `false`.

```js
[1, 2, 3, 4, 5].find(item => item > 2)
// <- 3
[1, 2, 3, 4, 5].find((item, i) => i === 3)
// <- 4
[1, 2, 3, 4, 5].find(item => item === Infinity)
// <- undefined
```

There's really not much else to say about this method. It's just that simple! We did want this method a lot, as evidenced in libraries like Lodash and Underscore. Speaking of those libraries... -- `.findIndex` was also born there.

# `Array.prototype.findIndex`

This method is also an equivalent of [`.some`][10] and [`.find`](#arrayprototypefind). Instead of returning `true`, like `.some`; or `item`, like `.find`; this method returns the `index` position so that `array[index] === item`. If none of the elements in the collection match the `callback(item, i, array)` criteria, the return value is `-1`.

```js
[1, 2, 3, 4, 5].find(item => item > 2)
// <- 2
[1, 2, 3, 4, 5].find((item, i) => i === 3)
// <- 3
[1, 2, 3, 4, 5].find(item => item === Infinity)
// <- -1
```

Again, quite straightforward.

# `Array.prototype.keys`

Returns an iterator that yields a sequence holding the keys for the array. The returned value is an iterator, meaning you can use it with all of the usual suspects like [`for..of`][2], the [spread operator][4], or by hand by manually calling `.next()`.

```js
[1, 2, 3].keys()
// <- ArrayIterator {}
```

Here's an example using `for..of`.

```js
for (let key of [1, 2, 3].keys()) {
  console.log(key)
  // <- 0
  // <- 1
  // <- 2
}
```

Unlike `Object.keys` and most methods that iterate over arrays, this sequence doesn't ignore holes.

```js
[<mark>...</mark>new Array(3)<mark>.keys()</mark>]
// <- [0, 1, 2]
Object.keys(new Array(3))
// <- []
```

Now onto values.

# `Array.prototype.values`

Same thing as `.keys()`, but the returned iterator is a sequence of values instead of indices. In practice, you'll probably just iterate over the array itself, but sometimes getting an iterator can come in handy.

```js
[1, 2, 3].values()
// <- ArrayIterator {}
```

Then you can use `for..of` or any other methods like a spread operator to pull out the sequence. The example below shows how using the spread operator on an array's `.values()` doesn't really make a lot of sense _-- you already had that collection to begin with!_

```js
[<mark>...</mark>[1, 2, 3]<mark>.values()</mark>]
// <- [1, 2, 3]
```

Do note that the returned array in the example above is _a different array_ and not a reference to the original one.

> Time for `.entries`.

# `Array.prototype.entries`

Similar to both preceding methods, but this one returns an iterator with a sequence of key-value pairs.

```js
['a', 'b', 'c'].entries()
// <- ArrayIterator {}
```

Each entry contains a two dimensional array element with the key and the value for an item in the array.

```js
[...['a', 'b', 'c'].entries()]
// <- [[0, 'a'], [1, 'b'], [2, 'c']]
```

Great, last one to go!

# `Array.prototype[Symbol.iterator]`

This is <del>basically</del> <ins>**exactly**</ins> the same as the [`.values`](#arrayprototypevalues) method. The example below combines a spread operator, an array, and `Symbol.iterator` to iterate over its values.

```js
[<mark>...</mark>['a', 'b', 'c']<mark>[Symbol.iterator]()</mark>]
// <- ['a', 'b', 'c']
```

Of course, you should probably just omit the spread operator and the `[Symbol.iterator]` part in most use cases. Same time tomorrow? We'll cover changes to the `Object` API.

[1]: /articles/fun-with-native-arrays "Fun with Native Arrays on Pony Foo"
[2]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[3]: /articles/tagged/es6-in-depth "Articles tagged es6-in-depth on Pony Foo"
[4]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[5]: http://jquery.com "jQuery: Do less, bloat more"
[6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#Syntax "Using the new operator on an Array on MDN"
[7]: http://www.2ality.com/2013/07/array-iteration-holes.html "Array iteration and holes in JavaScript on 2ality.com"
[8]: https://developer.mozilla.org/en/docs/Web/JavaScript/Typed_arrays "JavaScript typed arrays on MDN"
[9]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/splice "Array.prototype.splice on MDN"
[10]: http://ponyfoo.com/articles/fun-with-native-arrays#asserting-with-some-and-every "Asserting with .some and .every on Pony Foo"
