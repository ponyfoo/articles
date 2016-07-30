# Upcoming `Object` Changes

Objects didn't get as many new methods in ES6 as [arrays did][1]. In the case of objects, we get four new static methods, and no new instance methods or properties.

- [`Object.assign`](#objectassign)
- [`Object.is`](#objectis)
- [`Object.getOwnPropertySymbols`](#objectgetownpropertysymbols)
- [`Object.setPrototypeOf`](#objectsetprototypeof)

And just like arrays, objects are slated to get a few more static methods in ES2016 _(ES7)_. We're not going to cover these today.

- `Object.observe`
- `Object.unobserve`

_Shall we?_

# `Object.assign`

This is another example of the kind of helper method that has been beaten to death by libraries like Underscore and Lodash. I even wrote my own implementation that's [around 20 lines of code][2]. You can use `Object.assign` to recursively overwrite properties on an object with properties from other objects. The first argument passed to `Object.assign`, `target`, will be _used as the return value as well._ Subsequent values are _"applied"_ onto that object.

```js
Object.assign(<mark>{}</mark>, { a: 1 })
// <- <mark>{ a: 1 }</mark>
```

If you already had a property, it's overwritten.

```js
Object.assign(<mark>{ a: 1 }</mark>, { a: 2 })
// <- <mark>{ a: 2 }</mark>
```

Properties that aren't present in the object being assigned are left untouched.

```js
Object.assign(<mark>{ a: 1, b: 2 }</mark>, { a: 3 })
// <- <mark>{ a: 3, b: 2 }</mark>
```

You can assign as many objects as you want. You can think of `Object.assign(a, b, c)` as the equivalent of doing `Object.assign(Object.assign(a, b), c)`, if that makes it easier for you to reason about it. I like to reason about it as a [reduce][3] operation.

```js
Object.assign(<mark>{ a: 1, b: 2 }</mark>, { a: 3 }, { c: 4 })
// <- <mark>{ a: 3, b: 2, c: 4 }</mark>
```

Note that only enumerable own properties are copied over _-- think `Object.keys` plus `Object.getOwnPropertySymbols`._ The example below shows an `invisible` property that didn't get copied over. Properties from the prototype chain aren't taken into account either.

```js
var a = { b: 'c' }
Object.defineProperty(a, 'invisible', { <mark>enumerable: false</mark>, value: 'boo! ahhh!' })
Object.assign(<mark>{}</mark>, a)
// <- <mark>{ b: 'c' }</mark>
```

You can use this API against arrays as well.

```js
Object.assign(<mark>[1, 2, 3]</mark>, [4, 5])
// <- <mark>[4, 5, 3]</mark>
```

Properties using symbols as their keys are also copied over.

```js
Object.assign(<mark>{ a: 'b' }</mark>, { [Symbol('c')]: 'd' })
// <- <mark>{ a: 'b', Symbol(c): 'd' }</mark>
```

As long as they're enumerable and found directly on the object, that is.

```js
var a = {}
Object.defineProperty(a, <mark>Symbol('b')</mark>, { <mark>enumerable: false</mark>, value: 'c' })
Object.assign(<mark>{}</mark>, a)
// <- <mark>{}</mark>
```

There's a problem with `Object.assign`. It doesn't allow you to control how deep you want to go. You may be hoping for a way to do the following while preserving the `target.a.d` property, but `Object.assign` replaces `target.a` entirely with `source.a`.

```js
var target = { a: { b: 'c', d: 'e' } }
var source = { a: { b: 'ahh!' } }
Object.assign(target, source)
// <- <mark>{ a: { b: 'ahh!' } }</mark>
```

Most implementations in the wild work differently, at least giving you _the option_ to make a _"deep assign"_. Take [`assignment`][10] for instance. If it finds an object reference in `target` for a given property, it has two options.

- If the value in `source[key]` is an object, it goes recursive with an [`assignment(target[key], source[key])`][11] call
- If the value is not an object, it just replaces it: `target[key] = source[key]`

This means that the last example we saw would work differently with [`assignment`][10] than how it did with `Object.assign`, which only allows for shallow extensions.

```js
var target = { a: { b: 'c', d: 'e' } }
var source = { a: { b: 'ahh!' } }
assignment(target, source)
// <- { a: { b: 'ahh!', <mark>d: 'e'</mark> } }
```

The [`assignment`][10] approach is **usually preferred** when it comes to the most common use case of this type of method: providing sensible defaults that can be overwritten by the user. Consider the following example. It uses the well-known pattern of providing your _"assign"_ method with an empty object, that's then filled with default values, and then poured user preferences for good measure. Note that it doesn't change the defaults object directly because those are supposed to stay the same across invocations.

```js
function markdownEditor (user) {
  var defaults = {
    height: 400,
    markdown: {
      githubFlavored: true,
      tables: false
    }
  }
  var options = <mark>Object.assign({}, defaults, user)</mark>
  console.log(options)
}
```

The problem with `Object.assign` is that if the `markdownEditor` consumer wants to change `markdown.tables` to `true`, all of the other defaults in `markdown` will be lost!

```js
markdownEditor({ markdown: { tables: true } })
// <- {
//      height: 400,
//      markdown: {
//        tables: true
//      }
//    }
```

From both the library author's perspective and the library's user perspective, this is just unacceptable and weird. If we were to use `assignment` we wouldn't be having those issues, because `assignment` is built with this particular use case in mind. Libraries like Lodash usually provide [many different flavors][4] of this method.

Note that when it comes to nested arrays, **replacement** _probably is_ the behavior you want most of the time. Given defaults like `{ extensions: ['css', 'js', 'html'] }`, the following would be quite weird.

```js
markdownEditor({ extensions: ['js'] })
// <- { extensions: ['js', <mark>'js', 'html'</mark>] }
```

For that reason, [`assignment`][2] replaces arrays entirely, just like `Object.assign` would. This difference **doesn't** make `Object.assign` useless, but it's still necessary to know about the difference between shallow and deep assignment.

# `Object.is`

This method is pretty much a programmatic way to use the `===` operator. You pass in two arguments and it tells you whether they're the same reference or the same primitive value.

```js
Object.is('foo', 'foo')
// <- true
Object.is({}, {})
// <- false
```

There are **two important differences**, however. First off, `-0` and `+0` are considered unequal by this method, even though `===` returns `true`.

```js
-0 === +0
// <- true
Object.is(-0, +0)
// <- <mark>false</mark>
```

The other difference is when it comes to `NaN`. The `Object.is` method treats `NaN` as equal to `NaN`. This is a behavior we've [already observed in maps and sets][6], which also treats `NaN` as being the same value as `NaN`.

```js
NaN === NaN
// <- false
Object.is(NaN, NaN)
// <- <mark>true</mark>
```

While this may be convenient in some cases, I'd probably go for the more explicit [`Number.isNaN`][5] most of the time.

# `Object.getOwnPropertySymbols`

This method returns all own property symbols found on an object.

```js
var a = {
  [Symbol('b')]: 'c',
  [Symbol('d')]: 'e',
  'f': 'g',
  'h': 'i'
}
Object.getOwnPropertySymbols(a)
// <- <mark>[Symbol(b), Symbol(d)]</mark>
```

We've already covered `Object.getOwnPropertySymbols` in depth in the [symbols dossier][7]. If I were you, I'd read it!

# `Object.setPrototypeOf`

Again, something we've covered earlier in the series. One of the articles about [proxy traps][8] covers this method tangentially. You can use `Object.setPrototypeOf` to change the prototype of an object.

It is, in fact, the equivalent of setting [`__proto__`][9] on runtimes that have that property.

[1]: /articles/es6-array-extensions-in-depth "ES6 Array Extensions in Depth on Pony Foo"
[2]: https://github.com/bevacqua/assignment/blob/master/assignment.js "bevacqua/assignment on GitHub"
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce "Array.prototype.reduce on MDN"
[4]: https://lodash.com/docs#defaultsDeep "See .assign, .defaults, and .defaultsDeep on their documentation"
[5]: /articles/es6-number-improvements-in-depth#numberisnan "ES6 Number Improvements in Depth on Pony Foo"
[6]: /articles/es6-maps-in-depth "ES6 Maps in Depth on Pony Foo"
[7]: /articles/es6-symbols-in-depth "ES6 Symbols in Depth on Pony Foo"
[8]: /articles/more-es6-proxy-traps-in-depth#setprototypeof ".setPrototypeOf is in the More ES6 Proxy Traps in Depth article on Pony Foo"
[9]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto "Object.prototype.__proto__ on MDN"
[10]: https://github.com/bevacqua/assignment/blob/master/assignment.js#L3 "assignment on GitHub"
[11]: https://github.com/bevacqua/assignment/blob/master/assignment.js#L13 "Recursive assignment on GitHub"
