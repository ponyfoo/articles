# Rationale

Before getting into the implementation, it might be useful to read about the [rationale behind this addition][3] to the language, which was put together by the people who drafted the proposal.

> Without a reasonable way to pad a string using native methods, working with JavaScript strings today is more painful than it should be. Without these functions, the language feels incomplete, and is a paper cut to what could be a very polished experience.
>
> Due to common use, string padding functions exist in a majority of websites and frameworks. For example, nearly every app in FirefoxOS had implemented a left pad function, because they all needed some generic string padding operation.
>
> It is highly probable that the majority of current string padding implementations are inefficient. Bringing this into the platform will improve performance of the web, and developer productivity as they no longer have to implement these common functions.

Let's look into `.padStart` first.

# `String.prototype.padStart(max, fillString)`

In its simplest incarnation, `.padStart` will add enough spaces to the start of a string so that it arrives at the requested <mark>length</mark>.

```js
'abc'.padStart(<mark>6</mark>);
// <- '   abc'
```

When a string's length is at -- or over -- the specified `max` length, no padding characters will be inserted.

```js
'abc'.padStart(3);
// <- 'abc'
'abc'.padStart(2);
// <- 'abc'
```

Naturally you can change the padding character to something that's not an space.

```js
'abc'.padStart(5, 'x');
// <- 'xxabc'
```

The padding doesn't need to be a single character either. The padding string is repeated as long as needed but it will be sliced to meet the character length requirement.

```js
'abc'.padStart(7, 'xo');
// <- 'xoxoabc'
'abc'.padStart(<mark>6</mark>, 'xo');
// <- '<mark>xox</mark>abc'
```

An interesting use case for `padStart` might be for providing input masks. The example below shows how you could implement such an input mask for ten-digit strings.

```js
'1'.padStart(10, '0');
// <- '0000000001'
'12'.padStart(10, '0');
// <- '0000000012'
'123456'.padStart(10, '0');
// <- '0000123456'
```

How about dates?

```js
'12'.padStart(10, 'YYYY-MM-DD');
// <- 'YYYY-MM-12'
'09-12'.padStart(10, 'YYYY-MM-DD');
// <- 'YYYY-09-12'
```

Here's an adaptation of the [reference polyfill][1] stripped off sanity checks and all the usual spec-babble that makes these code snippets harder to read. If you want to use the method in an application today, I suggest you install the [`string.prototype.padstart`][2] module, _instead of copying and pasting the polyfill._

```js
if (!String.prototype.padStart) {
  String.prototype.padStart = function (max, fillString) {
    return padStart(this, max, fillString);
  };
}

function padStart (text, max, mask) {
  const cur = text.length;
  if (max <= cur) {
    return text;
  }
  const masked = max - cur;
  let filler = String(mask) || ' ';
  while (filler.length < masked) {
    filler += filler;
  }
  const fillerSlice = filler.slice(0, masked);
  return fillerSlice + text;
}
```

Time to move onto `.padEnd`.

# `String.prototype.padEnd(max, fillString)`

This method is equivalent to `.padStart` except for the fact that padding is appended to the string rather than prepended to it.

```js
'123'.padEnd(6);
// <- '123   '
```

You can specify the padding character with `.padEnd` as well.

```js
'123'.padEnd(6, 'x');
// <- '123xxx'
```

Note that masking preserves the original padding, which may lead to confusion. For example, you may expect the following to produce `'123456'`.

```js
'123'.padEnd(6, '123456');
// <- '123123'
```

Similarly, you may want the following to produce `'2016-MM-DD'`.

```js
'2016'.padEnd(10, 'YYYY-MM-DD');
// <- '2016YYYY-M'
```

One work-around to get the desired result in these cases could be to just `.slice` the mask against the length of the original string.

```js
const year = '2016';
year.padEnd(10, 'YYYY-MM-DD'<mark>.slice(year.length)</mark>);
// <- '2016-MM-DD'
```

Here's an adaptation of the [reference polyfill][5] stripped off sanity checks and all the usual spec-babble that makes these code snippets harder to read. If you want to use the method in an application today, I suggest you install the [`string.prototype.padend`][4] module, _instead of copying and pasting the polyfill._

```js
if (!String.prototype.padEnd) {
  String.prototype.padEnd = function (max, fillString) {
    return padEnd(this, max, fillString);
  };
}

function padEnd (text, max, mask) {
  const cur = text.length;
  if (max <= cur) {
    return text;
  }
  const masked = max - cur;
  let filler = String(mask) || ' ';
  while (filler.length < masked) {
    filler += filler;
  }
  const fillerSlice = filler.slice(0, masked);
  return text + fillerSlice;
}
```

> As you can see above, the only difference between the polyfill for `.padStart` and the one for `.padEnd` is what side of `text` the `fillerSlice` portion of padded fill mask gets concatenated.

If you like this kind of article let me know and I'll see what I can do to write more like it! 😉

[1]: https://github.com/tc39/proposal-string-pad-start-end/blob/8606ec6ae00e442cb2fa7e9504dcab1c77360aa5/polyfill.js#L15-L37 "String.prototype.padStart polyfill on GitHub"
[2]: https://github.com/es-shims/String.prototype.padStart "es-shims/String.prototype.padStart on GitHub"
[3]: https://github.com/tc39/proposal-string-pad-start-end/tree/8606ec6ae00e442cb2fa7e9504dcab1c77360aa5#rationale "Rationale behind String.prototype.padStart / String.prototype.padEnd"
[4]: https://github.com/es-shims/String.prototype.padEnd "es-shims/String.prototype.padEnd on GitHub"
[5]: https://github.com/tc39/proposal-string-pad-start-end/blob/8606ec6ae00e442cb2fa7e9504dcab1c77360aa5/polyfill.js#L39-L61 "String.prototype.padEnd polyfill on GitHub"
