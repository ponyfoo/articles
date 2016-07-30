## Property Value Shorthands

Whenever you find yourself assigning a property value that matches a property name, you can omit the property value, it's implicit in ES6.

```js
var foo = 'bar'
var baz = { foo }
console.log(baz.foo)
// <- 'bar'
```

In the snippet shown below I re-implemented part of `localStorage` in memory as a polyfill. It displays a pattern that I've followed countless times [in my code][1].

```js
var ms = {}

function getItem (key) {
  return key in ms ? ms[key] : null
}

function setItem (key, value) {
  ms[key] = value
}

function clear () {
  ms = {}
}

<mark>module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
}</mark>
```

The reasons why _-- most often --_ I don't place functions directly on an object definition are _several._

- Less indentation needed
- Public API stands out
- Harder to tightly couple methods
- Easier to reason about

With ES6, we can throw another bullet into that list, and that's that the export can be even easier using _property value shorthands_. You can omit the property value if it matches the property name. The `module.exports` from the code above thus becomes:

```js
module.exports = { getItem, setItem, clear }
```

So good!

## Computed Property Names

We already covered computed property names briefly in the [destructuring article][2]. This was a very common thing to do for me:

```js
var foo = 'bar'
var baz = {}
baz[foo] = 'ponyfoo'
console.log(baz)
// <- { bar: 'ponyfoo' }
```

Computed property names allow you to write an _expression_ wrapped in square brackets instead of the regular property name. Whatever the expression evaluates to will become the property name.

```js
var foo = 'bar'
var baz = { [foo]: 'ponyfoo' }
console.log(baz)
// <- { bar: 'ponyfoo' }
```

One limitation of computed property names is that you won't be able to use the shorthand expression with it. I presume this is because shorthand expression is meant to be simple, compile-time sugar.

```js
var foo = 'bar'
var bar = 'ponyfoo'
var baz = { [foo] }
console.log(baz)
// <- SyntaxError
```

That being said, I believe this to be the most common use case. Here our code is simpler because we don't have to spend three steps in allocating a `foo` variable, assigning to `foo[type]`, and returning `foo`. Instead we can do all three in a single statement.

```js
function getModel (type) {
  return {
    [type]: {
      message: 'hello, this is doge',
      date: new Date()
    }
  }
}
```

Neat. What else?

## Method Definitions

Typically in ES5 you declare methods on an object like so:

```js
var foo = {
  bar: function (baz) {
  }
}
```

While getters and setters have a syntax like this, where there's no need for the `function` keyword. It's just inferred from context.

```js
var cart = {
  _wheels: 4,
  get wheels () {
    return this._wheels
  },
  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('hey, come back here!')  
    }
    this._wheels = value
  }
}
```

Starting in ES6, you can declare regular methods with a similar syntax, only difference is it's not prefixed by `get` or `set`.

```js
var cart = {
  _wheels: 4,
  get wheels () {
    return this._wheels
  },
  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('hey, come back here!')  
    }
    this._wheels = value
  },
  <mark>dismantle () {
    this._wheels = 0
    console.warn(`you're all going to pay for this!`)
  }</mark>
}
```

I think it's nice that methods converged together with getters and setter. I for one don't use this syntax a lot because I like to name my functions and decouple them from their host objects as I explained in the [shorthand][3] section. However, it's still useful in some situations and definitely useful when declaring _"classes"_ -- if you're into that sort of thing.

[1]: https://github.com/bevacqua/local-storage/blob/b9725b0fc77faabc737ba7c6ee57d343afa95102/stub.js#L3-L32 "See bevacqua/local-storage on GitHub"
[2]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
[3]: #property-value-shorthands
