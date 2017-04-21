## When not to use a regex ##

Regex is a _scaringly powerful_ tool, when applied properly. That doesn't mean you should apply it to everything that is composed of strings. Parsing HTML with regex is [just wrong](http://stackoverflow.com/a/1732454/389745 "Legendary regex parsing answer on SO"), and a considerable waste of time.

> Regex is a trap. They're _hard to read_, and interpreting their intent can be a nightmare. They're **hard to debug**, and the developer next to you probably has _no idea_ what it's doing.

> They are convenient, though, when parsing text _looking for certain patterns_.

## The Basics ##

A regular expression denotes a **pattern** you want to _match_ in a particular string. Lets look at an example:

```js
var test = /my ca[rt] ate? (the|[2-4]) [sp]lums?/;

// the above regex matches all strings below
var strings = [
    'my cat ate the plum',
    'my cat ate 3 plums',
    'my car at the slums'
];
```

Think of regex as _regular strings_. If a character isn't a _special marker_, it will match that _character_. Therefore, the `/my ca/` regex, will match the `'my ca'` sub-string.

The special `/` character marks the beginning and the end of a regular expression in JavaScript. You can create regex using a constructor `new RegExp('my ca')`, but I recommend the `/form/` form.

Next up we have `[`. Everything contained in the brackets has a special meaning. `[rt]` means we'll match either an `'r'`, or a `'t'`. For example, `/ba[rz]/` matches both `'bar'` and `'baz'`.

The second special expression we have is `?`. This is a **quantifier**. Quantifiers determine how the previous expression is matched. `?` means the previous expression is _optional_. `/ate?/` will match both `'ate'` and `'at'`.

Usually, we want to do this in longer expressions than just a single character. In this case, we will use groups. These are expressions enclosed in parenthesis. `/foo (bar )?baz/`, for instance, will match both `'foo baz'`, and `'foo bar baz'`.

That brings us to the next portion of our regex, `(the|[2-4])`. Here we used a group, but we have several other special characters. The `[2-4]` expression is a range, and it means either 2, 3, or 4. Anything in the _inclusive_ 2-4 range.

The other special character in this portion, is `|`, is effectively a logical `OR`, and it means we should match one or both of the sides of this expression. In the end, this expression will be able to match any of the following: `'the'`, `'2'`, `'3'`, and `'4'`.

## Modifiers ##

Are you keeping up? Good! I'm glad I'm not as cryptic as I thought I would be. We'll look at a few more regex examples, but before, lets talk about [modifiers](http://www.regular-expressions.info/modifiers.html "Regular expression modifiers").

In JavaScript, modifiers can be provided with the `/regex/modifiers` form, such as `/foo/i`, or using the constructor form, `new RegExp('foo', 'i')`.

These are some of the most common modifiers you can use.

- `i`: Case insensitivity. Allows `/foo/i` to match `'foo'`, `'FOO'`, etc.
- `g`: Global. Matching doesn't stop after the first coincidence.

## Anchors ##

`^` represents the start of a string. Similarly, `$` represents the end. 

For example, in the string `'who let the dogs out? never let them out!'`, the regex `/out[!?]$/` will match `'out!'` in the end of the string, but it won't match `'out?'`.

A commonly used modifier I purposely left out in the previous section is `m`. The multi-line modifier. Using this modifier, anchors will work on a _line-by-line basis_, rather than on the whole string.

## Quantifiers ##

Quantifiers let you repeat patterns while staying [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself "Don't Repeat Yourself principle").

- `?`. The one we've covered, optionally matches the preceding expression.
- `+`. The preceding expression must occur at least once.
- `*`. The preceding expression can occur zero, one, or more times.
- `{n}`. The preceding expression has to occur `n` times.
- `{n,}`. The preceding expression has to occur at least `n` times.
- `{n,m}`. The preceding expression has to occur `n` to `m` times.

## Built-in patterns ##

Some, very simple, patterns that are built into regular expressions. Here are the most useful ones.

- `.` means any character, _except the new-line_
- `\` will escape any character. If you need to match an actual dot, you can use `/\./`
- `\s` matches whitespace. `\S` is any non-whitespace
- `\d` matches digits, effectively the same as `[0-9]`. `\D` negates it
- `\w` matches words, the same as `[A-z0-9]`. `\W` is the opposite

## Groups ##

Groups are useful for _replacing patterns_, I'll cover that in a minute.

There are two kinds of groups. **Capturing, and non-capturing**. Capturing groups are the groups we've been talking about so far. Enclosed in parenthesis, such as `/S(\d{2})E(\d{2})/`, which will match strings such as `'S11E18'`. It will **capture** the values `11` and `18`.

Capturing is important to perform _replacements_, one of the fundamental uses of regex. But sometimes, we want groups for other reasons, for example, when we wrote `(the|[2-4])`, we did so to keep the `OR` contained in just that portion of the regex.

In these cases, we'll want to use the _non-capturing_ group syntax. This means adding `?:` to our pattern, like this: `(?:the|[2-4])`.

## Replacements ##

To replace a string using a regex in JS, we can use `String.prototype.replace`, [passing a regex](https://developer.mozilla.org/en/docs/JavaScript/Reference/Global_Objects/String/replace "replace - MDN") in the first parameter. Lets do an example.

```js
// a pretty non-sensical example
'the cow is a cow, but the cat is not a cow.'.replace(/cow/, 'dog');

// the result is
'the dog is a cow, but the cat is not a cow.'

// not quite what we wanted, we forgot to add the g modifier.
'the cow is a cow, but the cat is not a cow.'.replace(/cow/g, 'dog');

// the result now is
'the dog is a dog, but the cat is not a dog.'
```

You could also use `$1`, `$2`, and so on, in the replacement string. These will get replaced with the group captured when matching your regex. Another example!

```js
// more non-sense please
var rimportant = /(\d+|boss)/g,
    remphasize = '<em>$1</em>';

'build 102 errored... tell the boss it failed!'.replace(rimportant, remphasize);

// results in
'build <em>102</em> errored... tell the <em>boss</em> it failed!'
```

Alternatively, you could provide a function callback as a replacement parameter, and it will be invoked once for each match. You can find more info on that on [MDN](https://developer.mozilla.org/en/docs/JavaScript/Reference/Global_Objects/String/replace "replace - MDN").

You could also use `RegExp.prototype.test` to [test](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/RegExp/test "test - MDN") whether a pattern matches the provided string.

### Conclusion ###

In this post we've looked at some of the most common patterns of regex. Most importantly, we've looked at the way in which we can build simple regular expressions. I intentionally left out a complicated subset of regular expressions, in [assertions](http://www.regular-expressions.info/lookaround.html "Lookaround assertions"), I will definitely cover that topic at some point in the future.

I test out my regular expressions using this online [REGex Tester](http://regextester.com/ "REGex Tester Tool") tool, or directly in my browser if they are simple enough.

[TL;DR cheatsheet](https://i.imgur.com/UTlGckN.png "Regular Expressions Cheat Sheet")
