# Unicode Flag `/u`

ES6 also introduced a `u` flag. The `u` stands for Unicode, but this flag can also be thought of as a more strict version of regular expressions.

Without the `u` flag, the following snippet has a regular expression containing an `'a'` character literal that was unnecessarily escaped.

```js
/\a/.test('ab')
// <- true
```

Using an escape sequence for an unreserved character such as `a` in a regular expression with the `u` flag results in an error, as shown in the following bit of code.

```js
/\a/u.test('ab')
// <- SyntaxError: Invalid regular expression: /\a/: Invalid escape
```

The following example attempts to embed the horse emoji in a regular expression by way of the `\u{1f40e}` notation which ES6 introduced for strings like `'\u{1f40e}'`, but the regular expression fails to match against the horse emoji. Without the `u` flag, the `\u{…}` pattern is interpreted as having an unnecessarily escaped `u` character followed by the rest of the sequence.

```js
/\u{1f40e}/.test(':racehorse:') // <- false
/\u{1f40e}/.test('u{1f40e}') // <- true
```

The `u` flag introduces support for Unicode code point escapes, like the `\u{1f40e}` horse emoji, within regular expressions.

```js
/\u{1f40e}/u.test(':racehorse:')
// <- true
```

Without the `u` flag, the `.` pattern matches any BMP symbol except for line terminators. The following example shows `U+1D11E MUSICAL SYMBOL G CLEF`, an astral symbol that wouldn't match the dot pattern in plain regular expressions.

```js
const rdot = /^.$/
rdot.test('a') // <- true
rdot.test('\n') // <- false
rdot.test('\u{1d11e}') // <- false
```

When using the `u` flag, Unicode symbols that aren't on the BMP are matched as well. The next snippet shows how the astral symbol matches when the flag is set.

```js
const rdot = /^.$/u
rdot.test('a') // <- true
rdot.test('\n') // <- false
rdot.test('\u{1d11e}') // <- true
```

When the `u` flag is set, similar Unicode awareness improvements can be found in quantifiers and character classes, both of which treat each Unicode code point as a single symbol, instead of matching on the first code unit only. Insensitive case matching with the `i` flag performs Unicode case folding when the `u` flag is set as well, which is used to normalize code points in both the input string and the regular expression.

For more details around the `u` flag in regular expressions, read [this piece by Mathias Bynens][ru].

# Named Capture Groups

Up until now, JavaScript regular expressions could group matches in numbered capturing groups and non-capturing groups. In the next snippet we're using a couple of groups to extract a key and value from an input string containing a key value pair delimited by `'='`.

```js
function parseKeyValuePair(input) {
  const rattribute = /([a-z]+)=([a-z]+)/
  const [, key, value] = rattribute.exec(input)
  return { key, value }
}
parseKeyValuePair('strong=true')
// <- { key: 'strong', value: 'true' }
```

There's also non-capturing groups, which are discarded and not present in the final result, but are still useful for matching. The following example supports input with key value pairs delimited by `' is '` in addition to `'='`.

```js
function parseKeyValuePair(input) {
  const rattribute = /([a-z]+)(?:=|\sis\s)([a-z]+)/
  const [, key, value] = rattribute.exec(input)
  return { key, value }
}
parseKeyValuePair('strong is true') // <- { key: 'strong', value: 'true' }
parseKeyValuePair('flexible=too') // <- { key: 'flexible', value: 'too' }
```

While array destructuring in the previous example hid our code's reliance on magic array indices, the fact remains that matches are placed in an ordered array regardless. The [named capture groups proposal][ncg] *(in stage 3 at the time of this writing)* adds syntax like `(?<groupName>)` where we can name capturing groups, which are then returned in a `groups` property of the returned match object. The `groups` property can then be destructured from the resulting object when calling `RegExp#exec` or `String#match`.

```js
function parseKeyValuePair(input) {
  const rattribute = /(?<key>[a-z]+)(?:=|\sis\s)(?<value>[a-z]+)/u
  const { groups } = rattribute.exec(input)
  return groups
}
parseKeyValuePair('strong=true') // <- { key: 'strong', value: 'true' }
parseKeyValuePair('flexible=too') // <- { key: 'flexible', value: 'too' }
```

JavaScript regular expressions support backreferences, where captured groups can be reused to look for duplicates. The following snippet uses a backreference for the first capturing group to identify cases where a username is the same as a password in a piece of `'user:password'` input.

```js
function hasSameUserAndPassword(input) {
  const rduplicate = /([^:]+):\1/
  return rduplicate.exec(input) !== null
}
hasSameUserAndPassword('root:root') // <- true
hasSameUserAndPassword('root:pF6GGlyPhoy1!9i') // <- false
```

The named capture groups proposal adds support for named backreferences, which refer back to named groups.

```js
function hasSameUserAndPassword(input) {
  const rduplicate = /(?<user>[^:]+):\k<user>/u
  return rduplicate.exec(input) !== null
}
hasSameUserAndPassword('root:root') // <- true
hasSameUserAndPassword('root:pF6GGlyPhoy1!9i') // <- false
```

The `\k<groupName>` reference can be used in tandem with numbered references, but the latter are better avoided when already using named references.

Lastly, named groups can be referenced from the replacement passed to `String#replace`. In the next code snippet we use `String#replace` and named groups to change an American date string to use Hungarian formatting.

```js
function americanDateToHungarianFormat(input) {
  const ramerican = /(?<month>\d{2})\/(?<day>\d{2})\/(?<year>\d{4})/u
  const hungarian = input.replace(ramerican, '$<year>-$<month>-$<day>')
  return hungarian
}
americanDateToHungarianFormat('06/09/1988') // <- '1988-09-06'
```

If the second argument to `String#replace` is a function, then the named groups can be accessed via a new parameter called `groups` that is at the end of the parameter list. The signature for that function now is `(match, ...captures, groups)`. In the following example, note how we're using a template literal that's similar to the replacement string found in the last example. The fact that replacement strings follow a `$<groupName>` syntax as opposed to a `${groupName}` syntax means we can name groups in replacement strings without having to resort to escape codes if we were using template literals.

```js
function americanDateToHungarianFormat(input) {
  const ramerican = /(?<month>\d{2})\/(?<day>\d{2})\/(?<year>\d{4})/u
  const hungarian = input.replace(ramerican, (match, capture1, capture2, capture3, groups) => {
    const { month, day, year } = groups
    return `${ year }-${ month }-${ day }`
  })
  return hungarian
}
americanDateToHungarianFormat('06/09/1988') // <- '1988-09-06'
```

# Unicode Property Escapes

The proposed [Unicode property escapes][upe] *(currently in stage 3)* are a new kind of escape sequence that's available in regular expressions marked with the `u` flag. This proposal adds a escape in the form of `\p{LoneUnicodePropertyNameOrValue}` for binary Unicode properties and `\p{UnicodePropertyName=UnicodePropertyValue}` for non-binary Unicode properties. In addition, `\P` is the negated version of a `\p` escape sequence.

The Unicode standard defines properties for every symbol. Armed with these properties, one may make advanced queries about Unicode characters. For example, symbols in the greek alphabet have a `Script` property set to `Greek`. We could use the new escapes to match any greek Unicode symbol.

```js
function isGreekSymbol(input) {
  const rgreek = /^\p{Script=Greek}$/u
  return rgreek.test(input)
}
isGreekSymbol('π')
// <- true
```

Or, using `\P`, we could match non-greek Unicode symbols.

```js
function isNonGreekSymbol(input) {
  const rgreek = /^\P{Script=Greek}$/u
  return rgreek.test(input)
}
isNonGreekSymbol('π')
// <- false
```

When we need to match every Unicode decimal number symbol, and not just `[0-9]` like `\d` does, we could use `\p{Decimal_Number}` as shown next.

```js
function isDecimalNumber(input) {
  const rdigits = /^\p{Decimal_Number}+$/u
  return rdigits.test(input)
}
isDecimalNumber('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼')
// <- true
```

Linked next is a full list of [supported Unicode properties and values][props].

# Lookbehind Assertions

JavaScript has had positive lookahead assertions for a long time. That feature allows us to match an expression but only if it's followed by another expression. These assertions are expressed as `(?=…)`. Regardless of whether a lookahead assertion matches, the results of that match are discarded and no characters of the input string are consumed.

The following example uses a positive lookahead to test whether an input string has a sequence of letters followed by `.js`, in which case it returns the filename without the `.js` part.

```js
function getJavaScriptFilename(input) {
  const rfile = /^(?<filename>[a-z]+)(?=\.js)\.[a-z]+$/u
  const match = rfile.exec(input)
  if (match === null) {
    return null
  }
  return match.groups.filename
}
getJavaScriptFilename('index.js') // <- 'index'
getJavaScriptFilename('index.php') // <- null
```

There are also negative lookahead assertions, which are expressed as `(?!…)` as opposed to `(?=…)` for positive lookaheads. In this case, the assertion succeeds only if the lookahead expression isn't matched. The next bit of code uses a negative lookahead and we can observe how the results are flipped: now any expression other than `'.js'` results in a passed assertion.

```js
function getNonJavaScriptFilename(input) {
  const rfile = /^(?<filename>[a-z]+)(?!\.js)\.[a-z]+$/u
  const match = rfile.exec(input)
  if (match === null) {
    return null
  }
  return match.groups.filename
}
getNonJavaScriptFilename('index.js') // <- null
getNonJavaScriptFilename('index.php') // <- 'index'
```

The [proposal for lookbehind][lb] *(stage 3)* introduces positive and negative lookbehind assertions, denoted with `(?<=…)` and `(?<!…)` respectively. These assertions can be used to ensure a pattern we want to match is or isn't preceded by another given pattern. The following snippet uses a positive lookbehind to match the digits in dollar amounts, but not for amounts in euros.

```js
function getDollarAmount(input) {
  const rdollars = /^(?<=\$)(?<amount>\d+(?:\.\d+)?)$/u
  const match = rdollars.exec(input)
  if (match === null) {
    return null
  }
  return match.groups.amount
}
getDollarAmount('$12.34') // <- '12.34'
getDollarAmount('€12.34') // <- null
```

On the other hand, a negative lookbehind could be used to match numbers that aren't preceded by a dollar sign.

```js
function getNonDollarAmount(input) {
  const rnumbers = /^(?<!\$)(?<amount>\d+(?:\.\d+)?)$/u
  const match = rnumbers.exec(input)
  if (match === null) {
    return null
  }
  return match.groups.amount
}
getNonDollarAmount('$12.34') // <- null
getNonDollarAmount('€12.34') // <- '12.34'
```

# A New `/s` _(`dotAll`)_ Flag

When using the `.` pattern, we typically expect to match every single character. In JavaScript, however, a `.` expression doesn't match astral characters *(which can be fixed by adding the `u` flag)* nor line terminators.

```js
const rcharacter = /^.$/
rcharacter.test('a') // <- true
rcharacter.test('\t') // <- true
rcharacter.test('\n') // <- false
```

This sometimes drives developers to write other kinds of expressions to synthesize a pattern that matches any character. The expression in the next bit of code matches any character that's either a whitespace character or a non-whitespace character, delivering the behavior we'd expect from the `.` pattern matcher.

```js
const rcharacter = /^[\s\S]$/
rcharacter.test('a') // <- true
rcharacter.test('\t') // <- true
rcharacter.test('\n') // <- true
```

The [`dotAll` proposal][dotall] *(stage 3)* adds an `s` flag which changes the behavior of `.` in JavaScript regular expressions to match any single character.

```js
const rcharacter = /^.$/s
rcharacter.test('a') // <- true
rcharacter.test('\t') // <- true
rcharacter.test('\n') // <- true
```

# `String#matchAll`

Often, when we have a regular expression with a global or sticky flag, we want to iterate over the set of captured groups for each match. Currently, it can be a bit of a hassle to produce the list of matches: we need to collect the captured groups using `String#match` or `RegExp#exec` in a loop, until the regular expression doesn't match the input starting at the `lastIndex` position property. In the following piece of code, the `parseAttributes` generator function does just that for a given regular expression.

```js
function* parseAttributes(input) {
  const rattributes = /(\w+)="([^"]+)"\s/ig
  while (true) {
    const match = rattributes.exec(input)
    if (match === null) {
      break
    }
    const [ , key, value] = match
    yield [key, value]
  }
}
const html = '<input type="email" placeholder="hello@mjavascript.com" />'
console.log(...parseAttributes(html))
// <- ['type', 'email'] ['placeholder', 'hello@mjavascript.com']
```

One problem with this approach is that it's tailor-made for our regular expression and its capturing groups. We could fix that issue by creating a `matchAll` generator which is only concerned about looping over matches and collecting sets of captured groups, as shown in the following snippet.

```js
function* matchAll(regex, input) {
  while (true) {
    const match = regex.exec(input)
    if (match === null) {
      break
    }
    const [ , ...captures] = match
    yield captures
  }
}
function* parseAttributes(input) {
  const rattributes = /(\w+)="([^"]+)"\s/ig
  yield* matchAll(rattributes, input)
}
const html = '<input type="email" placeholder="hello@mjavascript.com" />'
console.log(...parseAttributes(html))
// <- ['type', 'email'] ['placeholder', 'hello@mjavascript.com']
```

A bigger source of confusion is that `rattributes` mutates its `lastIndex` property on each call to `RegExp#exec`, which is how it can track the position after the last match. When there are no matches left, `lastIndex` is reset back to `0`. A problem arises when we don't iterate over all possible matches for a piece of input in one go -- which would reset `lastIndex` to `0` -- and then we use the regular expression on a second piece of input, obtaining unexpected results.

While it looks like our `matchAll` implementation wouldn't fall victim of this given it loops over all matches, it's be possible to iterate over the generator by hand, meaning that we'd run into trouble if we reused the same regular expression, as shown in the next bit of code. Note how the second matcher should report `['type', 'text']` but instead starts at an index much further ahead than `0`, even misreporting the `'placeholder'` key as `'laceholder'`.

```js
const rattributes = /(\w+)="([^"]+)"\s/ig
const email = '<input type="email" placeholder="hello@mjavascript.com" />'
const emailMatcher = matchAll(rattributes, email)
const address = '<input type="text" placeholder="Enter your business address" />'
const addressMatcher = matchAll(rattributes, address)
console.log(emailMatcher.next().value)
// <- ['type', 'email']
console.log(addressMatcher.next().value)
// <- ['laceholder', 'Enter your business address']
```

One solution would be to change `matchAll` so that `lastIndex` is always `0` when we yield back to the consumer code, while keeping track of `lastIndex` internally so that we can pick up where we left off in each step of the sequence.

The following piece of code shows that indeed, that'd fix the problems we're observing. Reusable global regular expressions are often avoided for this very reason: so that we don't have to worry about resetting `lastIndex` after every use.

```js
function* matchAll(regex, input) {
  let lastIndex = 0
  while (true) {
    regex.lastIndex = lastIndex
    const match = regex.exec(input)
    if (match === null) {
      break
    }
    lastIndex = regex.lastIndex
    regex.lastIndex = 0
    const [ , ...captures] = match
    yield captures
  }
}
const rattributes = /(\w+)="([^"]+)"\s/ig
const email = '<input type="email" placeholder="hello@mjavascript.com" />'
const emailMatcher = matchAll(rattributes, email)
const address = '<input type="text" placeholder="Enter your business address" />'
const addressMatcher = matchAll(rattributes, address)
console.log(emailMatcher.next().value)
// <- ['type', 'email']
console.log(addressMatcher.next().value)
// <- ['type', 'text']
console.log(emailMatcher.next().value)
// <- ['placeholder', 'hello@mjavascript.com']
console.log(addressMatcher.next().value)
// <- ['placeholder', 'Enter your business address']
```

The [`String#matchAll` proposal][sma] *(in stage 1 at the time of this writing)* introduces a new method for the string prototype which would behave in a similar fashion as our `matchAll` implementation, except the returned iterable is a sequence of `match` object as opposed to just the `captures` in the example above. Note that the `String#matchAll` sequence contains entire `match` objects, and not just numbered captures. This means we could access named captures through `match.groups` for each `match` in the sequence.

```js
const rattributes = /(?<key>\w+)="(?<value>[^"]+)"\s/igu
const email = '<input type="email" placeholder="hello@mjavascript.com" />'
for (const { groups: { key, value } } of email.matchAll(rattributes)) {
  console.log(`${ key }: ${ value }`)
}
// <- type: email
// <- placeholder: hello@mjavascript.com
```

[ncg]: https://mjavascript.com/out/regexp-named-groups "Named capture groups proposal document"
[upe]: https://mjavascript.com/out/unicode-property-escapes "Unicode property escapes proposal document"
[props]: https://mjavascript.com/out/unicode-property-list "Exhaustive overview of supported Unicode properties and values"
[lb]: https://mjavascript.com/out/regexp-lookbehind "Lookbehind assertions proposal document"
[dotall]: https://mjavascript.com/out/regexp-dotall "'dotAll' flag proposal document"
[sma]: https://mjavascript.com/out/string-matchall "String#matchAll proposal document"
[ru]:  https://mjavascript.com/out/regexp-unicode "Unicode-aware regular expressions in ECMAScript 6 by Mathias Bynens"
