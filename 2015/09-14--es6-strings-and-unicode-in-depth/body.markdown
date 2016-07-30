# Updates to `String`

We've already covered [template literals][1] earlier in the series, and you may recall that those can be used to mix strings and variables to produce string output.

```js
function greet (name) {
  return `hello ${name}!`
}
greet('ponyfoo')
// <- 'hello ponyfoo!'
```

Besides template literals, strings are getting a numbre of new methods come ES6. These can be categorized as string manipulation methods and unicode related methods.

- String Manipulation
  - [`String.prototype.startsWith`](#stringprototypestartswith)
  - [`String.prototype.endsWith`](#stringprototypeendswith)
  - [`String.prototype.includes`](#stringprototypeincludes)
  - [`String.prototype.repeat`](#stringprototyperepeat)
  - [`String.prototype[Symbol.iterator]`](#stringprototype-symboliterator)
- [Unicode](#unicode)
  - [`String.prototype.codePointAt`](#stringprototypecodepointat)
  - [`String.fromCodePoint`](#stringfromcodepoint`)
  - [`String.prototype.normalize`](#stringprototypenormalize)

We'll begin with the string manipulation methods and then we'll take a look at the unicode related ones.

# `String.prototype.startsWith`

A very common question in our code is _"does this string start with this other string?"_. In ES5 we'd ask that question using the `.indexOf` method.

```js
'ponyfoo'.indexOf('foo')
// <- 4
'ponyfoo'.indexOf('pony')
// <- 0
'ponyfoo'.indexOf('horse')
// <- -1
```

If you wanted to check if a string started with another one, you'd compare them with `.indexOf` and check whether the _"needle"_ starts at the `0` position -- the beginning of the string.

```js
'ponyfoo'.indexOf('pony') <mark>=== 0</mark>
// <- true
'ponyfoo'.indexOf('foo') === 0
// <- false
'ponyfoo'.indexOf('horse') === 0
// <- false
```

You can now use the more descriptive and terse `.startsWith` method instead.

```js
'ponyfoo'<mark>.startsWith</mark>('pony')
// <- true
'ponyfoo'<mark>.startsWith</mark>('foo')
// <- false
'ponyfoo'<mark>.startsWith</mark>('horse')
// <- false
```

If you wanted to figure out whether a string contains another one starting in a specific location, it would get quite verbose, as you'd need to grab a slice of that string first.

```js
'ponyfoo'<mark>.slice(4)</mark>.indexOf('foo') === 0
// <- true
```

The reason why you can't just ask `=== 4` is that this would give you false negatives when the query is found before reaching that index.

```js
'foo,<mark>foo</mark>'.indexOf('foo') === 4
// <- <mark>false</mark>, because result was <mark>0</mark>
```

Of course, you could use the `startIndex` parameter for `indexOf` to get around that. Note that we're still comparing against `4` in this case, because the string wasn't split into smaller parts.

```js
'foo,foo'.indexOf('foo', <mark>4</mark>) === <mark>4</mark>
// <- true
```

Instead of keeping all of these string searching implementation details in your head and writing code that worries too much about the how and not so much about the what, you could just use `startsWith` passing in the optional `startIndex` parameter as well.

```js
'foo,foo'<mark>.startsWith</mark>('foo', <mark>4</mark>)
// <- true
```

Then again, it's kind of confusing that the method is called `.startsWith` but we're starting at a non-zero index -- that being said it sure beats using `.indexOf` when we actually want a boolean result.

# `String.prototype.endsWith`

This method mirrors `.startsWith` in the same way that `.lastIndexOf` mirrors `.indexOf`. It tells us whether a string ends with another string.

```js
'ponyfoo'.endsWith('foo')
// <- true
'ponyfoo'.endsWith('pony')
// <- false
```

Just like `.startsWith`, we have a position index that indicates where the lookup should end. It defaults to the length of the string.

```js
'ponyfoo'.endsWith('foo', 7)
// <- true
'ponyfoo'.endsWith('pony', 0)
// <- false
'ponyfoo'.endsWith('pony', 4)
// <- true
```

Yet another method that simplifies a specific use case for `.indexOf` is `.includes`.

# `String.prototype.includes`

You can use `.includes` to figure out whether a string contains another one.

```js
'ponyfoo'<mark>.includes</mark>('ny')
// <- true
'ponyfoo'.includes('sf')
// <- false
```

This is equivalent to the ES5 use case of `.indexOf` where we'd compare its results with `-1` to see if the search string was anywhere to be found.

```js
'ponyfoo'<mark>.indexOf</mark>('ny') <mark>!== -1</mark>
// <- true
'ponyfoo'<mark>.indexOf</mark>('sf') <mark>!== -1</mark>
// <- false
```

Naturally you can also pass in a start index where the search should begin.

```js
'ponyfoo'.includes('ny', <mark>3</mark>)
// <- false
'ponyfoo'.includes('ny', <mark>2</mark>)
// <- true
```

Let's move onto something that's not an `.indexOf` replacement.

# `String.prototype.repeat`

This handy method allows you to repeat a string `count` times.

```js
'na'<mark>.repeat</mark>(0)
// <- ''
'na'.repeat(1)
// <- 'na'
'na'.repeat(2)
// <- 'na<mark>na</mark>'
'na'.repeat(5)
// <- 'na<mark>nananana</mark>'
```

The provided `count` should be a positive finite number.

```js
'na'.repeat(Infinity)
// <- RangeError
'na'.repeat(-1)
// <- RangeError
```

Non-numeric values are coerced into numbers.

```js
'na'.repeat('na')
// <- ''
'na'.repeat('3')
// <- 'nanana'
```

Using `NaN` is as good as `0`.

```js
'na'.repeat(NaN)
// <- ''
```

Decimal values are floored.

```js
'na'.repeat(3.9)
// <- <mark>'nanana'</mark>, count was floored to 3
```

Values in the `(-1, 0)` range are rounded to `-0` becase `count` is passed through [`ToInteger`][14], as documented by [the specification][15]. That step in the specification dictates that `count` be casted with a formula like the one below.

```js
function ToInteger (number) {
  return Math.floor(Math.abs(number)) * Math.sign(number)
}
```

The above translates to `-0` for any values in the `(-1, 0)` range. Numbers below that will throw, and numbers above that won't behave surprisingly, as you can only take `Math.floor` into account for positive values.

```js
'na'.repeat(-0.1)
// <- <mark>''</mark>, count was rounded to <mark>-0</mark>
```

A good example use case for `.repeat` may be your typical _"padding"_ method. The method shown below takes a multiline string and pads every line with as many `spaces` as desired.

```js
function pad (text, spaces) {
  return text.split('\n').map(line => ' '<mark>.repeat(spaces)</mark> + line).join('\n')
}
pad('a\nb\nc', 2)
// <- '  a\n  b\n  c'
```

In ES6, strings adhere to the [iterable protocol][2].

# `String.prototype[Symbol.iterator]`

Before ES6, you could access each **code unit** _(we'll define these in a second)_ in a string via indices -- kind of like with arrays. That meant you could loop over **code units** in a string with a `for` loop.

```js
var text = 'foo'
for (let i = 0; i < text.length; i++) {
  console.log(<mark>text[i]</mark>)
  // <- 'f'
  // <- 'o'
  // <- 'o'
}
```

In ES6, you could loop over the **code points** _(not the same as **code units**)_ of a string using a `for..of` loop, because strings are _iterable_.

```js
for (let <mark>codePoint</mark> of 'foo') {
  console.log(codePoint)
  // <- 'f'
  // <- 'o'
  // <- 'o'
}
```

What is this `codePoint` variable? There is _a not-so-subtle distinction_ between **code units** and **code points**. Let's switch protocols and talk about _Unicode_.

# Unicode

JavaScript strings are represented using [_UTF-16 code units_][16]. Each code unit can be used to represent a code point in the `[U+0000, U+FFFF]` range -- also known as the _"basic multilingual plane"_ (BMP). You can represent individual code points in the BMP plane using the `'\u3456'` syntax. You could also represent code units in the `[U+0000, U+0255]` using the `\x00..\xff` notation. For instance, `'\xbb'` represents `'»'`, the `187` character, as you can verify by doing `parseInt('bb', 16)` -- or `String.fromCharCode(187)`.

For code points beyond `U+FFFF`, you'd represent them as a surrogate pair. That is to say, two contiguous code units. For instance, the horse emoji `'🐎'` code point is represented with the `'\ud83d\udc0e'` contiguous code units. In ES6 notation you can also represent code points using the `'\u{1f40e}'` notation _(that example is also the horse emoji)_. Note that the internal representation hasn't changed, so there's **still two code units** behind that code point. In fact, `'\u{1f40e}'.length` evaluates to `2`.

The `'\ud83d\udc0e\ud83d\udc71\u2764'` string found below evaluates to a few emoji.

```js
'\ud83d\udc0e\ud83d\udc71\u2764'
// <- '🐎👱❤'
```

While that string consists of 5 code units, we know that the length should really be three -- as there's only three emoji.

```js
'\ud83d\udc0e\ud83d\udc71\u2764'.length
// <- 5
'🐎👱❤'.length
// <- <mark>5</mark>, still
```

Before ES6, JavaScript didn't make any effort to figure out unicode quirks on your behalf -- you were pretty much on your own when it came to counting cards _(err, code points)_. Take for instance `Object.keys`, still five code units long.

```js
console.log(Object.keys('<mark>🐎👱❤</mark>'))
// <- ['0', '1', '2', <mark>'3'</mark>, <mark>'4'</mark>]
```

If we now go back to our `for` loop, we can observe how this is a problem. We actually wanted `'🐎'`, `'👱'`, `'❤'`, but we didn't get that.

```js
var text = '<mark>🐎👱❤</mark>'
for (let i = 0; i < text.length; i++) {
  console.log(<mark>text[i]</mark>)
  // <- '<mark>?</mark>'
  // <- '<mark>?</mark>'
  // <- '<mark>?</mark>'
  // <- '<mark>?</mark>'
  // <- '❤'
}
```

Instead, we got some weird unicode boxes -- and that's if we were lucky and looking at Firefox.

![Printing some emoji character by character on the Firefox console][5]

That didn't turn out okay. In ES6 we can use the string iterator to go over the code points instead. The iterators produced by the string iterable are aware of this limitation of looping by code units, and so they _yield code points_ instead.

```js
for (let codePoint of '<mark>🐎👱❤</mark>') {
  console.log(<mark>codePoint</mark>)
  // <- '<mark>🐎</mark>'
  // <- '<mark>👱</mark>'
  // <- '❤'
}
```

If we want to measure the length, we'd have trouble with the `.length` property, as we saw earlier. We can use the iterator to **split the string into its code points** _-- as seen in the `for..of` example we just went over._ That means the unicode-aware length of a string equals the length of the array that contains the sequence of code points produced by an iterator. We could use the [spread operator][4] to place the code points in an array, and then pull that array's `.length`.

```js
[<mark>...</mark>'<mark>🐎👱❤</mark>']<mark>.length</mark>
// <- 3
```

Keep in mind that splitting strings into code points isn't enough if you want to be _100% precise_ about string length. Take for instance the [_"combining overline"_ `\u0305`][3] unicode code unit. On its own, this code unit is just an overline.

```js
'\u0305'
// <- ' ̅'
```

When preceded by another code unit, however, they are **combined together** into a single glyph.

```js
'_<mark>\u0305</mark>'
// <- '_̅'
'foo<mark>\u0305</mark>'
// <- 'foo̅'
```

Attempts to figure out the actual length by counting code points prove **insufficient** -- just like using `.length`.

```js
'foo<mark>\u0305</mark>'.length
// <- 4
'foo̅'.length
// <- 4
[<mark>...</mark>'foo̅']<mark>.length</mark>
// <- 4
```

I was confused about this one as I'm no expert when it comes to unicode. So I went to someone who _is_ an expert -- [Mathias Bynens][6]. He promptly pointed out that _-- indeed --_ splitting by code points isn't enough. Unlike surrogate pairs like the emojis we've used in our earlier examples, other _grapheme clusters_ aren't taken into account by the string iterator.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/nzgb">@nzgb</a> Exactly. The string iterator iterates over code points, but not grapheme clusters. <a href="https://t.co/fTGQUOvMj8">https://t.co/fTGQUOvMj8</a></p>&mdash; Mathias Bynens (@mathias) <a href="https://twitter.com/mathias/status/643206231214161920">September 13, 2015</a></blockquote>

In these cases we're out of luck, and we simply have to [fall back to regular expressions][7] to correctly calculate the string length. For a comprehensive discussion of the subject I suggest you read his excellent ["JavaScript has a Unicode problem"][8] piece.

Let's look at the other methods.

# `String.prototype.codePointAt`

You can use `.codePointAt` to get the base-10 numeric representation of a code point at a given position in a string. Note that the position is indexed by code unit, not by code point. In the example below we print the code points for each of the three emoji in our demo `'🐎👱❤'` string.

```js
'\ud83d\udc0e\ud83d\udc71\u2764'<mark>.codePointAt(0)</mark>
// <- 128014
'\ud83d\udc0e\ud83d\udc71\u2764'.codePointAt(<mark>2</mark>)
// <- 128113
'\ud83d\udc0e\ud83d\udc71\u2764'.codePointAt(<mark>4</mark>)
// <- 10084
```

Figuring out the indices on your own may prove cumbersome, which is why you should just loop through the string iterator so that figures them out on your behalf. You can then just call `.codePointAt(0)` for each code point in the sequence.

```js
for (let codePoint of '\ud83d\udc0e\ud83d\udc71\u2764') {
  console.log(codePoint<mark>.codePointAt(0)</mark>)
  // <- 128014
  // <- 128113
  // <- 10084
}
```

Or maybe just use a combination of [spread][4] and `.map`.

```js
[<mark>...</mark>'\ud83d\udc0e\ud83d\udc71\u2764'].map(<mark>cp => cp.codePointAt(0)</mark>)
// <- [128014, 128113, 10084]
```

You could then take the hexadecimal _(base-16)_ representation of those base-10 integers and render them on a string using the new unicode code point escape syntax of `\u{codePoint}`. This syntax allows you to represent unicode code points that are beyond the _"basic multilingual plane"_ (BMP) -- i.e, code points outside the `[U+0000, U+FFFF]` range that are typically represented using the `\u1234` syntax.

Let's start by updating our example to print the hexadecimal version of our code points.

```js
for (let codePoint of '\ud83d\udc0e\ud83d\udc71\u2764') {
  console.log(codePoint.codePointAt(0)<mark>.toString(16)</mark>)
  // <- '1f40e'
  // <- '1f471'
  // <- '2764'
}
```

You can wrap those in `'\u{codePoint}'` and voilá _-- you'll get the emoji out of the string once again._

```js
'\u{1f40e}'
// <- '🐎'
'\u{1f471}'
// <- '👱'
'\u{2764}'
// <- '❤'
```

Yay!

# `String.fromCodePoint`

This method takes in a number and returns a code point. Note how I can use the `0x` prefix with the terse base-16 code points we got from `.codePointAt` moments ago.

```js
String.fromCodePoint(<mark>0x1f40e</mark>)
// <- '🐎'
String.fromCodePoint(<mark>0x1f471</mark>)
// <- '👱'
String.fromCodePoint(<mark>0x2764</mark>)
// <- '❤'
```

Obviously, you can just as well use their base-10 counterparts to achieve the same results.

```js
String.fromCodePoint(<mark>128014</mark>)
// <- '🐎'
String.fromCodePoint(<mark>128113</mark>)
// <- '👱'
String.fromCodePoint(<mark>10084</mark>)
// <- '❤'
```

You can pass in as many code points as you'd like.

```js
String.fromCodePoint(<mark>128014</mark>, <mark>128113</mark>, <mark>10084</mark>)
// <- '🐎👱❤'
```

As an exercise in futility, we could map a string to their numeric representation of code points, and back to the code points themselves.

```js
[...'\ud83d\udc0e\ud83d\udc71\u2764']
  .map(cp => cp.codePointAt(0))
  .map(cp => String.fromCodePoint(cp))
  .join('')
// <- '🐎👱❤'
```

Maybe you're feeling like playing a joke on your fellow inmates -- _I mean, coworkers_. You can now stab them to death with this piece of code that doesn't really do anything other than converting the string into code points and then spreading those code points as parameters to `String.fromCodePoint`, which in turn restores the original string. _As amusing as it is useless!_

```js
String.fromCodePoint(<mark>...</mark>[
  <mark>...</mark>'\ud83d\udc0e\ud83d\udc71\u2764'
]<mark>.map</mark>(cp => cp<mark>.codePointAt</mark>(0)))
// <- '🐎👱❤'
```

Since we're on it, you may've noticed that reversing the string itself would cause issues.

```js
'\ud83d\udc0e\ud83d\udc71\u2764'<mark>.split('')</mark>.reverse().join('')
```

The problem is that you're reversing individual code units as opposed to code points.

![Reversing individual code units ends up breaking the code points][9]

If we were to use the spread operator to split the string by its code points, and then reverse that, the code points would be preserved and the string would be properly reversed.

```js
[<mark>...</mark>'\ud83d\udc0e\ud83d\udc71\u2764'].reverse().join('')
// <- '❤👱🐎'
```

This way we avoid breaking code points, but once again keep in mind that this won't work for _all_ grapheme clusters, as Mathias pointed out in his tweet.

```js
[...'foo<mark>\u0305</mark>'].reverse().join('')
// <- <mark>' ̅oof'</mark>
```

The last method we'll cover today is `.normalize`.

# `String.prototype.normalize`

There's different ways to represent strings that look identical to humans, even though their code points differ. Mathias gives an example as follows.

```js
'mañana' === 'mañana'
// <- false
```

What's going on here? We have a combining tilde [` ̃`][13] and an [`n`][12] on the right, while the left just has an [`ñ`][11]. These [look alike][10], but if you look at the code points you'll notice they're different.

```js
[...'mañana'].map(cp => cp.codePointAt(0).toString(16))
// <- ['6d', '61', <mark>'f1'</mark>, '61', '6e', '61']
[...'mañana'].map(cp => cp.codePointAt(0).toString(16))
// <- ['6d', '61', <mark>'6e', '303'</mark>, '61', '6e', '61']
```

Just like with the `'foo̅'` example, the second string has a length of `7`, even though it is `6` glyphs long.

```js
'mañana'.length
// <- 6
'mañana'.length
// <- 7
```

If we normalize the second version, we'll get back the same code points we had in the first version.

```js
var normalized = 'mañana'<mark>.normalize()</mark>
[...normalized].map(cp => cp.codePointAt(0).toString(16))
// <- ['6d', '61', <mark>'f1'</mark>, '61', '6e', '61']
normalized.length
// <- 6
```

Just for completeness' sake, note that you can represent these code points using the `'\x6d'` syntax.

```js
'\\x' + [...'mañana'].map(cp => cp.codePointAt(0).toString(16)).join('\\x')
// <- '\\x6d\\x61\\xf1\\x61\\x6e\\x61'
'\x6d\x61\xf1\x61\x6e\x61'
// <- 'mañana'
```

We could use `.normalize` on both strings to see if they're really equal.

```js
function compare (left, right) {
  return left.normalize() === right.normalize()
}
compare('mañana', 'mañana')
// <- true
```

Or, to prove the point in something that's a bit more visible to human eyes, let's use the `\x` syntax. Note that you can only use `\x` to represent code units with codes below 256 _(`\xff` is `255`)_. For anything larger than that we should use the `\u` escape. Such is the case of the [`U+0303`][12] combining tilde.

```js
compare(
  '\x6d\x61<mark>\xf1</mark>\x61\x6e\x61',
  '\x6d\x61<mark>\x6e\u0303</mark>\x61\x6e\x61'
)
// <- true
```

See you tomorrow? _Modules are coming!_

> _Many thanks to Mathias for reviewing drafts of this article_, ❤!

[1]: /articles/es6-template-strings-in-depth "ES6 Template Literals in Depth on Pony Foo"
[2]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[3]: http://www.fileformat.info/info/unicode/char/0305/index.htm "Combining Overline Unicode Character Info"
[4]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[5]: https://i.imgur.com/EhKaySs.png
[6]: https://mathiasbynens.be "You can visit him at mathiasbynens.be"
[7]: https://mathiasbynens.be/notes/javascript-unicode#other-grapheme-clusters "Accounting for Grapheme Clusters"
[8]: https://mathiasbynens.be/notes/javascript-unicode "Read it on mathiasbynens.be"
[9]: https://i.imgur.com/KctK6mu.png
[10]: https://mathiasbynens.be/notes/javascript-unicode#accounting-for-lookalikes "Accounting for lookalikes"
[11]: https://codepoints.net/U+00F1 "U+00F1 LATIN SMALL LETTER N WITH TILDE"
[12]: https://codepoints.net/U+006E "U+006E LATIN SMALL LETTER N"
[13]: https://codepoints.net/U+0303 "U+0303 COMBINING TILDE"
[14]: http://ecma-international.org/ecma-262/6.0/#sec-tointeger "7.1.4 ToInteger in ECMAScript 6 Specification"
[15]: http://ecma-international.org/ecma-262/6.0/#sec-string.prototype.repeat "21.1.3.13 String.prototype.repeat in ECMAScript 6 Specification"
[16]: http://unicodebook.readthedocs.io/unicode_encodings.html#ucs-2-ucs-4-utf-16-and-utf-32 "UCS-2, UCS-4, UTF-16 and UTF-32"
