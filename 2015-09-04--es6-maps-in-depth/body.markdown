# Before ES6, There Were Hash-Maps

A very common *ab*use case of JavaScript objects is hash-maps, where we map string keys to arbitrary values. For example, one might use an object to map `npm` package names to their metadata, like so:

```js
var registry = {}
function add (name, meta) {
  registry[name] = meta
}
function get (name) {
  return registry[name]
}
add('contra', { description: 'Asynchronous flow control' })
add('dragula', { description: 'Drag and drop' })
add('woofmark', { description: 'Markdown and WYSIWYG editor' })
```

There's several issues with this approach, to wit:

- **Security issues** where user-provided keys like `__proto__`, `toString`, or anything in `Object.prototype` break expectations and make interaction with these kinds of _hash-map_ data structures more cumbersome
- Iteration over list items is verbose with `Object.keys(registry).forEach` or implementing the [_iterable_ protocol][1] on the `registry`
- Keys are limited to strings, making it hard to create hash-maps where you'd like to index values by DOM elements or other non-string references

The first problem could be fixed using a prefix, and being careful to always get or set values in the hash-map through methods. It would be even better to use [ES6 proxies][2], but we _won't be covering those until tomorrow!_

```js
var registry = {}
function add (name, meta) {
  registry[<mark>'map:' + </mark>name] = meta
}
function get (name) {
  return registry[<mark>'map:' + </mark>name]
}
add('contra', { description: 'Asynchronous flow control' })
add('dragula', { description: 'Drag and drop' })
add('woofmark', { description: 'Markdown and WYSIWYG editor' })
```

Luckily for us, though, _ES6 maps_ provide us with an even better solution to the key-naming security issue. At the same time they facilitate collection behaviors out the box that may also come in handy. Let's plunge into their practical usage and inner workings.

# ES6 Maps

Map is a key/value data structure in ES6. It provides a better data structure to be used for hash-maps. Here's how what we had earlier looks like with ES6 maps.

```js
var map = <mark>new Map()</mark>
<mark>map.set</mark>('contra', { description: 'Asynchronous flow control' })
map.set('dragula', { description: 'Drag and drop' })
map.set('woofmark', { description: 'Markdown and WYSIWYG editor' })
```

One of the important differences is also that you're able to use anything for the keys. You're not just limited to primitive values like symbols, numbers, or strings, but you can even use functions, objects and dates -- too. Keys won't be casted to strings like with regular objects, either.

```js
var map = new Map()
map.set(new Date(), function today () {})
map.set(() => 'key', { pony: 'foo' })
map.set(Symbol('items'), [1, 2])
```

You can also provide `Map` objects with any object that follows the [_iterable_ protocol][1] and produces a collection such as `[['key', 'value'], ['key', 'value']]`.

```js
var map = new Map([
  [new Date(), function today () {}],
  [() => 'key', { pony: 'foo' }],
  [Symbol('items'), [1, 2]]
])
```

The above would be effectively the same as the following. Note how we're using destructuring in the parameters of `items.forEach` to _effortlessly_ pull the `key` and `value` out of the two-dimensional `item`.

```js
var items = [
  [new Date(), function today () {}],
  [() => 'key', { pony: 'foo' }],
  [Symbol('items'), [1, 2]]
]
var map = new Map()
items.forEach((<mark>[key, value]</mark>) => map.set(key, value))
```

Of course, it's kind of silly to go through the trouble of adding items one by one when you can just feed an iterable to your `Map`. Speaking of iterables -- `Map` adheres to the [_iterable_][1] protocol. It's very easy to pull a key-value pair collection much like the ones you can feed to the `Map` constructor.

Naturally, we can use the spread operator to this effect.

```js
var map = new Map()
map.set('p', 'o')
map.set('n', 'y')
map.set('f', 'o')
map.set('o', '!')
console.log([...map])
// <- [['p', 'o'], ['n', 'y'], ['f', 'o'], ['o', '!']]
```

You could also use a `for..of` loop, and we could combine that with [destructuring][3] to make it seriously terse. Also, remember [template literals][4]?

```js
var map = new Map()
map.set('p', 'o')
map.set('n', 'y')
map.set('f', 'o')
map.set('o', '!')
for (let <mark>[key, value]</mark> of map) {
  console.log(<mark>`${key}: ${value}`</mark>)
  // <- 'p: o'
  // <- 'n: y'
  // <- 'f: o'
  // <- 'o: !'
}
```

Even though maps have a programmatic API to add items, keys are unique, just like with hash-maps. Setting a key over and over again will only overwrite its value.

```js
var map = new Map()
map.set('a', 'a')
map.set('a', 'b')
map.set('a', 'c')
console.log([...map])
// <- [['a', 'c']]
```

In ES6 `Map`, `NaN` becomes a "corner-case" that gets **treated as a value that's equal to itself** even though the following expression actually evaluates to `true` -- `NaN !== NaN`.

```js
console.log(NaN === NaN)
// <- false
var map = new Map()
map.set(NaN, 'foo')
map.set(NaN, 'bar')
console.log([...map])
// <- [[NaN, 'bar']]
```

## Hash-Maps and the DOM

In ES5, whenever we had a DOM element we wanted to associate with an API object for some library, we had to follow a verbose and slow pattern like the one below. The following piece of code just returns an API object with a bunch of methods for a given DOM element, allowing us to put and remove DOM elements from the cache, and also allowing us to retrieve the API object for a DOM element -- if one already exists.

```js
var cache = []
function put (el, api) {
  cache.push({ el: el, api: api })
}
function find (el) {
  for (i = 0; i < cache.length; i++) {
    if (cache[i].el === el) {
      return cache[i].api
    }
  }
}
function destroy (el) {
  for (i = 0; i < cache.length; i++) {
    if (cache[i].el === el) {
      cache.splice(i, 1)
      return
    }
  }
}
function thing (el) {
  var api = find(el)
  if (api) {
    return api
  }
  api = {
    method: method,
    method2: method2,
    method3: method3,
    destroy: destroy.bind(null, el)
  }
  put(el, api)
  return api
}
```

One of the coolest aspects of `Map`, _as I've previously mentioned_, is the ability to index by DOM elements. The fact that `Map` also has collection manipulation abilities also greatly simplifies things.

```js
var cache = new Map()
function put (el, api) {
  <mark>cache.set(el, api)</mark>
}
function find (el) {
  return <mark>cache.get(el)</mark>
}
function destroy (el) {
  <mark>cache.delete(el)</mark>
}
function thing (el) {
  var api = find(el)
  if (api) {
    return api
  }
  api = {
    method: method,
    method2: method2,
    method3: method3,
    destroy: destroy.bind(null, el)
  }
  put(el, api)
  return api
}
```

The fact that these methods have now become one liners means we can just inline them, as readability is no longer an issue. We just went from _~30 LOC_ to **half that amount**. Needless to say, at some point in the future this will also perform _much_ faster than the haystack alternative.

```js
var cache = new Map()
function thing (el) {
  var api = <mark>cache.get(el)</mark>
  if (api) {
    return api
  }
  api = {
    method: method,
    method2: method2,
    method3: method3,
    destroy: () => <mark>cache.delete(el)</mark>
  }
  <mark>cache.set(el, api)</mark>
  return api
}
```

The simplicity of `Map` is amazing. If you ask me, we desperately needed this feature in JavaScript. Being to index a collection by arbitrary objects is **super important**.

> What else can we do with `Map`?

## Collection Methods in `Map`

Maps make it very easy to probe the collection and figure out whether a `key` is defined in the `Map`. As we noted earlier, `NaN` equals `NaN` as far as `Map` is concerned. However, `Symbol` values are always different, so you'll have to use them by value!

```js
var map = new Map([[NaN, 1], [Symbol(), 2], ['foo', 'bar']])
console.log(map.has(NaN))
// <- true
console.log(map.has(Symbol()))
// <- <mark>false</mark>
console.log(map.has('foo'))
// <- true
console.log(map.has('bar'))
// <- false
```

As long as you keep a `Symbol` reference around, you'll be okay. _Keep your references close, and your `Symbol`s closer?_

```js
<mark>var sym = Symbol()</mark>
var map = new Map([[NaN, 1], [sym, 2], ['foo', 'bar']])
console.log(map.has(sym))
// <- <mark>true</mark>
```

Also, remember the **no key-casting** thing? _Beware!_ We are so used to objects casting keys to strings that this may bite you if you're not careful.

```js
var map = new Map([[1, 'a']])
console.log(map.has(1))
// <- true
console.log(map.has('1'))
// <- <mark>false</mark>
```

You can also clear a `Map` entirely of entries without losing a reference to it. This can be very handy sometimes.

```js
var map = new Map([[1, 2], [3, 4], [5, 6]])
map.clear()
console.log(map.has(1))
// <- false
console.log([...map])
// <- []
```

When you use `Map` as an iterable, you are actually looping over its `.entries()`. That means that you don't need to **explicitly** iterate over `.entries()`. It'll be done on your behalf anyways. You do remember [`Symbol.iterator`][1], right?

```js
console.log(map[<mark>Symbol.iterator</mark>] === map.entries)
// <- true
```

Just like `.entries()`, `Map` has two other iterators you can leverage. These are `.keys()` and `.values()`. I'm sure you guessed what sequences of values they yield, but here's a code snippet anyways.

```js
var map = new Map([[1, 2], [3, 4], [5, 6]])
console.log([...map.keys()])
// <- [1, 3, 5]
console.log([...map.values()])
// <- [2, 4, 6]
```

Maps also come with a _read-only_ `.size` property that behaves sort of like `Array.prototype.length` -- at any point in time it gives you the current amount of entries in the map.

```js
var map = new Map([[1, 2], [3, 4], [5, 6]])
console.log(map.size)
// <- 3
map.delete(3)
console.log(map.size)
// <- 2
map.clear()
console.log(map.size)
// <- 0
```

One more aspect of `Map` that's worth mentioning is that their entries are always iterated in **insertion order**. This is in contrast with `Object.keys` loops which follow [an arbitrary order][5].

> The [`for..in`][5] statement iterates over the enumerable properties of an object, in arbitrary order.

Maps also have a `.forEach` method that's identical in _behavior_ to that in ES5 `Array` objects. Once again, keys do not get casted into strings here.

```js
var map = new Map([[NaN, 1], [Symbol(), 2], ['foo', 'bar']])
map.forEach(<mark>(value, key)</mark> => console.log(key, value))
// <- NaN 1
// <- Symbol() 2
// <- 'foo' 'bar'
```

> Get up early tomorrow morning, we'll be having [`WeakMap`, `Set`, and `WeakSet`][6] for breakfast :)

[1]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[2]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Proxy "Proxy Objects in ES6 on MDN"
[3]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
[4]: /articles/es6-template-strings-in-depth "ES6 Template Literals in Depth on Pony Foo"
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in "for..in on MDN"
[6]: http://ponyfoo.com/articles/es6-weakmaps-sets-and-weaksets-in-depth "ES6 WeakMaps, Sets, and WeakSets in Depth on Pony Foo"
