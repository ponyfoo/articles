# `Math` Additions in ES6

There's _heaps_ of additions to `Math` in ES6. Just like you're used to, these are **static** methods on the `Math` built-in. Some of these methods were specifically engineered towards making it easier to compile C into JavaScript, and you may never come across a need for them in day-to-day development -- particularly not when it comes to front-end development. Other methods are complements to the existing rounding, exponentiation, and trigonometry API surface.

Below is a full list of methods added to `Math`. They are grouped by functionality and sorted by relevance.

- **Utility**
  - [`Math.sign`](#mathsign) -- sign function of a number
  - [`Math.trunc`](#mathtrunc) -- integer part of a number
- **Exponentiation and Logarithmic**
  - [`Math.cbrt`](#mathcbrt) -- cubic root of value, or <code class='md-code md-code-inline'>∛‾value</code>
  - [`Math.expm1`](#mathexpm1) -- `e` to the `value` minus `1`, or <code class='md-code md-code-inline'>e<sup>value</sup> - 1</code>
  - [`Math.log1p`](#mathlog1p) -- natural logarithm of `value + 1`, or <code class='md-code md-code-inline'><em>ln</em>(value + 1)</code>
  - [`Math.log10`](#mathlog10) -- base 10 logarithm of `value`, or <code class='md-code md-code-inline'><em>log</em><sub>10</sub>(value)</code>
  - [`Math.log2`](#mathlog2) -- base 2 logarithm of `value`, or <code class='md-code md-code-inline'><em>log</em><sub>2</sub>(value)</code>
- **Trigonometry**
  - [`Math.sinh`](#mathsinh) -- hyperbolic sine of a number
  - [`Math.cosh`](#mathcosh) -- hyperbolic cosine of a number
  - [`Math.tanh`](#mathtanh) -- hyperbolic tangent of a number
  - [`Math.asinh`](#mathasinh) -- hyperbolic arc-sine of a number
  - [`Math.acosh`](#mathacosh) -- hyperbolic arc-cosine of a number
  - [`Math.atanh`](#mathatanh) -- hyperbolic arc-tangent of a number
  - [`Math.hypot`](#mathhypot) -- square root of the sum of squares
- **Bitwise**
  - [`Math.clz32`](#mathclz32) -- leading zero bits in the 32-bit representation of a number
- **Compile-to-JavaScript**
  - [`Math.imul`](#mathimul) -- _C-like_ 32-bit multiplication
  - [`Math.fround`](#mathfround) -- nearest single-precision float representation of a number

Let's get right into it.

# `Math.sign`

Many languages have a [`Math.Sign`][3] method _(or equivalent)_ that returns a vector like `-1`, `0`, or `1`, depending on the sign of the provided input. Surely then, you would think JavaScript's `Math.sign` method does the same. Well, sort of. The JavaScript flavor of this method has two more alternatives: `-0`, and `NaN`.

```js
Math.sign(1)
// <- 1
Math.sign(0)
// <- 0
Math.sign(-0)
// <- <mark>-0</mark>
Math.sign(-30)
// <- -1
Math.sign(NaN)
// <- <mark>NaN</mark>
Math.sign('foo')
// <- NaN, because Number('foo') is NaN
Math.sign('0')
// <- <mark>0</mark>, because Number('0') is 0
Math.sign('-1')
// <- <mark>-1</mark>, because Number('-1') is -1
```

This is just one of those methods. It **grinds my gears.** After all the trouble we went through to document how methods ported over to `Number`, such as `Number.isNaN`, don't indulge in [unnecessary type coercion][2], why is it that `Math.sign` _does_ coerce its input? I have no idea. Most of the methods in `Math` share this trait, though. The methods that were added to `Number` don't.

!['What really grinds my gears', on Family Guy][1]

Sure, we're not a statically typed language, we dislike throwing exceptions, and we're [fault tolerant][5] -- after all, this is one of the founding languages of the web. But was not coercing everything into a `Number` too much to ask? Couldn't we just return `NaN` for _non-numeric_ values?

I'd love for us to get over implicit casting, but it seems we're _not quite there yet_ for the time being.

# `Math.trunc`

One of the oddities in `Math` methods is how abruptly they were named. It's like they were trying to save keystrokes or something. After all, it's not like we stopped adding _super-precise_ method names like [`Object.getOwnPropertySymbols()`][4]. Why `trunc` instead of `truncate`, then? Who knows.

Anyways, `Math.trunc` is a simple alternative to `Math.floor` and `Math.ceil` where we simply discard the decimal part of a number. Once again, the input is coerced into a numeric value through `Number(value)`.

```js
Math.trunc(12.34567)
// <- 12
Math.trunc(-13.58)
// <- -13
Math.trunc(-0.1234)
// <- <mark>-0</mark>
Math.trunc(NaN)
// <- NaN
Math.trunc('foo')
// <- <mark>NaN</mark>, because Number('foo') is NaN
Math.trunc('123.456')
// <- <mark>123</mark>, because Number('123.456') is 123.456
```

While it still coerces any values into numbers, at least it stayed consistent with `Math.floor` and `Math.ceil`, enough that you could use them to create a simple polyfill for `Math.trunc`.

```js
Math.trunc = function truncate (value) {
  return value > 0 ? Math.floor(value) : Math.ceil(value)
}
```

Another apt example of how _"succintly"_ `Math` methods have been named in ES6 can be found in `Math.cbrt` _-- although this one matches the pre-existing `Math.sqrt` method, to be fair._

# `Math.cbrt`

As hinted above, `Math.cbrt` is short for _"cubic root"_. The examples below show the sorts of output it produces.

```js
Math.cbrt(-1)
// <- -1
Math.cbrt(3)
// <- 1.4422495703074083
Math.cbrt(8)
// <- 2
Math.cbrt(27)
// <- 3
```

Not much explaining to do here. Note that this method _also_ coerces non-numerical values into numbers.

```js
Math.cbrt('8')
// <- <mark>2</mark>, because Number('8') is 8
Math.cbrt('ponyfoo')
// <- <mark>NaN</mark>, because Number('ponyfoo') is NaN
```

Let's move onto something else.

# `Math.expm1`

This operation is the result of computing `e` to the `value` minus `1`. In JavaScript, the `e` constant is defined as `Math.E`. The method below is a rough equivalent of `Math.expm1`.

```js
function expm1 (value) {
  return Math.pow(Math.E, value) - 1
}
```

The <code class='md-code md-code-inline'>e<sup>value</sup></code> operation can be expressed as `Math.exp(value)` as well.

```js
function expm1 (value) {
  return Math.exp(value) - 1
}
```

Note that this method has higher precision than merely doing `Math.exp(value) - 1`, and should be the preferred alternative.

```js
expm1(1e-20)
// <- 0
Math.expm1(1e-20)
// <- 1e-20
expm1(1e-10)
// <- 1.000000082740371e-10
Math.expm1(1e-10)
// <- 1.00000000005e-10
```

The inverse function of `Math.expm1` is `Math.log1p`.

# `Math.log1p`

This is the natural logarithm of `value` plus `1`, -- <code class='md-code md-code-inline'><em>ln</em>(value + 1)</code> -- and the inverse function of `Math.expm1`. The base `e` logarithm of a number can be expressed as `Math.log` in JavaScript.

```js
function log1p (value) {
 return Math.log(value + 1)
}
```

This method is **more precise** than executing the `Math.log(value + 1)` operation by hand, just like the [`Math.expm1`](#mathexpm1) case.

```js
log1p(1.00000000005e-10)
// <- 1.000000082690371e-10
Math.log1p(1.00000000005e-10)
// <- <mark>1e-10</mark>, exactly the inverse of Math.expm1(1e-10)
```

Next up is `Math.log10`.

# `Math.log10`

Base ten logarithm of a number -- <code class='md-code md-code-inline'><em>log</em><sub>10</sub>(value)</code>.

```js
Math.log10(1000)
// <- 3
```

You could polyfill `Math.log10` using the `Math.LN10` constant.

```js
function log10 (value) {
  return Math.log(x) / Math.LN10
}
```

And then there's `Math.log2`.

# `Math.log2`

Base two logarithm of a number -- <code class='md-code md-code-inline'><em>log</em><sub>2</sub>(value)</code>.

```js
Math.log2(1024)
// <- 10
```

You could polyfill `Math.log2` using the `Math.LN2` constant.

```js
function log2 (value) {
  return Math.log(x) / Math.LN2
}
```

Note that the polyfilled version won't be as precise as `Math.log2` in some cases. Remember that the `<<` operator means [_"bitwise left shift"_][10].

```js
Math.log2(1 << 29)
// <- 29
log2(1 << 29)
// <- 29.000000000000004
```

Naturally, you could use `Math.round` or [`Number.EPSILON`][6] to get around rounding issues.

# `Math.sinh`

Returns the hyperbolic sine of `value`.

# `Math.cosh`

Returns the hyperbolic cosine of `value`.

# `Math.tanh`

Returns the hyperbolic tangent of `value`.

# `Math.asinh`

Returns the hyperbolic arc-sine of `value`.

# `Math.acosh`

Returns the hyperbolic arc-cosine of `value`.

# `Math.atanh`

Returns the hyperbolic arc-tangent of `value`.

# `Math.hypot`

Returns the square root of the sum of the squares of the arguments.

```js
Math.hypot(1, 2, 3)
// <- 3.741657386773941
```

We could polyfill `Math.hypot` by doing the operations manually. We can use `Math.sqrt` to compute the square root and [`Array.prototype.reduce`][9] combined with the [spread operator][7] to sum the squares. I'll throw in [an arrow function][8] for good measure!

```js
function hypot (<mark>...</mark>values) {
  return Math.sqrt(values<mark>.reduce</mark>((sum, value) => sum + <mark>value * value</mark>, 0))
}
```

Surprisingly, our handmade method is more precise than the native one _(at least on Chrome 45)_ for **this case in particular**.

```js
Math.hypot(1, 2, 3)
// <- 3.741657386773941
hypot(1, 2, 3)
// <- <mark>3.7416573867739413</mark>
```

And now for the **"_really_ fun"** methods!

# `Math.clz32`

Definitely not immediately obvious, but the name for this method is an acronym for _"count leading zero bits in 32-bit binary representations of a number"_. Remember that the `<<` operator means [_"bitwise left shift"_][10], and thus...

```js
Math.clz32(0)
// <- 32
Math.clz32(1)
// <- 31
Math.clz32(1 << 1)
// <- 30
Math.clz32(1 << 2)
// <- 29
Math.clz32(1 << 29)
// <- 2
```

Cool, and also probably the last time you're going to see that method in use for the foreseeable future. For completeness' sake, I'll add a sentence about the pair of methods that were added mostly to aid with `asm.js` compilation of C programs. _I doubt you'll be using these directly, ever._

# `Math.imul`

Returns the result of a C-like 32-bit multiplication.

# `Math.fround`

Rounds `value` to the nearest 32-bit float representation of a number.

# Conclusions

Some nice methods rounding out the `Math` API. It would've been nice to see additions to the tune of more flavors of `Math.random` and similar utilities that end up being implemented by libraries in almost every large-enough application, such as Lodash'es [`_.random`][12] and [`_.shuffle`][13].

That being said, any help towards making [`asm.js`][11] faster and more of a reality are desperately welcome additions to the language.

[1]: https://i.imgur.com/ZQiiZsF.png
[2]: /articles/es6-number-improvements-in-depth "ES6 Number Improvements in Depth on Pony Foo"
[3]: https://msdn.microsoft.com/en-us/library/ywb0xks3(v=vs.110).aspx "C#, for instance"
[4]: /articles/es6-symbols-in-depth "ES6 Symbols in Depth on Pony Foo"
[5]: http://blog.codinghorror.com/javascript-and-html-forgiveness-by-default/ "JavaScript and HTML: Forgiveness by Default"
[6]: /articles/es6-number-improvements-in-depth#numberepsilon "Number.EPSILON, ES6 Number Improvements in Depth on Pony Foo"
[7]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[8]: /articles/es6-arrow-functions-in-depth "ES6 Arrow Functions in Depth on Pony Foo"
[9]: /articles/fun-with-native-arrays "Fun with Native Arrays on Pony Foo"
[10]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Left_shift "Bitwise left shift operator on MDN"
[11]: http://asmjs.org/faq.html "asm.js frequently asked questions"
[12]: https://lodash.com/docs#random "_.random([min=0], [max=1], [floating]) on Lodash documentation"
[13]: https://lodash.com/docs#shuffle "_.shuffle(collection) on Lodash documentation"
