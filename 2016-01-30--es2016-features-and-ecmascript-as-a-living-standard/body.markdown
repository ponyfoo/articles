# `Array.prototype.includes`

Originally, this method was going to be named `Array.prototype.contains`, but that [would've proven problematic][3] as it would've clashed with popular library `MooTools` -- whose implementation was incompatible with ECMA's, potentially resulting in many broken websites once the feature was implemented according to the standard.

> This proposal was formerly for `Array.prototype.contains`, but that name [is not web-compatible][4]. Per the November 2014 TC39 meeting, the name of both `String.prototype.contains` and `Array.prototype.contains` was changed to `includes` to dodge that bullet.

The `.includes` method returns whether the provided reference value is included in the array or not.

```js
['a', 'b', 'c'].includes('c');
// <- true
[{}, {}].includes({}); // strictly a reference comparison
// <- false
var a = {};
[{}, a].includes(a);
// <- true
```

You can also specify a `fromIndex` as the second parameter, and the search will start at that position in the array. When this value isn't provided, `0` is assumed and the whole array is searched.

```js
['a', 'b', 'c', 'd'].includes('b'<mark>, 2</mark>);
// <- false
```

Even though `NaN !== NaN`, `.includes` uses the [`SameValueZero` comparison algorithm][5] where `NaN` is equivalent to itself.

```js
[NaN].includes(NaN);
// <- true
```

There isn't much else to say about this method.

# Exponentiation operator -- `**`

The exponentiation operator -- or `a ** b` -- is the syntactic equivalent to doing `Math.pow(a, b)`. It'll work similarly to the [exponentiation operator in Python][2].

```js
1 ** 2 === Math.pow(1, 2)
// <- true
```

Just like with any other operators, it's possible to mix exponentiation with assignment, as show below.

```js
var a = 2;
a **= 3; // equivalent to <mark>a = Math.pow(a, 3)</mark>
console.log(a);
// <- 8
```

# ECMAScript as a Living Standard

In an article shared this weekend, Axel Rauschmayer has detailed [how the ECMAScript release process will work][1] from now on. In this sense, the ECMAScript standarization process will behave more like HTML and CSS ones, where new features are standarized as consensus is reached, rather than clean-cut _-- or "limited by" --_ by version numbers.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/nzgb">@nzgb</a> <a href="https://twitter.com/rauschma">@rauschma</a> Apparently. The Epoch of Unversioned Web.</p>&mdash; Ingvar Stepanyan (@RReverser) <a href="https://twitter.com/RReverser/status/693544959618760704">January 30, 2016</a></blockquote>

This standarization style is what we've come to know as a living standard. It's something that I think was long overdue when it comes to JavaScript, even though **in practice, implementation was already behaving this way**: browsers already implement much of ES6 and there never was a waiting period until 100% of ES6 is implemented.

Firefox, Chrome, and Edge all offer **over 90% compliancy of the ES6 specification**, and we've been able to use the features they implemented as they came out rather than having to wait until the flip of a switch when their implementation of ES6 was 100% finalized. This can also be connected with what Chris Heilmann frequently brings up when he mentions browser vendors need developers to try out features in order to improve them, but developers need to be able to play with those features in order to try them out.

Personally, I'm pretty excited about JavaScript becoming a living standard, and so should you! We've seen how much CSS improved as a result, so this is another step in the right direction by the ECMAScript council. Yay, JavaScript!

[1]: http://www.2ality.com/2016/01/ecmascript-2016.html "The final feature set of ECMAScript 2016 (ES7) on 2ality.com"
[2]: http://www.pythonforbeginners.com/basics/python-operators "Python Operators – Python for Beginners"
[3]: https://github.com/tc39/Array.prototype.includes/tree/b6671aec098db241ab2d27d7bc182cc8a074edef "tc39/Array.prototype.includes on GitHub"
[4]: http://esdiscuss.org/topic/having-a-non-enumerable-array-prototype-contains-may-not-be-web-compatible "Having a non-enumerable Array.prototype.contains may not be web-compatible"
[5]:  http://www.ecma-international.org/ecma-262/6.0/#sec-samevaluezero "SameValueZero algorithm in ECMAScript specification"
