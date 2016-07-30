## But Wait! There's More... _(Proxy Trap Handlers)_

This article covers all the trap handlers that weren't covered by the two previous articles on proxies. For the most part, the traps that we discussed yesterday had to do with property manipulation, while the first five traps we'll dig into today have mostly to do with the object being proxied _itself_. The last two have to do with properties once again -- but they're a bit more involved than yesterday's traps, which were much easier to "fall into" _(the trap -- muahaha)_ in your everyday code.

- [`construct`](#construct) -- traps usage of the `new` operator
- [`getPrototypeOf`](#getprototypeof) -- traps internal calls to `[[GetPrototypeOf]]`
- [`setPrototypeOf`](#setprototypeof) -- traps calls to `Object.setPrototypeOf`
- [`isExtensible`](#isextensible) -- traps calls to `Object.isExtensible`
- [`preventExtensions`](#preventextensions) -- traps calls to `Object.preventExtensions`
- [`getOwnPropertyDescriptor`](#getownpropertydescriptor) -- traps calls to `Object.getOwnPropertyDescriptor`

### `construct`

You can use the [`handler.construct`][14] method to trap usage of the `new` operator. Here's a quick _"default implementation"_ that doesn't alter the behavior of `new` at all. _Remember our friend the [spread operator][23]?_

```js
var handler = {
  construct (target, args) {
    return new target(<mark>...args</mark>)
  }
}
```

If you use the `handler` options above, the `new` behavior you're already used to would remain unchanged. That's great because it means whatever you're trying to accomplish you can still fall back to the **default behavior** _-- and that's always important._

```js
function target (a, b, c) {
  this.a = a
  this.b = b
  this.c = c
}
var proxy = new Proxy(target, handler)
console.log(new proxy(1,2,3))
// <- { a: 1, b: 2, c: 3 }
```

Obvious use cases for `construct` traps include data massaging in the arguments, doing things that should always be done around a call to `new proxy()`, logging and tracking object creation, and swapping implementations entirely. Imagine a proxy like the following in situations where you have inheritance _"branching"_.

```js
class Automobile {}
class Car extends Automobile {}
class SurveillanceVan extends Automobile {}
class SUV extends Automobile {}
class SportsCar extends Car {}
function target () {}
var handler = {
  construct (target, args) {
    var <mark>[status]</mark> = args
    if (status === 'nsa') {
      return <mark>new SurveillanceVan(...args)</mark>
    }
    if (status === 'single') {
      return new SportsCar(...args)
    }
    return new SUV(...args) // family
  }
}
```

Naturally, you could've used a regular method for the branching part, but using the `new` operator also makes sense in these types of situations, as you'll end up creating a new object in all code branches anyways.

```js
console.log(new proxy('nsa').constructor.name)
// <- `SurveillanceVan`
```

The most common use case for `construct` traps yet may be something simpler, and that's extending the `target` object right after creation, _-- and before anything else happens --_ in such a way that it better supports the `proxy` gatekeeper. You might have to add a `proxied` flag to the `target` object, or something akin to that.

### `getPrototypeOf`

You can use the [`handler.getPrototypeOf`][3] method as a trap for all of the following.

- [`Object.prototype.__proto__`][31] property
- [`Object.prototype.isPrototypeOf()`][32] method
- [`Object.getPrototypeOf()`][33] method
- [`Reflect.getPrototypeOf()`][34] method
- [`instanceof`][35] operator

You could use this _trap_ to make an object pretend it's an `Array`, when accessed through the proxy. However, note that that on its own isn't sufficient for the `proxy` to be an actual `Array`.

```js
var handler = {
  getPrototypeOf: <mark>target => Array.prototype</mark>
}
var target = {}
var proxy = new Proxy(target, handler)
console.log(proxy instanceof Array)
// <- true
console.log(proxy.push)
// <- undefined
```

Naturally, you could keep on patching your `proxy` until you get the behavior you want. In this case, you may want to use a `get` trap to mix the `Array.prototype` with the actual back-end `target`. Whenever a property isn't found on the `target`, we'll use reflection to look it up on `Array.prototype`. It turns out, this is **good enough** for most operations.

```js
var handler = {
  getPrototypeOf: target => Array.prototype,
  get (target, key) {
    return Reflect.get(target, key) || <mark>Reflect.get(Array.prototype, key)</mark>
  }
}
var target = {}
var proxy = new Proxy(target, handler)
console.log(proxy.push)
// <- function push () { [native code] }
<mark>proxy.push('a', 'b')</mark>
console.log(proxy)
// <- { 0: 'a', 1: 'b', length: 2 }
```

I definitely see some advanced use cases for `getPrototypeOf` traps in the future, but it's too early to tell what patterns may come out of it.

### `setPrototypeOf `

The [`Object.setPrototypeOf`][30] method does exactly what its name conveys: it sets the prototype of an object to a reference to another object. It's considered the proper way of setting the prototype as opposed to using `__proto__` which is a legacy feature _-- and now standarized as such._

You can use the [`handler.setPrototypeOf`][4] method to set up a _trap_ for [`Object.setPrototypeOf`][30]. The snippet of code shown below doesn't alter the default behavior of changing a prototype to the value of `proto`.

```js
var handler = {
  setPrototypeOf (target, proto) {
    <mark>Object.setPrototypeOf(target, proto)</mark>
  }
}
var proto = {}
var target = function () {}
var proxy = new Proxy(target, handler)
proxy.setPrototypeOf(proxy, proto)
console.log(proxy.prototype === proto)
// <- true
```

The field for `setPrototypeOf` is pretty open. You could simply not call `Object.setPrototypeOf` and the trap would sink the call into a _no-op_. You could `throw` an exception making the failure more explicit -- for instance if you deem the new prototype to be invalid or you don't want consumers pulling the rug from under your feet.

This is a nice _trap_ to have if you want proxies to have limited access to what they can do with your `target` object. I would definitely implement a trap like the one below if I had any security concerns at all in a proxy I'm passing away to third party code.

```js
var handler = {
  setPrototypeOf (target, proto) {
    <mark>throw new Error('Changing the prototype is forbidden')</mark>
  }
}
var proto = {}
var target = function () {}
var proxy = new Proxy(target, handler)
proxy.setPrototypeOf(proxy, proto)
// <- Error: Changing the prototype is forbidden
```

Then again, you may want to fail silently with no error being thrown at all if you'd rather confuse the consumer -- and that may just make them go away.

### `isExtensible`

The [`handler.isExtensible`][5] method can be mostly used for logging or auditing calls to `Object.isExtensible`. This _trap_ is subject to a harsh invariant that puts a hard limit to what you can do with it.

> If `Object.isExtensible(proxy) !== Object.isExtensible(target)`, then a `TypeError` is thrown.

You could use the [`handler.isExtensible`][5] trap to `throw` if you don't want consumers to know whether the original object is extensible or not, but there seem to be limited situations that would warrant such an **incarnation of evil**. For completeness' sake, the piece of code below shows a trap for `isExtensible` that throws errors every once in a while, but otherwise behaves as expected.

```js
var handler = {
  isExtensible (target) {
    if (Math.random() > 0.1) {
      throw new Error('gotta love sporadic obscure errors!')
    }
    return <mark>Object.isExtensible(target)</mark>
  }
}
var target = {}
var proxy = new Proxy(target, handler)
console.log(Object.isExtensible(proxy))
// <- true
console.log(Object.isExtensible(proxy))
// <- true
console.log(Object.isExtensible(proxy))
// <- true
console.log(Object.isExtensible(proxy))
// <- <mark>Error: gotta love sporadic obscure errors!</mark>
```

While this _trap_ is nearly useless other than for auditing purposes and to cover all your bases, the _hard-to-break_ invariant makes sense because there's also the `preventExtensions` _trap_. **That one is a little bit more useful!**

### `preventExtensions`

You can use [`handler.preventExtensions`][6] to _trap_ the `Object.preventExtensions` method. When extensions are prevented on an object, new properties can't be added any longer _-- it can't be extended._

Imagine a scenario where you want to selectively be able to `preventExtensions` on some objects -- but not all of them. In this scenario, you could use a `WeakSet` to keep track of the objects that should be extensible. If an object is in the set, then the `preventExtensions` _trap_ should be able to capture those requests and discard them. The snippet below does exactly that. Note that the _trap_ always returns the opposite of `Object.isExtensible(target)`, because it should report whether the object has been made non-extensible.

```js
var mustExtend = <mark>new WeakSet()</mark>
var handler = {
  preventExtensions (target) {
    if (<mark>!mustExtend.has(target)</mark>) {
      Object.preventExtensions(target)
    }
    return <mark>!Object.isExtensible(target)</mark>
  }
}
```

Now that we've set up the `handler` and our `WeakSet`, we can create a back-end object, a `proxy`, and add the back-end to our set. Then, you can try `Object.preventExtensions` on the proxy and you'll notice it fails to prevent extensions to the object. This is the intended behavior as the `target` can be found in the `mustExtend` set.

```js
var target = {}
var proxy = new Proxy(target, handler)
<mark>mustExtend.add(target)</mark>
Object.preventExtensions(proxy)
// <- TypeError: proxy preventExtensions handler returned false
```

If we removed the `target` from the `mustExtend` set before calling `Object.preventExtensions`, then `target` would be made non-extensible as originally intended.

```js
var target = {}
var proxy = new Proxy(target, handler)
mustExtend.add(target)
<mark>mustExtend.delete(target)</mark>
Object.preventExtensions(proxy)
console.log(<mark>Object.isExtensible(proxy)</mark>)
// <- false
```

Naturally, you could use this distinction to prevent `proxy` consumers from making the proxy non-extensible in cases where that could lead to undesired behavior. In most cases, you probably won't have to deal with this _trap_, though. That's because you're usually going to be working with the _back-end `target`_ for the most part, and not so much with the `proxy` object itself.

### `getOwnPropertyDescriptor`

You can use the [`handler.getOwnPropertyDescriptor`][7] method as a trap for [`Object.getOwnPropertyDescriptor`][26]. It may return a property descriptor, such as the result from `Object.getOwnPropertyDescriptor(target, key)`; or `undefined`, signaling that the property doesn't exist. As usual, you also have the third option of throwing an exception, aborting the operation entirely.

If we go back to our canonical _"private property space"_ example, we could implement a _trap_ such as the one seen below to prevent consumers from learning about property descriptors of private properties.

```js
var handler = {
  <mark>getOwnPropertyDescriptor (target, key) {</mark>
    invariant(key, 'get property descriptor for')
    return Object.getOwnPropertyDescriptor(target, key)
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`)
  }
}
var target = {}
var proxy = new Proxy(target, handler)
Object.getOwnPropertyDescriptor(proxy, '_foo')
// <- <mark>Error: Invalid attempt to get property descriptor for private "_foo" property</mark>
```

The problem with that approach is that you're effectively telling consumers that properties with the `_` prefix are somehow off-limits. It might be best to conceal them entirely by returning `undefined`. This way, your private properties will behave no differently than properties that are _actually absent_ from the `target` object.

```js
var handler = {
  getOwnPropertyDescriptor (target, key) {
    if (key[0] === '_') {
      <mark>return</mark>
    }
    return Object.getOwnPropertyDescriptor(target, key)
  }
}
var target = { <mark>_foo</mark>: 'bar', baz: 'tar' }
var proxy = new Proxy(target, handler)
console.log(Object.getOwnPropertyDescriptor(proxy, 'wat'))
// <- undefined
console.log(<mark>Object.getOwnPropertyDescriptor(proxy, '_foo')</mark>)
// <- undefined
console.log(Object.getOwnPropertyDescriptor(proxy, 'baz'))
// <- { value: 'tar', writable: true, enumerable: true, configurable: true }
```

Usually when you're trying to hide things it's best to have them try and behave as if they fell in some other category than the category they're actually in. Throwing, however, just sends the _"there's something sketchy here, but we can't quite tell you what that is..."_ message -- and the consumer will eventually find out why that is.

> Keep in mind that if debugging concerns outweight security concerns, you probably should go for the `throw` statement.

# Conclusions

This has certainly been fun. I now have a much better understanding of what proxies can do for me, and I think they'll be an instant hit once ES6 starts gaining more traction. I for one can't be more excited about them becoming well-supported in more browsers soon. I wouldn't hold out my hopes about proxies for Babel, as many of the traps are **ridiculously hard** _(or downright impossible)_ to implement in ES5.

As we've learned over the last few days, there's **a myriad use cases** for proxies. Off the top of my head, we can use `Proxy` for all of the following.

- Add validation rules _-- and enforce them --_ on plain old JavaScript objects
- Keep track of every interaction that goes through a proxy
- Decorate objects without changing them at all
- Make certain properties on an object completely invisible to the consumer of a proxy
- Revoke access _at will_ when the consumer should no longer be able to use a proxy
- Modify the arguments passed to a proxied method
- Modify the result produced by a proxied method
- Prevent deletion of specific properties through the proxy
- Prevent new definitions from succeeding, according to the desired property descriptor
- Shuffle arguments around in a constructor
- Return a result other than the object being `new`-ed up in a constructor
- Swap out the prototype of an object for something else

I can say without a shadow of a doubt that there's **hundreds more of use cases** for proxies. I'm sure many libraries will adopt a pattern we've discussed here in the series where a _"back-end"_ `target` object is created and used for storage purposes but the consumer is only provided with a _"front-end"_ `proxy` object with _limited and audited interaction_ with the back-end.

> What would _you_ use `Proxy` for?

Meet me tomorrow at.. say -- _same time?_ We can talk about `Reflect` then.

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
[20]: /articles/es6-proxy-traps-in-depth#has "ES6 Proxy has trap example"
[21]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in#Description "for..in loops on MDN"
[22]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[23]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[24]:  /articles/es6-classes-in-depth "ES6 Classes in Depth on Pony Foo"
[25]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty "Object.defineProperty() on MDN"
[26]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor "Object.getOwnPropertyDescriptor() on MDN"
[29]: /articles/es6-proxies-in-depth "ES6 Proxies in Depth on Pony Foo"
[30]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf "Object.setPrototypeOf() on MDN"
[31]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto "__proto__ on MDN"
[32]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf "Object.prototype.isPrototypeOf() on MDN"
[33]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf "Object.getPrototypeOf() on MDN"
[34]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/getPrototypeOf "Reflect.getPrototypeOf() on MDN"
[35]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof "instanceof on MDN"
