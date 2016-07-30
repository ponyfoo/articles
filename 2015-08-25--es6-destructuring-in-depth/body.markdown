## Destructuring

This is easily one of the features I've been using the most. It's also one of the simplest. It binds properties to as many variables as you need and it works with both Arrays and Objects.

```js
var foo = { bar: 'pony', baz: 3 }
var {bar, baz} = foo
console.log(bar)
// <- 'pony'
console.log(baz)
// <- 3
```

It makes it very quick to pull out a specific property from an object. You're also allowed to map properties into aliases as well.

```js
var foo = { bar: 'pony', baz: 3 }
var {bar: a, baz: b} = foo
console.log(a)
// <- 'pony'
console.log(b)
// <- 3
```

You can also pull properties as deep as you want, and you could also alias those deep bindings.

```js
var foo = { bar: { deep: 'pony', dangerouslySetInnerHTML: 'lol' } }
var {bar: { deep, dangerouslySetInnerHTML: sure }} = foo
console.log(deep)
// <- 'pony'
console.log(sure)
// <- 'lol'
```

By default, properties that aren't found will be `undefined`, just like when accessing properties on an object with the dot or bracket notation.

```js
var {foo} = {bar: 'baz'}
console.log(foo)
// <- undefined
```

If you're trying to access a deeply nested property of a parent that doesn't exist, then you'll get an exception, though.

```js
var {foo:{bar}} = {baz: 'ouch'}
// <- Exception
```

That makes a lot of sense, if you think of destructuring as sugar for ES5 like the code below.

```js
var _temp = { baz: 'ouch' }
var bar = _temp.foo.bar
// <- Exception
```

A cool property of destructuring is that it allows you to swap variables without the need for the infamous `aux` variable.

```js
function es5 () {
  var left = 10
  var right = 20
  var aux
  if (right > left) {
    aux = right
    right = left
    left = aux
  }
}
```

```js
function es6 () {
  var left = 10
  var right = 20
  if (right > left) {
    <mark>[left, right] = [right, left]</mark>
  }
}
```

Another convenient aspect of destructuring is the ability to pull keys using [computed property names][3].

```js
var key = 'such_dynamic'
var { <mark>[key]</mark>: foo } = { such_dynamic: 'bar' }
console.log(foo)
// <- 'bar'
```

In ES5, that'd take an extra statement and variable allocation on your behalf.

```js
var key = 'such_dynamic'
var baz = { such_dynamic: 'bar' }
var foo = baz[key]
console.log(foo)
```

You can also define default values, for the case where the pulled property evaluates to `undefined`.

```js
var {foo=3} = { foo: 2 }
console.log(foo)
// <- 2
var {foo=3} = { foo: undefined }
console.log(foo)
// <- 3
var {foo=3} = { bar: 2 }
console.log(foo)
// <- 3
```

Destructuring works for Arrays as well, as we mentioned earlier. Note how I'm **using square brackets** in the destructuring side of the declaration now.

```js
var [a] = [10]
console.log(a)
// <- 10
```

Here, again, we can use the default values and follow the same rules.

```js
var [a] = []
console.log(a)
// <- undefined
var [b=10] = [undefined]
console.log(b)
// <- 10
var [c=10] = []
console.log(c)
// <- 10
```

When it comes to Arrays you can conveniently skip over elements that you don't care about.

```js
var [,,a,b] = [1,2,3,4,5]
console.log(a)
// <- 3
console.log(b)
// <- 4
```

You can also use destructuring in a `function`'s parameter list.

```js
function greet ({ age, name:greeting='she' }) {
  console.log(<mark>`${greeting} is ${age} years old.`</mark>)
}
greet({ name: 'nico', age: 27 })
// <- 'nico is 27 years old'
greet({ age: 24 })
// <- 'she is 24 years old'
```

That's roughly **how** you can use destructuring. What is destructuring **good** for?

## Use Cases for Destructuring

There are many situations where destructuring comes in handy. Here's some of the most common ones. Whenever you have a method that returns an object, destructuring makes it much terser to interact with.

```js
function getCoords () {
  return {
    x: 10,
    y: 22
  }
}
var {x, y} = getCoords()
console.log(x)
// <- 10
console.log(y)
// <- 22
```

A similar use case but in the opposite direction is being able to define default options when you have a method with a bunch of options that need default values. This is particularly interesting as an alternative to named parameters in other languages like Python and C#.

```js
function random ({ min=1, max=300 }) {
  return Math.floor(Math.random() * (max - min)) + min
}
console.log(random({}))
// <- 174
console.log(random({max: 24}))
// <- 18
```

If you wanted to make the options object _entirely optional_ you could change the syntax to the following.

```js
function random ({ min=1, max=300 }<mark> = {}</mark>) {
  return Math.floor(Math.random() * (max - min)) + min
}
console.log(random())
// <- 133
```

A great fit for destructuring are things like regular expressions, where you would just love to name parameters without having to resort to index numbers. Here's an example parsing a URL with a random `RegExp` [I got on StackOverflow][1].

```js
function getUrlParts (url) {
  var magic = /^(https?):\/\/(ponyfoo\.com)(\/articles\/([a-z0-9-]+))$/
  return magic.exec(url)
}
var parts = getUrlParts('http://ponyfoo.com/articles/es6-destructuring-in-depth')
var [,protocol,host,pathname,slug] = parts
console.log(protocol)
// <- 'http'
console.log(host)
// <- 'ponyfoo.com'
console.log(pathname)
// <- '/articles/es6-destructuring-in-depth'
console.log(slug)
// <- 'es6-destructuring-in-depth'
```

### Special Case: `import` Statements

Even though `import` statements don't follow destructuring rules, they behave a bit similarly. This is probably the _"destructuring-like"_ use case I find myself using the most, even though it's not actually destructuring. Whenever you're writing module `import` statements, you can pull just what you need from a module's public API. An example using [`contra`][2]:

```js
import {series, concurrent, map } from 'contra'
series(tasks, done)
concurrent(tasks, done)
map(items, mapper, done)
```

Note that, however, `import` statements have a different syntax. When compared against destructuring, none of the following `import` statements will work.

- Use defaults values such as `import {series = noop} from 'contra'`
- "Deep" destructuring style like `import {map: { series }} from 'contra'`
- Aliasing syntax `import {map: mapAsync} from 'contra'`

The main reason for these limitations is that the `import` statement brings in a _binding_, and not a reference or a value. This is an important differentiation that we'll explore more in depth in a future article about ES6 modules.

> I'll keep posting about ES6 & ES7 features every day, so make sure to subscribe if you want to know more!

<sub><mark>\*</mark> How about we visit string interpolation tomorrow?</sub>  
<sub><mark>\*\*</mark>We'll leave arrow functions for monday!</sub>

[1]: http://stackoverflow.com/a/27755/389745 "Getting parts of a URL on StackOverflow"
[2]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names "Computed Property Names – MDN"
