# Expression Interpolation

Assuming an ES6 codebase, there are several ways to do interpolation in strings.

You could use string concatenation, involving `+` signs and a string for each piece of the template you're working on. Using concatenation can quickly get out of hand when dealing with longer templates, not to mention reusability is hard unless you abstract away your template in a small function.

```js
'Hello ' + name + '! It\'s a pleasure to greet you.'
```

You could use `util.format` in Node. Here we have placeholders such as `%s` which describes how the value is to be formatted, but doesn't say anything about the value that should be placed in that socket in the template. This method _-- and similar ones --_ scales better than string concatenation, but it can be difficult to infer the context for each replacement value in the template without a small function that matches its arguments to each value for the template.

```js
util.format('Hello %s! It\'s a pleasure to greet you.', name)
```

You could use [string interpolation in template literals][strings]. Here you can inline JavaScript expressions such as `expression` wrapped in `${expression}`. It's nice to have a name for the slot, such as the `name` for the person you're greeting, placed directly within the template.

```js
`Hello ${name}! It's a pleasure to greet you.`
```

Of course, nothing is stopping you from using template literals in the previous two incarnations of interpolation. Overlooking the fact that we should probably be just using expression interpolation within a single template literal, nothing stops us from using template literals in the exact same way as we used quoted strings.

```js
`Hello ` + name + `! It's a pleasure to greet you.`
util.format(`Hello %s! It's a pleasure to greet you.`, name)
```

Template literals give us the choice to interpolate inline in the string, if we want to, while quoted strings don't. As an added bonus, interpolation could be several levels deep: any expressions _-- including other template literals --_ can be used for interpolations.

# Character Escaping

This one is fairly minor, but it's still worth pointing out.

In single quoted strings, we need to escape single quotes using `\'`.

```js
'What\'s that?'
```

In double quoted strings, we need to escape double quotes using `\"`.

```js
"Hey, who's \"that\" are you referring to?"
```

In template literals, we need to escape backticks using `` \` ``.

```js
`Hey, programmers use backticks to render code, like \`this\`.`
```

You're far less likely to need backticks in your everyday strings than single or double quotes, which are commonplace in english and language in general. Using template literals, then, translates into less escape codes that can pollute your otherwise beautiful strings.

# Multiline Strings

Making regular strings multiline can be annoying. We have a variety of options, though, for both single and double quoted strings.

Using an array and then concatenating with `\n` is possibly my favorite approach. Here we need individual strings per line, and the surrounding array with its join, but there's no escaping involved.

```js
[
  '# Ingredients',
  '',
  '- 2x Tomato',
  '- 2x Lemon',
  '- 300cc Vodka',
  '- Pepper'
].join('\n')
```

An almost identical approach, that seems more typically favored by JavaScript developers, is to concatenate using `+`. In this case we'd have to add our `\n` ourselves. Typically, however, developers do not mind the lack of new lines because their multiline string is used to declare a piece of HTML for Angular and jQuery applications. Using `+` is a bit noisier than using `,`, and we'd have to hand-roll our `\n`, but it does allow for us to use a single string. Even though inline HTML is far from ideal, avoiding new lines is hardly a problem on top of that.

```js
'<div>' +
  '<span>Hello </span>' +
  '<span>' +
  name +
  '</span>' +
'</div>'
```

An approach you hardly ever see is using `\n` new line escapes followed by an escape `\` that allows you to continue writing the same string on a new line. This is noisy on the right hand side, but not as noisy on the left hand side, but it does allow for us to use a single string.

```js
'# Ingredients\n\
\n\
- 2x Tomato\n\
- 2x Lemon\n\
- 300cc Vodka\n\
- Pepper'
```

Plus, if you don't need the `\n` escapes, _-- as is the case with HTML strings --_ you can end up with quite decent looking strings.

```js
'<div>\
  <span>Hello </span>\
  <span>' +
  name +
  '</span>\
</div>'
```

Wait, that didn't look so great. That's because as soon as we have some sort of interpolation in the string, we're back to concatenating, and it can get awful.

Then there's other less frequently surfaced approaches, such as using a comment in a function and a library to extract the comment into a multiline string. Here's an example using Sindre's [`multiline`][ml] package. This approach only has noise on the beginning and end of the declaration, but not within, which is why it can be preferred. It does involve a third party package and a convoluted hack just to render multiline strings, which some people could take issue with.

```js
multiline(function(){/*
<!doctype html>
<html>
  <body>
    <h1>Pony Foo</h1>
  </body>
</html>
*/});
```

When it comes to template literals, multiline support is included by default. The following example, using template literals, is equivalent to the previous one where we used `multiline`.

```js
`<!doctype html>
<html>
  <body>
    <h1>Pony Foo</h1>
  </body>
</html>`
```

If you have expressions you need to interpolate into your multiline strings, template literals aren't shaken up like other approaches.

```js
`<!doctype html>
<html>
  <body>
    <h1><mark>${title}</mark></h1>
  </body>
</html>`
```

Plus, if you're worried about XSS and you'd like to HTML-encode interpolations, you could use tagged templates that automatically do this for you. The following example uses an `encode` tagged template to escape interpolations in an HTML string using a brittle `escape` implementation.

```js
function encode (template, ...expressions) {
  const escape = text => text
    .replace(/&/g, `&amp;`)
    .replace(/"/g, `&quot;`)
    .replace(/'/g, `&#39;`)
    .replace(/</g, `&lt;`)
    .replace(/>/g, `&gt;`)

  return template.reduce((result, part, i) => {
    return result + <mark>escape(expressions[i - 1])</mark> + part)
  })
}

const title = `<script src='https://malicio.us/js'></script>`

<mark>encode</mark>`<!doctype html>
<html>
  <body>
    <h1>${title}</h1>
  </body>
</html>`
```

The resulting string is plain HTML, where interpolated expressions have been HTML encoded.

```html
<!doctype html>
<html>
  <body>
    <h1>&lt;script src=&#39;https://malicio.us/js&#39;&gt;&lt;/script&gt;</h1>
  </body>
</html>
```

Again, you have the option of doing all of this, but nothing impedes you from applying any of the previous approaches to template literals.

# But, JSON.

The JSON argument is bound to come up, but we can shrug it off as a weak one. In JSON, strings _-- and even property keys --_ must be double quoted.

```json
{
  "key": "value"
}
```

You can't use single quotes to declare strings in JSON, yet they are typically favored over double quotes when it comes to JavaScript code. The same argument can, _by transitive property_, be applied to template literals: you can't use template literals to declare strings in JSON, yet they should be favored over single or double quotes.

# Choose Your Weapon

Here's a Twitter poll I ran last week, asking about whether people are using template literals as their default string representation of choice.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">🐑 How do you prefer your JavaScript strings?<br>💡 Strongly prefer template literals<br>🚓 No need to escape, interpolation</p>&mdash; Nicolás Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/772811506270470144">September 5, 2016</a></blockquote>

Half of respondents still use single quotes, where close to half claim to be prefer template literals. Some replied to my tweet saying that they use template literals when they explicitly need interpolation or multiline support, but otherwise fall back to single quoted strings.

Using backticks is better because you rarely need to escape backticks `` \` ``, you get multiline support, and you get variable interpolation. Many admitted they'd switch over from single quoted strings to template literals when interpolation or multiline support was required, so why not switch everything over by default and take advantage of the lower amount of `` \` `` escapes as opposed to `\'`?

The only valid argument I've read against using template literals, much like with `const`, is that people are used to single quoted strings. While familiarity is a fair claim, you can conjure up the habit of using template literals very quickly, as we'll explore next.

# Using `eslint`: the `quotes` Rule

[ESLint][eslint] is a modern linter for JavaScript code bases. If you're still on another linter, I strongly encourage you to check it out.

The [`"quotes"` rule][eslintrc] tells ESLint that string literals must be declared using backticks, or template literals, and that otherwise an error should be reported. This way we can ensure that, going forward, we're using template literals across our codebase.

```js
"quotes": ["error", "backtick"]
```

_"But, Nico"_ -- you may object -- _"a rule like this would report hundreds of stylistic errors in my code base, and I have no time to fix all of that."_. You'd be absolutely right.

ESLint ships with a `--fix` flag for its CLI, where it'll attempt to resolve stylistic issues, before reporting them as hard errors. If an stylistic issue can be fixed, then `--fix` will take care of it for you without ever complaining about it. As it turns out, the `quotes` rule is an stylistic choice that is easily fixed with `--fix`.

The following command would turn all your strings into template literals, provided you've set up the `quotes` rule to `"backtick"`.

```
eslint --fix .
```

Ta-da! 🎉

You may find your code base now has bits like the following, instead of the single quoted strings you previously had.

```js
`Hello ` + name + `!`
```

You can gradually upgrade code like the above into interpolated expressions over time, getting to a more consistent code base. There's no excuse, _however_, to get away with not using template literals!

Now go evangelize the use of template literals everywhere, so that I can sleep better at night. 🌒

_<sub>This article is sponsored by the backtick conglomerate holding commonly referred to as "best practices".</sub>_

[strings]: /articles/es6-template-strings-in-depth "ES6 Template Literals in Depth on Pony Foo"
[ml]: https://github.com/sindresorhus/multiline "sindresorhus/multiline on GitHub"
[eslintrc]: https://github.com/ponyfoo/ponyfoo/blob/e10c224c44c8a9c69c52ca2afc19a849bd4059f2/.eslintrc.json#L14 "An example .eslintrc.json file"
[eslint]: http://eslint.org/ "The Pluggable JavaScript Linter"
