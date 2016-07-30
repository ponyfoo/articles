## What are Symbols?

Symbols are a new primitive type in ES6. If you ask me, they're _an awful lot like strings_. Just like with numbers and strings, symbols also come with their accompanying `Symbol` wrapper object.

We can create our own Symbols.

```js
var mystery = Symbol()
```

Note that there was no `new`. The `new` operator even throws a `TypeError` when we try it on `Symbol`.

```js
var oops = new Symbol()
// <- TypeError
```

For debugging purposes, you can describe symbols.

```js
var mystery = Symbol('this is a descriptive description')
```

Symbols are _immutable_. Just like numbers or strings. Note however that symbols are _unique_, unlike primitive numbers and strings.

```js
console.log(Symbol() === Symbol())
// <- false
console.log(Symbol('foo') === Symbol('foo'))
// <- false
```

Symbols are _symbols_.

```js
console.log(typeof Symbol())
// <- 'symbol'
console.log(typeof Symbol('foo'))
// <- 'symbol'
```

There are three different flavors of symbols -- each flavor is accessed in a different way. We'll explore each of these and slowly figure out what all of this means.

- You can access local symbols by obtaining a reference to them _directly_
- You can place symbols on the _global registry_ and access them across _realms_
- "Well-known" symbols exist across _realms_ -- but you can't create them and they're not on the _global registry_

What the heck is a _realm_, you say? A _realm_ is **spec-speak** for any execution context, such as the page your application is running in, or an `<iframe>` within your page.

## The "Runtime-Wide" Symbol Registry

There's two methods you can use to add symbols to the runtime-wide symbol registry: `Symbol.for(key)` and `Symbol.keyFor(symbol)`. What do these do?

### `Symbol.for(key)`

This method looks up `key` in the runtime-wide symbol registry. If a symbol with that `key` exists in the global registry, that symbol is returned. If no symbol with that `key` is found in the registry, one is created. That's to say, `Symbol.for(key)` is _idempotent_. In the snippet below, the first call to `Symbol.for('foo')` creates a symbol, adds it to the registry, and returns it. The second call returns that same symbol because the `key` is already in the registry by then -- and associated to the symbol returned by the first call.

```js
Symbol.for('foo') === Symbol.for('foo')
// <- true
```

That is in contrast to what we knew about symbols being unique. The global symbol registry however keeps track of symbols by a `key`. Note that your `key` will also be used as a `description` when the symbols that go into the registry are created. Also note that these symbols are **as global as globals get in JavaScript**, so play nice and use a prefix and don't just name your symbols `'user'` or some generic name like that.

### `Symbol.keyFor(symbol)`

Given a symbol `symbol`, `Symbol.keyFor(symbol)` returns the `key` that was associated with `symbol` when the symbol was added to the global registry.

```js
var symbol = Symbol.for('foo')
console.log(Symbol.keyFor(symbol))
// <- 'foo'
```

### How Wide is Runtime-Wide?

Runtime-wide means the symbols in the global registry are _accessible across code realms_. I'll probably have more success explaining this with a piece of code. It just means the registry is shared across realms.

```js
var frame = document.createElement('iframe')
document.body.appendChild(frame)
console.log(Symbol.for('foo') === frame.contentWindow.Symbol.for('foo'))
// <- true
```

## The "Well-Known" Symbols

Let me put you at ease: **these aren't actually well-known at all.** Far from it. I didn't have any idea these things existed until a few months ago. Why are they _"well-known"_, then? That's because they are JavaScript _built-ins_, and they are used to control parts of the language. They weren't exposed to user code before ES6, but now you can fiddle with them.

A great example of a _"well-known"_ symbol is something we've already been playing with on Pony Foo: the [`Symbol.iterator`][1] well-known symbol. We used that symbol to define the `@@iterator` method on objects that adhere to the _iterator_ protocol. There's [a list of well-known symbols][2] on MDN, but few of them are documented at the time of this writing.

One of the well-known symbols that _is_ documented at this time is [`Symbol.match`][3]. According to MDN, you can set the `Symbol.match` property on regular expressions to `false` and have them behave as string literals when matching _(instead of regular expressions, which don't play nice with [`.startsWith`][4], [`.endsWith`][5], or [`.includes`][6])_.

This part of the spec hasn't been implemented in Babel yet, _-- I assume that's just because it's not worth the trouble --_ but supposedly it goes like this.

```js
var text = '/foo/'
var literal = /foo/
<mark>literal[Symbol.match] = false</mark>
console.log(text.startsWith(literal))
// <- true
```

Why you'd want to do that instead of just casting `literal` to a string _is beyond me_.

```js
var text = '/foo/'
var casted = /foo/<mark>.toString()</mark>
console.log(text.startsWith(casted))
// <- true
```

I suspect the language has **legitimate performance reasons** that warrant the existence of this symbol, but I don't think it'll become a front-end development staple anytime soon.

> Regardless, [`Symbol.iterator`][1] is actually very useful, and I'm sure other well-known symbols are useful as well.

Note that well-known symbols are unique, but **shared across realms**, even when they're not accessible through the _global registry_.

```js
var frame = document.createElement('iframe')
document.body.appendChild(frame)
console.log(Symbol.iterator === frame.contentWindow.Symbol.iterator)
// <- true
```

Not accessible through the _global registry_? Nope!

```js
console.log(Symbol.keyFor(Symbol.iterator))
// <- undefined
```

Accessing them statically from anywhere should be more than enough, though.

## Symbols and Iteration

Any consumer of the _iterable_ protocol obviously ignores symbols other than the well-known [`Symbol.iterator`][1] that would define how to iterate and help identify the object as an _iterable_.

```js
var foo = {
  [Symbol()]: 'foo',
  [Symbol('foo')]: 'bar',
  [Symbol.for('bar')]: 'baz',
  what: 'ever'
}
console.log([...foo])
// <- []
```

The ES5 `Object.keys` method ignores symbols.

```js
console.log(Object.keys(foo))
// <- ['what']
```

Same goes for `JSON.stringify`.

```js
console.log(JSON.stringify(foo))
// <- {"what":"ever"}
```

So, `for..in` then? Nope.

```js
for (let key in foo) {
  console.log(key)
  // <- 'what'
}
```

I know, `Object.getOwnPropertyNames`. Nah! _-- but close._

```js
console.log(Object.getOwnPropertyNames(foo))
// <- ['what']
```

You need to be explicitly looking for symbols to stumble upon them. They're like JavaScript neutrinos. You can use [`Object.getOwnPropertySymbols`][7] to detect them.

```js
console.log(Object.getOwnPropertySymbols(foo))
// <- [Symbol(), Symbol('foo'), Symbol.for('bar')]
```

The magical drapes of symbols drop, and you can now iterate over the symbols with a `for..of` loop to finally figure out the treasures they were guarding. Hopefully, they won't be as disappointing as the flukes in the snippet below.

```js
for (let symbol of Object.getOwnPropertySymbols(foo)) {
  console.log(foo[symbol])
  // <- 'foo'
  // <- 'bar'
  // <- 'baz'
}
```

## Why Would I Want Symbols?

There's a few different uses for symbols.

### Name Clashes

You can use symbols to **avoid name clashes** in property keys. This is important when following the _"objects as hash maps"_ pattern, which regularly ends up failing miserably as native methods and properties are overridden unintentionally _(or maliciously)_.

### "Privacy"?

Symbols are _invisible to all "reflection" methods before ES6_. This can be useful in some scenarios, but they're not private by any stretch of imagination, as we've just demonstrated with the [`Object.getOwnPropertySymbols`][7] API.

That being said, the fact that you have to actively look for symbols to find them means they're useful in situations where you want to define metadata that shouldn't be part of iterable sequences for arrays or any _iterable_ objects.

### Defining Protocols

I think the _biggest use case for symbols_ is exactly what the ES6 implementers use them for: **defining protocols** -- just like there's [`Symbol.iterator`][1] which allows you to define how an object can be iterated.

Imagine for instance a library like [`dragula`][8] defining a protocol through `Symbol.for('dragula.moves')`, where you could add a method on that `Symbol` to any DOM elements. If a DOM element follows the protocol, then `dragula` could call the `el[Symbol.for('dragula.moves')]()` user-defined method to assert whether the element can be moved.

This way, the logic about elements being draggable by `dragula` is shifted from a single place for the entire `drake` _(the `options` for an instance of `dragula`)_, to each individual DOM element. That'd make it easier to deal with complex interactions in larger implementations, as the logic would be delegated to individual DOM nodes instead of being centralized in a single `options.moves` method.

[1]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[2]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol#Well-known_symbols "Well-known symbols on MDN"
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/match "Symbol.match on MDN"
[4]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/startsWith "String.prototype.startsWith() – MDN"
[5]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/endsWith "String.prototype.endsWith() – MDN"
[6]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/includes "String.prototype.includes() – MDN"
[7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols "Object.getOwnPropertySymbols() – MDN"
[8]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
