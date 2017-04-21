# Why Reflection?

Many strongly typed languages have long offered a reflection API _(such as [Python][2] or [C#][1])_, whereas JavaScript hardly has a need for a reflection API -- it _already being_ a dynamic language. The introduction of ES6 features a few **new extensibility points** where the developer gets access to _previously internal aspects_ of the language -- yes, I'm talking about [`Proxy`][3].

You could argue that **JavaScript already has reflection features in ES5**, even though they weren't ever called that by either the specification or the community. Methods like `Array.isArray`, `Object.getOwnPropertyDescriptor`, and even `Object.keys` are classical examples of what you'd find **categorized as reflection** in other languages. The `Reflect` built-in is, going forward, going to house future methods in the category. That makes a lot of sense, right? Why would you have __super reflection*y* static methods__ like `getOwnPropertyDescriptor` _(or even `create`)_ in `Object`? After all, `Object` is meant to be a base prototype, and not so much a repository of reflection methods. Having a dedicated interface that exposes most reflection methods makes more sense.

# `Reflect`

We've mentioned the `Reflect` object in passing the past few days. Much like `Math`, `Reflect` is a static object you can't `new` up nor _call_, and all of its methods are static. The _traps in [ES6 proxies][3] _(covered [here][4] and [here][5])_ are **mapped one-to-one** to the `Reflect` API. For every _trap_, there's a matching reflection method in `Reflect`.

The reflection API in JavaScript has _a number of benefits_ that are worth examining.

## Return Values in `Reflect` vs Reflection Through `Object`

The `Reflect` equivalents to reflection methods on `Object` also provide more **meaningful** _return values_. For instance, the `Reflect.defineProperty` method returns a boolean value indicating whether the property was successfully defined. Meanwhile, its `Object.defineProperty` counterpart returns the object it got as its first argument _-- not very **useful**._

To illustrate, below is a code snippet showing how to verify `Object.defineProperty` worked.

```js
try {
  Object.defineProperty(target, 'foo', { value: 'bar' })
  // yay!
} catch (e) {
  // oops.
}
```

As opposed to a _much more natural_ `Reflect.defineProperty` experience.

```js
var yay = Reflect.defineProperty(target, 'foo', { value: 'bar' })
if (yay) {
  // yay!
} else {
  // oops.
}
```

This way we avoided a `try`/`catch` block and made our code a little more maintainable in the process.

## Keyword Operators as First Class Citizens

Some of these reflection methods provide programmatic alternatives of doing things that were previously only possible through keywords. For example, `Reflect.deleteProperty(target, key)` is equivalent to the `delete target[key]` expression. Before ES6, if you wanted a method call to result in a `delete` call, you'd have to create a dedicated utility method that wrapped `delete` on your behalf.

```js
var target = { foo: 'bar', baz: 'wat' }
delete target.foo
console.log(target)
// <- { baz: 'wat' }
```

Today, with ES6, you already have such a method in `Reflect.deleteProperty`.

```js
var target = { foo: 'bar', baz: 'wat' }
Reflect.deleteProperty(target, 'foo')
console.log(target)
// <- { baz: 'wat' }
```

Just like `deleteProperty`, there's a few other methods that make it easy to do other things too.

## Easier to mix `new` with Arbitrary Argument Lists

In ES5, this is a hard problem: How do you create a `new Foo` passing an arbitrary number of arguments? You can't do it directly, and it's [super verbose][7] if you need to do it anyways. You have to create an intermediary object that gets passed the arguments as an `Array`. Then you have _that_ object's constructor return the result of applying the constructor of the object you originally intended to `.apply`. Straightforward, right? _-- What do you mean **no**?_

```js
var proto = Dominus.prototype
<mark>Applied.prototype = proto</mark>
function Applied (args) {
  return Dominus<mark>.apply(this, args)</mark>
}
function apply (a) {
  return <mark>new</mark> Applied(a)
}
```

Using `apply` is actually easy, thankfully.

```js
apply(['.foo', '.bar'])
apply.call(null, '.foo', '.bar')
```

But that was _insane_, right? **Who does that?** Well, in ES5, everyone who has a valid reason to do it! Luckily ES6 has less insane approaches to this problem. One of them is simply to use the [spread operator][2].

```js
new Dominus(<mark>...args</mark>)
```

Another alternative is to go the `Reflect` route.

```js
<mark>Reflect.construct</mark>(Dominus, args)
```

Both of these are tremendously simpler than what I had to do in the [`dominus`][7] codebase.

## Function Application, The Right Way

In ES5 if we want to call a method with an arbitrary number of arguments, we can use `.apply` passing a `this` context and our arguments.

```js
fn.apply(ctx, [1, 2, 3])
```

If we fear `fn` might shadow `apply` with a property of their own, we can rely on a safer but way more verbose alternative.

```js
Function.prototype.apply.call(fn, ctx, [1, 2, 3])
```

In ES6, you can use [spread][6] as an alternative to `.apply` for an arbitrary number of arguments.

```js
fn(<mark>...[1, 2, 3]</mark>)
```

That doesn't solve your problems when you need to define a `this` context, though. You could go back to the `Function.prototype` way but that's _way_ too verbose. Here's how `Reflect` can help.

```js
Reflect.apply(fn, ctx, args)
```

Naturally, one of the most fitting use cases for `Reflect` API methods is default behavior in [`Proxy`][3] [traps][4].

## Default Behavior in `Proxy` Traps

We've already talked about how _traps_ are mapped one-to-one to `Reflect` methods. We haven't yet touched on the fact that their interfaces match as well. That is to say, _both their arguments and their return values match_. In code, this means you could do something like this to get the default `get` _trap_ behavior in your [proxy handlers][3].

```js
var handler = {
  get () {
    return <mark>Reflect.get(...arguments)</mark>
  }
}
var target = { a: 'b' }
var proxy = new Proxy(target, handler)
console.log(proxy.a)
// <- 'b'
```

There is, in fact, nothing stopping you from making that `handler` even simpler. Of course, at this point you'd be better off leaving the _trap_ out entirely.

```js
var handler = {
  get: Reflect.get
}
```

The important take-away here is that you could set up a trap in your proxy handlers, wire up some custom functionality that ends up throwing or logging a console statement, and then in the default case you could just use the one-liner recipe found below.

```js
return Reflect[trapName](...arguments)
```

Certainly puts me at ease when it comes to demystifying [`Proxy`][3].

## Lastly, There's `__proto__`

Yesterday we talked about how the [legacy `__proto__` is part of the ES6 specification][5] but still strongly advised against and how you should use `Object.setPrototypeOf` and `Object.getPrototypeOf` instead. Turns out, there's also `Reflect` counterparts to those methods you could use. Think of these methods as _getter and setters_ for `__proto__` but without the cross-browser discrepancies.

> I wouldn't just hop onto the "`setPrototypeOf` all the things" bandwagon just yet. In fact, I hope there never is a train pulling that wagon to begin with.

[1]: http://www.codeproject.com/Articles/17269/Reflection-in-C-Tutorial "Reflection in C# Tutorial"
[2]: http://www.diveintopython.net/power_of_introspection/ "The Power of Introspection -- Dive Into Python"
[3]: /articles/es6-proxies-in-depth "ES6 Proxies in Depth on Pony Foo"
[4]: /articles/es6-proxy-traps-in-depth "ES6 Proxy Traps in Depth on Pony Foo"
[5]: /articles/more-es6-proxy-traps-in-depth "More ES6 Proxy Traps in Depth on Pony Foo"
[6]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[7]: https://github.com/bevacqua/dominus/blob/master/src/apply.js#L4-L14 "Dominus on GitHub has an example where this was necessary"
