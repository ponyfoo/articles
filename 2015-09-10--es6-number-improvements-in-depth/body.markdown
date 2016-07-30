# `Number` Improvements in ES6

There's a number of changes coming to `Number` in ES6 _-- see what I did there?_ First off, let's raise the curtain with a summary of the features we'll be talking about. We'll go over all of the following changes to `Number` today.

- [Binary and Octal Literals](#binary-and-octal-literals) _-- using `0b` and `0o`_
- [`Number.isNaN`](#numberisnan)
- [`Number.isFinite`](#numberisfinite)
- [`Number.parseInt`](#numberparseint)
- [`Number.parseFloat`](#numberparsefloat)
- [`Number.isInteger`](#numberisinteger)
- [`Number.EPSILON`](#numberepsilon)
- [`Number.MAX_SAFE_INTEGER`](#numbermax-safe-integer)
- [`Number.MIN_SAFE_INTEGER`](#numbermin-safe-integer)
- [`Number.isSafeInteger`](#numberissafeinteger)

![The curtain is raising...][1]

## Binary and Octal Literals

Before ES6, your best bet when it comes to binary representation of integers was to just pass them to `parseInt` with a radix of `2`.

```js
parseInt('101', <mark>2</mark>)
// <- 5
```

In ES6 you could also use the `0b` prefix to represent binary integer literals. You could also use `0B` but I suggest you stick with the lower-case option.

```js
console.log(<mark>0b</mark>001)
// <- 1
console.log(0b010)
// <- 2
console.log(0b011)
// <- 3
console.log(0b100)
// <- 4
```

Same goes for octal literals. In ES3, `parseInt` interpreted strings of digits starting with a `0` as an octal value. That meant things got weird quickly when you forgot to specify a radix of `10` _-- and that soon became a best practice._

```js
parseInt('01')
// <- 1
parseInt('08')
// <- 0
parseInt('8')
// <- 8
```

When ES5 came around, it got rid of the octal interpretation in `parseInt` -- although **it's still recommended** you specify a `radix` for backwards compatibility purposes. If you actually wanted octal, you could get those using a radix of `8`, anyways.

```js
parseInt('100', '8')
// <- 64
```

When it comes to ES6, you can now use the `0o` prefix for octal literals. You could also use `0O`, but that's going to look very confusing in some typefaces, so I suggest you stick with the `0o` notation.

```js
console.log(0o010)
// <- 8
console.log(0o100)
// <- 64
```

Keep in mind that octal literals aren't actually going to crop up everywhere in your front-end applications anytime soon, so you shouldn't worry too much about the seemingly odd choice _(font clarity wise)_ of a `0o` prefix. Besides, most of us use editors that have no trouble at all differentiating between `0o`, `0O`, `00`, `OO`, and `oo`.

![Those characters render just fine in Sublime Text 3][2]

If you're now perplexed and left wondering _"what about hexadecimal?"_, don't you worry, those were already part of the language in ES5, and you can still use them. The prefix for literal _hexadecimal_ notation is either `0x`, or `0X`.

```js
console.log(0x0ff)
// <- 255
console.log(0xf00)
// <- 3840
```

Enough with number literals, let's talk about something else. The first four additions to `Number` that we'll be discussing -- `Number.isNaN`, `Number.isFinite`, `Number.parseInt`, and `Number.parseFloat` -- already existed in ES5, but in the global namespace. In addition, the methods in `Number` are slightly different in that they don't coerce non-numeric values into numbers before producing a result.

## `Number.isNaN`

This method is almost identical to ES5 global [`isNaN`][3] method. `Number.isNaN` returns whether the provided `value` equals `NaN`. This is a very different question from _"is this not a number?"_.

The snippet shown below quickly shows that anything that's not `NaN` when passed to `Number.isNaN` will return `false`, while passing `NaN` into it will yield `true`.

```js
Number.isNaN(123)
// <- false, integers are not NaN
Number.isNaN(Infinity)
// <- false, Infinity is not NaN
Number.isNaN(<mark>'ponyfoo'</mark>)
// <- <mark>false</mark>, 'ponyfoo' is not NaN
Number.isNaN(<mark>NaN</mark>)
// <- true, NaN is NaN
Number.isNaN(<mark>'pony'/'foo'</mark>)
// <- <mark>true</mark>, 'pony'/'foo' is NaN, NaN is NaN
```

The ES5 `global.isNaN` method, in contrast, casts non-numeric values passed to it _before evaluating them against `NaN`_. That produces significantly different results. The example below produces incosistent results because, unlike `Number.isNaN`, `isNaN` casts the `value` passed to it through `Number` first.

```js
isNaN('ponyfoo')
// <- <mark>true</mark>, because Number('ponyfoo') is NaN
isNaN(new Date())
// <- true
```

While `Number.isNaN` is more precise than its global `isNaN` counterpart because it doesn't incur in casting, it's still going to confuse people **because reasons.**

1. `global.isNaN` casts input through `Number(value)` before comparison
2. `Number.isNaN` _doesn't_
3. Neither `Number.isNaN` nor `global.isNaN` answer the _"is this not a number?"_ question
4. They answer whether `value` _-- or `Number(value)` --_ is `NaN`

In most cases, what you actually want is to know whether a value identifies as a number _-- `typeof NaN === 'number'` --_ and _is_ a number. The method below does just that. Note that it'd work with both `global.isNaN` and `Number.isNaN` due to type checking. Everything that reports a `typeof` value of `'number'` is a number, except `NaN`, so we _weed those out_ to avoid false positives!

```js
function isNumber (value) {
  return typeof value === 'number' && !Number.isNaN(value)
}
```

You can use that method to figure out whether anything is **an actual number** or not. Here's some examples of what constitutes actual JavaScript numbers or not.

```js
isNumber(1)
// <- true
isNumber(Infinity)
// <- true
isNumber(NaN)
// <- false
isNumber('ponyfoo')
// <- false
isNumber(new Date())
// <- false
```

Speaking of `isNumber`, isn't there something like that in the language already? _Sort of._

## `Number.isFinite`

The _rarely-advertised_ `isFinite` method has been available since ES3 and it returns whether the provided `value` **matches none of**: `Infinity`, `-Infinity`, and `NaN`.

Want to take a guess about the difference between `global.isFinite` and `Number.isFinite`?

> Correct! the `global.isFinite` method coerces values through `Number(value)`, while `Number.isFinite` doesn't. Here are a few examples using `global.isFinite`. This means that values that can be coerced into _non-`NaN`_ numbers will be considered finite numbers by `global.isNumber` _-- even though they're aren't actually numbers!_

In most cases `isFinite` will be good enough, just like `isNaN`, but when it comes to non-numeric values it'll start acting up and producing unexpected results due to its `value` coercion into numbers.

```js
isFinite(NaN)
// <- false
isFinite(Infinity)
// <- false
isFinite(-Infinity)
// <- false
isFinite(<mark>null</mark>)
// <- <mark>true</mark>, because Number(null) is 0
isFinite('10')
// <- true, because Number('10') is 10
```

Using `Number.isFinite` is just an all-around safer bet as it doesn't incur in unwanted casting. You could always do `Number.isFinite(Number(value))` if you did want the `value` to be casted into its numeric representation.

```js
Number.isFinite(NaN)
// <- false
Number.isFinite(Infinity)
// <- false
Number.isFinite(-Infinity)
// <- false
Number.isFinite(<mark>null</mark>)
// <- <mark>false</mark>
Number.isFinite(0)
// <- true
```

Once again, the discrepancy doesn't do any good to the language, but `Number.isFinite` is consistently more useful than `isFinite`. Creating a polyfill for the `Number.isFinite` version is mostly a matter of type-checking.

```js
Number.isFinite = function (value) {
  return typeof value === 'number' && isFinite(value)
}
```

## `Number.parseInt`

This method works the same as `parseInt`. In fact, it is the same.

```js
console.log(Number.parseInt === parseInt)
// <- true
```

The `parseInt` method keeps producing inconsistencies, though -- even if it didn't even change, **that's the problem**. Before ES6, `parseInt` had support for hexadecimal literal notation in strings. Specifying the `radix` is not even necessary, `parseInt` infers that based on the `0x` prefix.

```js
parseInt('0xf00')
// <- 3840
parseInt('0xf00', <mark>16</mark>)
// <- 3840
```

If you hardcoded another `radix`, _-- and this is **yet another reason** for doing so --_ `parseInt` would bail after the first non-digit character.

```js
parseInt('0xf00', <mark>10</mark>)
// <- 0
parseInt('5xf00', 10)
// <- <mark>5</mark>, illustrating there's no special treatment here
```

So far, it's all good. Why wouldn't I want `parseInt` to drop `0x` from hexadecimal strings? It sounds good, although you may argue that **that's doing too much**, and you'd be _probably right_.

The aggravating issue, however, is that `parseInt` hasn't changed at all. Therefore, binary and octal literal notation in strings won't work.

```js
parseInt('0b011')
// <- 0
parseInt('0b011', 2)
// <- 0
parseInt('0o800')
// <- 0
parseInt('0o800', 8)
// <- 0
```

It'll be up to you to get rid of the prefix before `parseInt`. Remember to _hard-code_ the `radix`, though!

```js
parseInt('0b011'<mark>.slice(2)</mark>, 2)
// <- 3
parseInt('0o110'<mark>.slice(2)</mark>, 8)
// <- 72
```

What's _even weirder_ is that the `Number` method is **perfectly able to cast** these strings into the correct numbers.

```js
Number('0b011')
// <- 3
Number('0o110')
// <- 72
```

I'm not sure what drove them to keep `Number.parseInt` identical to `parseInt`. If it were up to me, I would've made it different so that it worked just like `Number` -- which _is_ able to **coerce octal and binary** number literal strings into the appropriate _base ten_ numbers.

It might be that this was a more involved _"fork"_ of `parseInt` than just _"not coercing `input` into a numeric representation"_ as we observed in `Number.isNaN` and `Number.isFinite`, but I'm just guessing here.

## `Number.parseFloat`

Just like `parseInt`, `parseFloat` was just added to `Number` without any modifications whatsoever.

```js
Number.parseFloat === parseFloat
// <- true
```

In this case, however, `parseFloat` already didn't have any special behavior with regard to hexadecimal literal strings, meaning that this is in fact the only method that won't introduce any confusion, other than it being ported over to `Number` for _completeness' sake_.

## `Number.isInteger`

This is a new method coming in ES6. It returns `true` if the provided `value` is **a finite number** that _doesn't have a decimal part_.

```js
console.log(Number.isInteger(Infinity))
// <- false
console.log(Number.isInteger(-Infinity))
// <- false
console.log(Number.isInteger(NaN))
// <- false
console.log(Number.isInteger(null))
// <- false
console.log(Number.isInteger(0))
// <- true
console.log(Number.isInteger(-10))
// <- true
console.log(Number.isInteger(10.3))
// <- false
```

If you want to look at a a polyfill for `isInteger`, you might want to consider the following code snippet. The modulus operator returns the remainder of dividing the same operands _-- effectively: the decimal part._ If that's `0`, that means the number is an integer.

```js
Number.isInteger = function (value) {
  return <mark>Number.isFinite(value)</mark> && <mark>value % 1</mark> === 0
}
```

Floating point arithmetic is well-documented as being kind of ridiculous. What is this `Number.EPSILON` thing?

## `Number.EPSILON`

Let me answer that question with a piece of code.

```js
Number.EPSILON
// <- <mark>2.220446049250313e-16</mark>, wait what?
Number.EPSILON.toFixed(20)
// <- <mark>'0.00000000000000022204'</mark>, got it
```

Ok, so `Number.EPSILON` is **a terribly small number**. What good is it for? Remember that thing about how floating point sum makes no sense? Here's the canonical example, I'm sure you remember it -- _Yeah, I know._

```js
0.1 + 0.2
// <- <mark>0.30000000000000004</mark>
0.1 + 0.2 === 0.3
// <- <mark>false</mark>
```

Let's try that one more time.

```js
0.1 + 0.2 - 0.3
// <- <mark>5.551115123125783e-17</mark>, what the hell?
5.551115123125783e-17.toFixed(20)
// <- <mark>'0.00000000000000005551'</mark>, got it
```

_So what?_ You can use `Number.EPSILON` to figure out whether the difference is small enough to fall under the _"floating point arithmetic is ridiculous and the difference is negligible"_ category.

```js
5.551115123125783e-17 < Number.EPSILON
// <- <mark>true</mark>
```

Can we trust that? Well, `0.00000000000000005551` is indeed smaller than `0.00000000000000022204`. What do you mean you don't trust me? Here they are side by side.

```js
0.00000000000000005551
0.00000000000000022204
```

See? `Number.EPSILON` is _larger_ than the difference. We can use `Number.EPSILON` as an acceptable margin of error due to floating point arithmetic rounding operations.

Thus, the following piece of code figures out whether the result of a floating point operation is within the expected margin of error. We use [`Math.abs`][4] because that way the order of `left` and `right` won't matter. In other words, `withinErrorMargin(left, right)` will produce the same result as `withinErrorMargin(right, left)`.

```js
function withinErrorMargin (left, right) {
  return <mark>Math.abs(left - right)</mark> < Number.EPSILON
}
withinErrorMargin(0.1 + 0.2, 0.3)
// <- true
withinErrorMargin(0.2 + 0.2, 0.3)
// <- false
```

> While, yes, you **could** do this, it's probably unnecessarily complicated unless you have to deal with very low-level mathematics. You'll be better of pulling a library like [`mathjs`][5] into your project.

Last but not least, there's the other weird aspect of number representation in JavaScript. **Not every integer** can be represented _precisely_, either.

## `Number.MAX_SAFE_INTEGER`

This is the largest integer that can be safely and precisely represented in JavaScript, or any language that represents integers using _floating point_ as specified by [IEEE-754][6] for that matter. The code below show just how large that number is. If we need to be able to deal with numbers larger than that, then I would once again point you to [`mathjs`][5], or maybe **try another language** for your computationally intensive services.

```js
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1
// <- true
Number.MAX_SAFE_INTEGER === 9007199254740991
// <- true
```

And you know what they say -- _If there's a maximum..._

## `Number.MIN_SAFE_INTEGER`

Right, nobody says that. However, there's a `Number.MIN_SAFE_INTEGER` regardless, and it's the negative value of `Number.MAX_SAFE_INTEGER`.

```js
Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER
// <- true
Number.MIN_SAFE_INTEGER === -9007199254740991
// <- true
```

How exactly can you leverage these two constants, I hear you say? In the case of the overflow problem, you don't have to implement your own `withinErrorMargin` method like you had to do for floating point precision. Instead, a `Number.isSafeInteger` is provided to you.

## `Number.isSafeInteger`

This method returns `true` for any integer in the `[MIN_SAFE_INTEGER, MAX_SAFE_INTEGER]` range. There's no type coercion here either. The input must be numeric, an integer, and within the aforementioned bounds in order for the method to return `true`. Here's a quite comprehensive set of examples for you to stare at.

```js
Number.isSafeInteger('a')
// <- false
Number.isSafeInteger(null)
// <- false
Number.isSafeInteger(NaN)
// <- false
Number.isSafeInteger(Infinity)
// <- false
Number.isSafeInteger(-Infinity)
// <- false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1)
// <- <mark>false</mark>
Number.isSafeInteger(Number.MIN_SAFE_INTEGER)
// <- <mark>true</mark>
Number.isSafeInteger(1)
// <- true
Number.isSafeInteger(1.2)
// <- false
Number.isSafeInteger(Number.MAX_SAFE_INTEGER)
// <- <mark>true</mark>
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)
// <- <mark>false</mark>
```

As [Dr. Axel Rauschmayer points out][8] in his article about ES6 numbers, when we want to verify if the result of an operation is within bounds, we must verify not only the result but also both operands. The reason for that is one _(or both)_ of the operands may be out of bounds, while the result is _"safe"_ **(but incorrect)**. Similarly, the result may be out of bounds itself, so checking all of `left`, `right`, and the result of `left op right` is necessary to verify that we can indeed trust the result.

In all of the examples below, **the result is incorrect**. Here's the first example, where both operands are safe even though the result is not.

```js
Number.isSafeInteger(9007199254740000)
// <- true
Number.isSafeInteger(993)
// <- true
Number.isSafeInteger(9007199254740000 + 993)
// <- false
9007199254740000 + 993
// <- <mark>9007199254740992</mark>, should be 9007199254740993
```

In this example one of the operands wasn't within range, so we can't trust the result to be accurate.

```js
Number.isSafeInteger(9007199254740993)
// <- false
Number.isSafeInteger(990)
// <- true
Number.isSafeInteger(9007199254740993 + 990)
// <- false
9007199254740993 + 990
// <-  <mark>9007199254741982</mark>, should be 9007199254741983
```

Note that in the example above, a subtraction would produce a result within bounds, and that result would _also_ be inaccurate.

```js
Number.isSafeInteger(9007199254740993)
// <- false
Number.isSafeInteger(990)
// <- true
Number.isSafeInteger(9007199254740993 - 990)
// <- true
9007199254740993 - 990
// <-  <mark>9007199254740002</mark>, should be 9007199254740003
```

It doesn't take a genius to figure out the case where both operands are out of bounds but the result is _deemed "safe"_, even though the result is incorrect.

```js
Number.isSafeInteger(9007199254740993)
// <- false
Number.isSafeInteger(9007199254740995)
// <- false
Number.isSafeInteger(9007199254740993 - 9007199254740995)
// <- true
9007199254740993 - 9007199254740995
// <- <mark>-4</mark>, should be -2
```

Thus, as you can see, we can conclude that the only safe way to assert whether an operation is correct is with a method like the one below. If we can't ascertain that the operation and both its operands are within bounds, then the result may be inaccurate, and that's a problem. It's best to `throw` in those situations and have a way to error-correct, but that's specific to your programs. The important part is to actually catch these kinds of difficult bugs to deal with.

```js
function trusty (left, right, result) {
  if (
    Number.isSafeInteger(left) &&
    Number.isSafeInteger(right) &&
    Number.isSafeInteger(result)
  ) {
    return result
  }
  throw new RangeError('Operation cannot be trusted!')
}
```

You could then use that every step of the way to ensure all operands remain safely within bounds. I've highlighted the unsafe values in the examples below. Note that even though none of the operations in my examples return accurate results, certain operations and numbers _may do so_ even when operands are out of bounds. The problem is that that **can't be guaranteed** _-- therefore the operation can't be trusted._

```js
trusty(9007199254740000, 993, <mark>9007199254740000 + 993</mark>)
// <- RangeError: Operation cannot be trusted!
trusty(<mark>9007199254740993</mark>, 990, <mark>9007199254740993 + 990</mark>)
// <- RangeError: Operation cannot be trusted!
trusty(<mark>9007199254740993</mark>, 990, 9007199254740993 - 990)
// <- RangeError: Operation cannot be trusted!
trusty(<mark>9007199254740993</mark>, <mark>9007199254740995</mark>, 9007199254740993 - 9007199254740995)
// <- RangeError: Operation cannot be trusted!
trusty(1, 2, 3)
// <- 3
```

I don't think I want to write about floating point again for a while. _Time to scrub myself up._

# Conclusions

While some of the hacks to guard against rounding errors and overflow safety are _nice to have_, they don't attack the heart of the problem: _math with the [IEEE-754][6] standard is hard._

These days JavaScript runs on all the things, so it'd be nice if a better standard were to be _implemented alongside [IEEE-754][6]_. Roughly a year ago, _Douglas Crockford_ came up with [DEC64][7], but opinions on its merits range from _"this is genius!"_ to _"this is the work of a madman"_ -- I guess that's the norm when it comes to most of the stuff Crockford publishes, though.

It'd be nice, to eventually see the day where JavaScript is able to _precisely_ compute decimal arithmetic as well as able to represent large integers safely. That day we'll probably have something **alongside** _floating point_.

[1]: https://i.imgur.com/GrB4w7R.jpg
[2]: https://i.imgur.com/buOiD1I.png
[3]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/isNaN "Global isNaN() on MDN"
[4]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/abs "Math.abs() on MDN"
[5]: https://github.com/josdejong/mathjs "josdejong/mathjs on GitHub"
[6]: https://en.wikipedia.org/wiki/IEEE_floating_point "IEEE Floating Point Standard on Wikipedia"
[7]: http://dec64.com/ "DEC64 number type"
[8]: http://www.2ality.com/2015/04/numbers-math-es6.html "New number and Math features in ES6 – 2ality.com"
