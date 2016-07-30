# ES6 WeakMaps

You can think of `WeakMap` as a subset of [`Map`][11]. There are a few limitations on `WeakMap` that we didn't find in `Map`. The biggest limitation is that `WeakMap` is not iterable, as opposed to `Map` -- that means there is no [_iterable_][1] protocol, no `.entries()`, no `.keys()`, no `.values()`, no `.forEach()` and no `.clear()`.

Another _"limitation"_ found in `WeakMap` as opposed to `Map` is that every `key` must be an object, and **value types are not admitted as keys**. Note that `Symbol` is a value type as well, and they're not allowed either.

```js
var map = new WeakMap()
map.set(1, 2)
// TypeError: 1 is not an object!
map.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
```

> This is more of a feature than an issue, though, as it enables map keys to be garbage collected when they're only being referenced as `WeakMap` keys. Usually you want this behavior when storing metadata related to something like a DOM node, and now you can keep that metadata in a `WeakMap`. If you want all of those you could always [use a regular `Map` as we explored earlier][11].

You are still able to pass an iterable to populate a `WeakMap` through its constructor.

```js
var map = new WeakMap([[new Date(), 'foo'], [() => 'bar', 'baz']])
```

Just like with `Map`, you can use `.has`, `.get`, and `.delete` too.

```js
var date = new Date()
var map = new WeakMap([[date, 'foo'], [() => 'bar', 'baz']])
console.log(map.has(date))
// <- true
console.log(map.get(date))
// <- 'foo'
map.delete(date)
console.log(map.has(date))
// <- false
```

## Is This a Strictly Worse Map?

I know! You must be wondering -- why the hell would I use `WeakMap` when it has so many limitations when compared to `Map`?

The difference that may make `WeakMap` worth it is in its name. `WeakMap` holds references to its keys _weakly_, meaning that if there are no other references to one of its keys, the object is subject to **garbage collection**.

Use cases for `WeakMap` generally revolve around the need to specify metadata or extend an object while still being able to garbage collect it if nobody else cares about it. A perfect example might be the underlying implementation for [`process.on('unhandledRejection')`][6] which [uses a `WeakMap`][7] to keep track of promises that were rejected but _no error handlers dealt with the rejection_ within a tick.

Keeping data about DOM elements that should be released from memory when they're no longer of interest is another very important use case, and in this regard using `WeakMap` is probably an even better solution to the DOM-related [API caching solution][8] we wrote about earlier using `Map`.

In so many words then, **no**. `WeakMap` is not strictly worse than `Map` _-- they just cater to different use cases._

## ES6 Sets

Sets are _yet another_ collection type in ES6. Sets are _very_ similar to `Map`. To wit:

- `Set` is also [_iterable_][1]
- `Set` constructor also accepts an _iterable_
- `Set` also has a `.size` property
- Keys can also be arbitrary values
- Keys must be unique
- `NaN` equals `NaN` when it comes to `Set` too
- All of `.keys`, `.values`, `.entries`, `.forEach`, <del>`.get`</del>, <del>`.set`</del>, `.has`, `.delete`, and `.clear`

However, there's a few differences as well!

- Sets only have `values`
- No `set.get` -- but **why** would you want `get(value) => value`?
- Having `set.set` would be weird, so we have `set.add` instead
- `set[Symbol.iterator] !== set.entries`
- `set[Symbol.iterator] === set.values`
- `set.keys === set.values`
- `set.entries()` returns an iterator on a sequence of items like `[value, value]` 

In the example below you can note how it takes an iterable with duplicate values, it can be spread over an `Array` using the [spread operator][9], and how the duplicate value _has been ignored_.

```js
var set = new Set([1, 2, 3, 4, <mark>4</mark>])
console.log([...set])
// <- <mark>[1, 2, 3, 4]</mark>
```

Sets may be a great alternative to work with DOM elements. The following piece of code creates a `Set` with all the `<div>` elements on a page and then prints how many it found. Then, we query the DOM _again_ and call `set.add` again for every DOM element. Since they're all already in the `set`, the `.size` property won't change, meaning the `set` remains the same.

```js
function divs () {
  return <mark>[...document.querySelectorAll('div')]</mark>
}
var set = new Set(<mark>divs()</mark>)
console.log(set.size)
// <- 56
<mark>divs().forEach(div => set.add(div))</mark>
console.log(set.size)
// <- 56
// <- look at that, no duplicates!
```

# ES6 WeakSets

Much like with `WeakMap` and `Map`, `WeakSet` is **`Set` plus weakness** minus the _iterability_ _-- I just made that term up, didn't I?_

That means you can't iterate over `WeakSet`. Its values must be **unique object references**. If nothing else is referencing a `value` found in a `WeakSet`, it'll be subject to garbage collection.

Much like in `WeakMap`, you can only `.add`, `.has`, and `.delete` values from a `WeakSet`. And just like in `Set`, there's no `.get`.

```js
var set = new WeakSet()
set.add({})
set.add(new Date())
```

As we know, we can't use primitive values.

```js
var set = new WeakSet()
set.add(Symbol())
// TypeError: invalid value used in weak set
```

Just like with `WeakMap`, passing iterators to the constructor is still allowed even though a `WeakSet` instance is not iterable itself.

```js
var set = new WeakSet([new Date(), {}, () => {}, [1]])
```

Use cases for `WeakSet` vary, and here's one from [a thread on _es-discuss_][10] -- the mailing list for the ECMAScript-262 specification of JavaScript.

```js
const foos = new WeakSet()
class Foo {
  constructor() {
    <mark>foos.add(this)</mark>
  }
  method () {
    if (!<mark>foos.has(this)</mark>) {
      throw new TypeError('Foo.prototype.method called on incompatible object!')
    }
  }
}
```

As a general rule of thumb, you can also try and figure out whether a `WeakSet` will do when you're considering to use a `WeakMap` as some use cases may overlap. Particularly, if all you need to check for is whether a reference value is in the `WeakSet` or not.

> Next week we'll be having [`Proxy`][2] for brunch :)

[1]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[2]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Proxy "Proxy Objects in ES6 on MDN"
[3]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
[4]: /articles/es6-template-strings-in-depth "ES6 Template Literals in Depth on Pony Foo"
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in "for..in on MDN"
[6]: https://iojs.org/api/process.html#process_event_unhandledrejection "Node.js Documentation for 'unhandledRejection' process event"
[7]: https://github.com/petkaantonov/io.js/commit/f46874357ee7b909ae54304c6791f2a4baddf613#diff-6ff379484cbabad48301d485db111c08R269 "node: implement unhandled rejection tracking"
[8]: /articles/es6-maps-in-depth#hash-maps-and-the-dom "Hash-Maps and the DOM"
[9]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[10]: https://esdiscuss.org/topic/actual-weakset-use-cases#content-1 "Actual WeakSet Use Cases on ES Discuss"
[11]: /articles/es6-maps-in-depth "ES6 Maps in Depth on Pony Foo"
