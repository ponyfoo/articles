# ES6 Proxies

Proxies are a quite interesting feature coming in ES6. In a nutshell, you can use a `Proxy` to determine behavior whenever the properties of a `target` object are accessed. A `handler` object can be used to configure _traps_ for your `Proxy`, as we'll see in a bit.

By default, proxies don't do much -- in fact they don't do anything. If you don't set any _"options"_, your `proxy` will just work as a _pass-through_ to the `target` object -- MDN calls this a ["no-op forwarding `Proxy`"][1], which makes sense.

```js
var target = {}
var handler = {}
var proxy = new Proxy(target, handler)
proxy.a = 'b'
console.log(target.a)
// <- 'b'
console.log(proxy.c === undefined)
// <- true
```

We can make our proxy a bit more interesting by adding traps. Traps allow you to intercept interactions with `target` in different ways, as long as those interactions happen through `proxy`. We could use a `get` _trap_ to log every attempt to pull a value out of a property in `target`. Let's try that next.

## `get`

The proxy below is able to track any and every **property access** event because it has a [`handler.get`][15] trap. It can also be used to _transform_ the value we get out of accessing any given property. We can already imagine `Proxy` becoming a staple when it comes to developer tooling.

```js
var handler = {
  get (target, key) {
    console.info(`Get on property "${key}"`)
    return target[key]
  }
}
var target = {}
var proxy = new Proxy(target, handler)
proxy.a = 'b'
proxy.a
// <- 'Get on property "a"'
proxy.b
// <- 'Get on property "b"'
```

Of course, your getter doesn't necessarily have to return the original `target[key]` value. How about finally making those `_prop` properties actually private?

## `set`

Know how we usually define conventions such as Angular's _dollar signs_ where properties prefixed by a single dollar sign should hardly be accessed from an application and properties prefixed by two dollar signs should **not be accessed at all**? We usually do something like that ourselves in our applications, typically in the form of underscore-prefixed variables.

The `Proxy` in the example below prevents property access for both `get` and `set` _(via a [`handler.set`][16] trap)_ while accessing `target` through `proxy`. Note how `set` always returns `true` here? -- this means that setting the property `key` to a given `value` should _succeed_. If the return value for the `set` trap is `false`, setting the property value will throw a `TypeError` under strict mode, and otherwise fail silently.

```js
var handler = {
  get (target, key) {
    invariant(key, 'get')
    return target[key]
  },
  set (target, key, value) {
    invariant(key, 'set')
    <mark>return true</mark>
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(<mark>`Invalid attempt to ${action} private "${key}" property`</mark>)
  }
}
var target = {}
var proxy = new Proxy(target, handler)
proxy.a = 'b'
console.log(proxy.a)
// <- 'b'
proxy._prop
// <- Error: Invalid attempt to get private "_prop" property
proxy._prop = 'c'
// <- Error: Invalid attempt to set private "_prop" property
```

<sub>_You do remember string interpolation with [template literals][17], right?_</sub>

It might be worth mentioning that the `target` object _(the object being proxied)_ should often be completely hidden from accessors in proxying scenarios. Effectively **preventing direct access** to the `target` and instead forcing access to `target` exclusively through `proxy`. Consumers of `proxy` will get to access `target` through the `Proxy` object, but will have to **obey your access rules** -- such as _"properties prefixed with `_` are off-limits"_.

To that end, you could simply wrap your proxied object in a method, and then return the `proxy`.

```js
function proxied () {
  var target = {}
  var handler = {
    get (target, key) {
      invariant(key, 'get')
      return target[key]
    },
    set (target, key, value) {
      invariant(key, 'set')
      return true
    }
  }
  return <mark>new Proxy(target, handler)</mark>
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}
```

Usage stays the same, except now access to `target` is completely governed by `proxy` and its mischievous traps. At this point, any `_prop` properties in `target` are completely inaccessible through the proxy, and since `target` can't be accessed directly from outside the `proxied` method, they're sealed off from consumers for good.

You might be tempted to argue that you could achieve the same behavior in ES5 simply by using variables privately scoped to the `proxied` method, without the need for the `Proxy` itself. The big difference is that proxies allow you to "privatize" property access **on different layers**. Imagine an underlying `underling` object that already has several _"private"_ properties, which you still access in some other `middletier` module that has intimate knowledge of the internals of `underling`. The `middletier` module could return a `proxied` version of `underling` without having to map the API onto an entirely new object in order to protect those internal variables. Just locking access to any of the "private" properties would suffice!

Here's a use case on schema validation using proxies.

## Schema Validation with Proxies

While, yes, _you could_ set up schema validation on the `target` object itself, doing it on a `Proxy` means that you separate the validation concerns from the `target` object, which will go on to live as a **POJO** _(Plain Old JavaScript Object)_ happily ever after. Similarly, you can use the proxy as an intermediary for access to many different objects that conform to a schema, without having to rely on prototypal inheritance or [ES6 `class` classes][24].

In the example below, `person` is a plain model object, and we've also defined a `validator` object with a `set` trap that will be used as the `handler` for a `proxy` validator of people models. As long as the `person` properties are set through `proxy`, the model invariants will be satisfied according to our validation rules.

```js
var validator = {
  set (target, key, value) {
    if (key === 'age') {
      if (typeof value !== 'number' || Number.isNaN(value)) {
        throw new TypeError('Age must be a number')
      }
      if (value <= 0) {
        throw new TypeError('Age must be a positive number')
      }
    }
    return true
  }
}
var person = { age: 27 }
var proxy = new Proxy(person, validator)
proxy.age = 'foo'
// <- TypeError: Age must be a number
proxy.age = NaN
// <- TypeError: Age must be a number
proxy.age = 0
// <- TypeError: Age must be a positive number
proxy.age = 28
console.log(person.age)
// <- 28
```

There's also a particularly "severe" type of proxies that allows us to completely shut off access to `target` whenever we deem it necessary.

## Revocable Proxies

We can use `Proxy.revocable` in a similar way to `Proxy`. The main differences are that the return value will be `{ proxy, revoke }`, and that once `revoke` is called the `proxy` **will throw** on _any operation_. Let's go back to our pass-through `Proxy` example and make it `revocable`. Note that we're _not using_ the `new` operator here. Calling `revoke()` over and over has no effect.

```js
var target = {}
var handler = {}
var <mark>{proxy, revoke} = Proxy.revocable(target, handler)</mark>
proxy.a = 'b'
console.log(proxy.a)
// <- 'b'
<mark>revoke()</mark>
revoke()
revoke()
console.log(proxy.a)
// <- TypeError: illegal operation attempted on a revoked proxy
```

This type of `Proxy` is particularly useful because you can now completely cut off access to the `proxy` granted to a consumer. You start by passing of a revocable `Proxy` and keeping around the `revoke` method _(hey, maybe you can [use a `WeakMap`][2] for that)_, and when its clear that the consumer shouldn't have access to `target` anymore, -- not even through `proxy` -- you `.revoke()` the hell out of their access. _Goodbye consumer!_

Furthermore, since `revoke` is available on the same scope where your `handler` traps live, you could set up **extremely paranoid rules** such as _"if a consumer attempts to access a private property more than once, revoke their `proxy` entirely"_.

> Check back tomorrow for the second part of the article about proxies, which discusses `Proxy` _traps_ beyond `get` and `set`.

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
[18]: #set "ES6 Proxy set trap example"
[19]: https://en.wikipedia.org/wiki/Goldilocks_and_the_Three_Bears "Goldilocks and the Three Bears on Wikipedia"
[20]: #has "ES6 Proxy has trap example"
[21]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in#Description "for..in loops on MDN"
[22]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[23]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[24]:  /articles/es6-classes-in-depth "ES6 Classes in Depth on Pony Foo"
[25]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty "Object.defineProperty() on MDN"
