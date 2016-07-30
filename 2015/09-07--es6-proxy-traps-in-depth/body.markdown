## Proxy Trap Handlers

An interesting aspect of proxies is how you can use them to intercept just about any interaction with a `target` object -- not just `get` or `set` operations. Below are some of the traps you can set up, here's a summary.

- [`has`](#has) -- traps `in` operator
- [`deleteProperty`](#deleteproperty) -- traps `delete` operator
- [`defineProperty`](#defineproperty) -- traps `Object.defineProperty` and declarative alternatives
- [`enumerate`](#enumerate) -- traps `for..in` loops
- [`ownKeys`](#ownkeys) -- traps `Object.keys` and related methods
- [`apply`](#apply) -- traps _function calls_

We'll bypass `get` and `set`, because we [already covered those two][29] yesterday; and there's a few more traps that aren't listed here that will make it into an article published tomorrow. _Stay tuned!_

### `has`

You can use [`handler.has`][9] to _"hide"_ any property you want. It's a trap for the `in` operator. In the [`set`][18] trap example we prevented changes and even access to _`_`-prefixed_ properties, but unwanted accessors could still ping our proxy to figure out whether these properties are actually there or not. _Like [Goldilocks][19]_, we have three options here.

- We can let `key in proxy` _fall through_ to `key in target`
- We can `return false` _(or `true`)_ -- even though `key` **may or may not** actually be there
- We can `throw` an error and deem the question **invalid** in the first place

The last option is quite harsh, and I imagine it being indeed a valid choice in some situations -- but you would be acknowledging that the property _(or "property space")_ is, in fact, _protected_. It's often best to just smoothly indicate that the property is not `in` the object. Usually, a fall-through case where you just return the result of the `key in target` expression is a good default case to have.

In our example, we probably want to `return false` for properties in the _`_`-prefixed "property space"_ and the default of `key in target` for all other properties. This will keep our inaccessible properties well hidden from unwanted visitors.

```js
var handler = {
  get (target, key) {
    invariant(key, 'get')
    return target[key]
  },
  set (target, key, value) {
    invariant(key, 'set')
    return true
  },
  <mark>has (target, key) {</mark>
    if (key[0] === '_') {
      <mark>return false</mark>
    }
    <mark>return key in target</mark>
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}
```

Note how accessing properties through the proxy will now return `false` whenever accessing one of our private properties, with the consumer being none the wiser -- completely unaware that we've intentionally hid the property from them.

```js
var target = { _prop: 'foo', pony: 'foo' }
var proxy = new Proxy(target, handler)
console.log('pony' in proxy)
// <- true
console.log('_prop' in proxy)
// <- false
console.log('_prop' in target)
// <- true
```

Sure, we could've thrown an exception instead. That'd be useful in situations where attempts to access properties in the private space is seen more of **as a mistake that results in broken modularity** than as a _security concern_ in code that aims to be embedded into third party websites.

> It really depends on your use case!

### `deleteProperty`

I use the `delete` operator a lot. Setting a property to `undefined` clears its value, but the property is still part of the object. Using the `delete` operator on a property with code like `delete foo.bar` means that the `bar` property will be forever gone from the `foo` object.

```js
var foo = { bar: 'baz' }
foo.bar = 'baz'
console.log('bar' in foo)
// <- true
delete foo.bar
console.log('bar' in foo)
// <- false
```

Remember our [`set`][18] trap example where we prevented access to _`_`-prefixed_ properties? That code had a problem. Even though you couldn't change the value of `_prop`, you could remove the property entirely using the `delete` operator. Even through the `proxy` object!

```js
var target = { _prop: 'foo' }
var proxy = new Proxy(target, handler)
console.log('_prop' in proxy)
// <- true
delete proxy._prop
console.log('_prop' in proxy)
// <- false
```

You can use [`handler.deleteProperty`][10] to prevent a `delete` operation from working. Just like with the `get` and `set` traps, throwing in the `deleteProperty` trap will be enough to prevent the deletion of a property.

```js
var handler = {
  get (target, key) {
    invariant(key, 'get')
    return target[key]
  },
  set (target, key, value) {
    invariant(key, 'set')
    return true
  },
  <mark>deleteProperty (target, key) {</mark>
    invariant(key, 'delete')
    return true
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}
```

If we run the exact same piece of code we tried earlier, we'll run into the exception while trying to delete `_prop` from the `proxy`.

```js
var target = { _prop: 'foo' }
var proxy = new Proxy(target, handler)
console.log('_prop' in proxy)
// <- true
<mark>delete proxy._prop</mark>
// <- Error: Invalid attempt to delete private "_prop" property
```

Deleting properties in your `_private` property space is no longer possible for consumers interacting with `target` through the `proxy`.

### `defineProperty`

We typically use [`Object.defineProperty(obj, key, descriptor)`][25] in two types of situations.

1. When we wanted to ensure cross-browser support of _getters and setters_
2. Whenever we want to define a custom property accessor

Properties added by hand are read-write, they are deletable, and they are enumerable. Properties added through [`Object.defineProperty`][25], _in contrast_, default to being _read-only_, _write-only_, _non-deletable_, and _non-enumerable_ -- in other words, the property starts off being completely **immutable**. You can customize these aspects of the property descriptor, and you can find them below -- alongside with their _default values_ when using [`Object.defineProperty`][25].

- `configurable: false` disables most changes to the property descriptor and makes the property _undeletable_
- `enumerable: false` hides the property from `for..in` loops and `Object.keys`
- `value: undefined` is the initial value for the property
- `writable: false` makes the property value immutable
- `get: undefined` is a method that acts as the getter for the property
- `set: undefined` is a method that receives the new `value` and updates the property's `value`

Note that when defining a property you'll have to choose between using `value` and `writable` or `get` and `set`. When choosing the former you're configuring a _data descriptor_ -- this is the kind you get when declaring properties like `foo.bar = 'baz'`, it has a `value` and it _may or may not_ be `writable`. When choosing the latter you're creating an _accessor descriptor_, which is entirely defined by the methods you can use to `get()` or `set(value)` the value for the property.

The code sample below shows how property descriptors are completely different depending on whether you went for the declarative option or through the programmatic API.

```js
var target = {}
<mark>target.foo = 'bar'</mark>
console.log(Object.getOwnPropertyDescriptor(target, 'foo'))
// <- { value: 'bar', writable: true, enumerable: true, configurable: true }
<mark>Object.defineProperty(target, 'baz', { value: 'ponyfoo' })</mark>
console.log(Object.getOwnPropertyDescriptor(target, 'baz'))
// <- { value: 'ponyfoo', writable: false, enumerable: false, configurable: false }
```

Now that we went over a blitzkrieg overview of [`Object.defineProperty`][25], we can move on to the trap.

#### It's a Trap

The [`handler.defineProperty`][8] trap can be used to intercept calls to `Object.defineProperty`. You get the `key` and the `descriptor` being used. The example below completely prevents the addition of properties through the `proxy`. How cool is it that this intercepts the declarative `foo.bar = 'baz'` property declaration alternative as well? _Quite cool!_

```js
var handler = {
  defineProperty (target, key, descriptor) {
    <mark>return false</mark>
  }
}
var target = {}
var proxy = new Proxy(target, handler)
proxy.foo = 'bar'
// <- TypeError: proxy defineProperty handler returned false for property '"foo"'
```

If we go back to our _"private properties"_ example, we could use the `defineProperty` trap to prevent the creation of private properties through the proxy. We'll reuse the `invariant` method we had to `throw` on attempts to define a property in the _"private `_`-prefixed space"_, and that's it.

```js
var handler = {
  defineProperty (target, key, descriptor) {
    <mark>invariant(key, 'define')</mark>
    return true
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}
```

You could then try it out on a `target` object, setting properties with a `_` prefix will now throw an error. You could make it fail silently by returning `false` _-- depends on your use case!_

```js
var target = {}
var proxy = new Proxy(target, handler)
proxy._foo = 'bar'
// <- Error: Invalid attempt to define private "_foo" property
```

Your `proxy` is now safely hiding `_private` properties behind a trap that guards them from definition through either `proxy[key] = value` or `Object.defineProperty(proxy, key, { value })` _-- pretty amazing!_

### `enumerate`

The [`handler.enumerate`][11] method can be used to trap `for..in` statements. With [`has`][20] we could prevent `key in proxy` from returning `true` for any property in our underscored private space, but what about a `for..in` loop? Even though our `has` trap hides the property from a `key in proxy` check, the consumer will accidentally stumble upon the property when using a `for..in` loop!

```js
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false
    }
    return key in target
  }
}
var target = { _prop: 'foo' }
var proxy = new Proxy(target, handler)
<mark>for (let key in proxy) {</mark>
  console.log(key)
  // <- '_prop'
}
```

We can use the `enumerate` trap to return an _iterator_ that'll be used instead of the [_enumerable properties_][21] found in `proxy` during a `for..in` loop. The returned iterator must conform to the iterator protocol, such as the iterators returned from any [`Symbol.iterator`][22] method. Here's a possible implementation of such a `proxy` that would return the output of `Object.keys` minus the properties found in our _private space_.

```js
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false
    }
    return key in target
  },
  enumerate (target) {
    return Object.keys(target).filter(key => key[0] !== '_')<mark>[Symbol.iterator]()</mark>
  }
}
var target = { pony: 'foo', _bar: 'baz', _prop: 'foo' }
var proxy = new Proxy(target, handler)
for (let key in proxy) {
  console.log(key)
  // <- 'pony'
}
```

Now your private properties are hidden from those prying `for..in` eyes!

### `ownKeys`

The [`handler.ownKeys`][12] method may be used to return an `Array` of properties that will be used as a result for `Reflect.ownKeys()` _-- it should include all properties of `target` (enumerable or not, and symbols too)._ A default implementation, _as seen below_, could just call `Reflect.ownKeys` on the proxied `target` object. Don't worry, we'll get to the `Reflect` built-in later in the [`es6-in-depth`][30] series.

```js
var handler = {
  ownKeys (target) {
    <mark>return Reflect.ownKeys(target)</mark>
  }
}
```

Interception wouldn't affect the output of `Object.keys` in this case.

```js
var target = {
  _bar: 'foo',
  _prop: 'bar',
  [Symbol('secret')]: 'baz',
  pony: 'ponyfoo'
}
var proxy = new Proxy(target, handler)
for (let key of Object.keys(proxy)) {
  console.log(key)
  // <- '_bar'
  // <- '_prop'
  // <- 'pony'
}
```

Do note that the `ownKeys` interceptor is used during all of the following operations.

- `Object.getOwnPropertyNames()` -- just non-symbol properties
- `Object.getOwnPropertySymbols()` -- just symbol properties
- `Object.keys()` -- just non-symbol enumerable properties
- `Reflect.ownKeys()` -- we'll get to `Reflect` later in the series!

In the use case where we want to shut off access to a property space prefixed by `_`, we could take the output of `Reflect.ownKeys(target)` and filter that.

```js
var handler = {
  ownKeys (target) {
    return Reflect.ownKeys(target).filter(<mark>key => key[0] !== '_'</mark>)
  }
}
```

If we now used the `handler` in the snippet above to pull the object keys, we'll just find the properties in the public, non `_`-prefixed space.

```js
var target = {
  _bar: 'foo',
  _prop: 'bar',
  [Symbol('secret')]: 'baz',
  pony: 'ponyfoo'
}
var proxy = new Proxy(target, handler)
for (let key of <mark>Object.keys(proxy)</mark>) {
  console.log(key)
  // <- 'pony'
}
```

Symbol iteration wouldn't be affected by this as `sym[0]` yields `undefined` _-- and in any case decidedly not `'_'`._

```js
var target = {
  _bar: 'foo',
  _prop: 'bar',
  [<mark>Symbol('secret')</mark>]: 'baz',
  pony: 'ponyfoo'
}
var proxy = new Proxy(target, handler)
for (let key of <mark>Object.getOwnPropertySymbols(proxy)</mark>) {
  console.log(key)
  // <- Symbol(secret)
}
```

We were able to hide properties prefixed with `_` from key enumeration while leaving symbols and other properties unaffected.

### `apply`

The [`handler.apply`][13] method is quite interesting. You can use it as a trap on any invocation of `proxy`. All of the following will go through the `apply` trap for your proxy.

```js
proxy(1, 2)
proxy(...args)
proxy.call(null, 1, 2)
proxy.apply(null, [1, 2])
```

The `apply` method takes three arguments.

- `target` -- the function being proxied
- `ctx` -- the context passed as `this` to `target` when applying a call
- `args` -- the arguments passed to `target` when applying the call

A naïve implementation might look like `target.apply(ctx, args)`, but below we'll be using `Reflect.apply(...arguments)`. We'll dig deeper into the `Reflect` built-in later in the series. For now, just think of them as equivalent, and take into account that the value returned by the `apply` trap is also going to be used as the result of a function call through `proxy`.

```js
var handler = {
  apply (target, ctx, args) {
    return <mark>Reflect.apply(...arguments)</mark>
  }
}
```

Besides the obvious _"being able to log all parameters of every function call for `proxy`"_, this trap can be used for parameter balancing and to tweak the results of a function call without changing the method itself _-- and without changing the calling code either._

The example below proxies a `sum` method through a `twice` trap handler that doubles the results of `sum` without affecting the code around it other than using the `proxy` instead of the `sum` method directly.

```js
var twice = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments) <mark>* 2</mark>
  }
}
function sum (left, right) {
  return left + right
}
var proxy = <mark>new Proxy(sum, twice)</mark>
console.log(proxy(1, 2))
// <- 6
console.log(proxy(...[3, 4]))
// <- 14
console.log(proxy.call(null, 5, 6))
// <- 22
console.log(proxy.apply(null, [7, 8]))
// <- 30
```

Naturally, calling `Reflect.apply` on the `proxy` will be caught by the `apply` _trap_ as well.

```js
Reflect.apply(proxy, null, [9, 10])
// <- 38
```

> What else would you use [`handler.apply`][13] for?

Tomorrow I'll publish the last article on `Proxy` _-- Promise! --_ It'll include the remaining _trap_ handlers, such as `construct` and `getPrototypeOf`. _Subscribe below so you don't miss them._

[1]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Proxy#No-op_forwarding_proxy "No-op forwarding proxy on MDN"
[2]: /articles/es6-weakmaps-sets-and-weaksets-in-depth "ES6 WeakMaps, Sets, and WeakSets in Depth on Pony Foo"
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/getPrototypeOf "handler.getPrototypeOf() on MDN"
[4]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/setPrototypeOf "handler.setPrototypeOf() on MDN"
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/isExtensible "handler.isExtensible() on MDN"
[6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/preventExtensions "handler.preventExtensions() on MDN"
[7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/getOwnPropertyDescriptor "handler.getOwnPropertyDescriptor()"
[8]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/defineProperty "handler.defineProperty() on MDN"
[9]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/has "handler.has() on MDN"
[10]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/deleteProperty "handler.deleteProperty() on MDN"
[11]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/enumerate "handler.enumerate() on MDN"
[12]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/ownKeys "handler.ownKeys() on MDN"
[13]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/apply "handler.apply() on MDN"
[14]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/construct "handler.construct() on MDN"
[15]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/get "handler.get() on MDN"
[16]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/set "handler.set() on MDN"
[17]: /articles/es6-template-strings-in-depth "ES6 Template Literals in Depth on Pony Foo"
[18]: /articles/es6-proxies-in-depth#set "ES6 Proxy set trap example"
[19]: https://en.wikipedia.org/wiki/Goldilocks_and_the_Three_Bears "Goldilocks and the Three Bears on Wikipedia"
[20]: #has "ES6 Proxy has trap example"
[21]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in#Description "for..in loops on MDN"
[22]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[23]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[24]:  /articles/es6-classes-in-depth "ES6 Classes in Depth on Pony Foo"
[25]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty "Object.defineProperty() on MDN"
[29]: /articles/es6-proxies-in-depth "ES6 Proxies in Depth on Pony Foo"
[30]:  /articles/tagged/es6-in-depth "Articles tagged es6-in-depth on Pony Foo"
