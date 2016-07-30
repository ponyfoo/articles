## Using Template Literals

We've already covered the basic _`` `I'm just a string` ``_. One aspect of template literals that may be worth mentioning is that you're now able to declare strings with both `'` and `"` quotation marks in them without having to escape anything.

```js
var text = `I'm "amazed" that we have so many quotation marks to choose from!`
```

That was neat, but surely there's more useful stuff we can apply template literals to. How about some _actual interpolation_? You can use the `${expression}` notation for that.

```js
var host = 'ponyfoo.com'
var text = `this blog lives at <mark>${host}</mark>`
console.log(text)
// <- 'this blog lives at ponyfoo.com'
```

I've already mentioned you can have any kind of expressions you want in <mark>there</mark>. Think of whatever expressions you put in there as defining a variable before the template runs, and then concatenating that value with the rest of the string. That means that variables you use, methods you call, and so on, should all be available to the current scope.

The following expressions would all work just as well. It'll be up to us to decide how much logic we cram into the interpolation expressions.

```js
var text = `this blog lives at ${'ponyfoo.com'}`
console.log(text)
// <- 'this blog lives at ponyfoo.com'
```

```js
var today = new Date()
var text = `the time and date is ${today.toLocaleString()}`
console.log(text)
// <- 'the time and date is 8/26/2015, 3:15:20 PM'
```

```js
import moment from 'moment'
var today = new Date()
var text = `today is the ${moment(today).format('Do [of] MMMM')}`
console.log(text)
// <- 'today is the 26th of August'
```

```js
var text = `such ${Infinity/0}, very uncertain`
console.log(text)
// <- 'such Infinity, very uncertain'
```

Multi-line strings mean that you no longer have to use methods like these anymore.

```js
var text = (
  'foo\n' +
  'bar\n' +
  'baz'
)
```

```js
var text = [
  'foo',
  'bar',
  'baz'
].join('\n')
```

Instead, you can now just use backticks! Note that spacing matters, so you might still want to use parenthesis in order to keep the first line of text away from the variable declaration.

```js
var text = (
`foo
bar
baz`)
```

Multi-line strings really shine when you have, _for instance_, a chunk of HTML you want to interpolate some variables to. Much like with [JSX][1], you're perfectly able to use an expression to iterate over a collection and `return` yet another template literal to declare list items. This makes it a breeze to declare sub-components in your templates. Note also how I'm [using destructuring][2] to avoid having to prefix every expression of mine with `article.`, I like to think of it as _"a `with` block, but not as insane"_.

```js
var article = {
  title: 'Hello Template Literals',
  teaser: 'String interpolation is awesome. Here are some features',
  body: 'Lots and lots of sanitized HTML',
  tags: ['es6', 'template-literals', 'es6-in-depth']
}
var {title,teaser,body,tags} = article
var html = `<article>
  <header>
    <h1>${title}</h1>
  </header>
  <section>
    <div>${teaser}</div>
    <div>${body}</div>
  </section>
  <footer>
    <ul>
      <mark>${tags.map(tag => `<li>${tag}</li>`).join('\n      ')}</mark>
    </ul>
  </footer>
</article>`
```

The above will produce output as shown below. Note how the spacing trick was enough to properly indent the `<li>` tags.

```html
<article>
  <header>
    <h1>Hello Template Literals</h1>
  </header>
  <section>
    <div>String interpolation is awesome. Here are some features</div>
    <div>Lots and lots of sanitized HTML</div>
  </section>
  <footer>
    <ul>
      <li>es6</li>
      <li>template-literals</li>
      <li>es6-in-depth</li>
    </ul>
  </footer>
</article>
```

Raw templates are the same in essence, you just have to prepend your template literal with `String.raw`. This can be very convenient in some use cases.

```js
var text = String.raw`The "\n" newline won't result in a new line.
It'll be escaped.`
console.log(text)
// The "\n" newline won't result in a new line.
// It'll be escaped.
```

You might've noticed that `String.raw` seems to be a special part of the template literal syntax, and you'd be right! The method you choose will be used to parse the template. Template literal methods -- called _"tagged templates"_ -- receive an array containing a list of the static parts of the template, as well as each expression on their own variables.

For instance a template literal like `` `hello ${name}. I am ${emotion}!` `` will pass arguments to the _"tagged template"_ in a function call like the one below.

```js
fn(['hello ', '. I am', '!'], 'nico', 'confused')
```

You might be confused by the seeming oddity in which the arguments are laid out, but they start to make sense when you think of it this way: for every item in the template array, there's an expression result after it.

## Demystifying Tagged Templates

I wrote an example `normal` method below, and it works _exactly like the default behavior_. This might help you better understand what happens under the hood for template literals.

> If you don't know what `.reduce` does, refer to [MDN][4] or my ["Fun with Native Arrays"][3] article. Reduce is always useful when you're trying to map a collection of values into a single value that can be computed from the collection.

In this case we can reduce the `template` starting from `template[0]` and then reducing all other parts by adding the preceding `expression` and the subsequent `part`.

```js
function normal (template, <mark>...expressions</mark>) {
  return template.reduce((accumulator, part, i) => {
    return accumulator + expressions[i - 1] + part
  })
}
```

The `...expressions` syntax is new in ES6 as well. It's called the [_"rest parameters syntax"_][6], and it'll basically place all the arguments passed to `normal` that come after `template` into a single array. You can try the tagged template as seen below, and you'll notice you get the same output as if you omitted `normal`.

```js
var name = 'nico'
var outfit = 'leather jacket'
var text = <mark>normal</mark>`hello ${name}, you look lovely today in that ${outfit}`
console.log(text)
// <- 'hello nico, you look lovely today in that leather jacket'
```

Now that we've figured out how tagged templates work, what can we do with them? Well, whatever we want. One possible use case might be to make user input uppercase, turning our greeting into something that sounds more satirical -- _I read the result out loud in my head with Gob's voice from Arrested Development, now I'm laughing alone. I've made a huge mistake_.

```js
function upperExpr (template, ...expressions) {
  return template.reduce((accumulator, part, i) => {
    return accumulator + expressions[i - 1].toUpperCase() + part
  })
}
var name = 'nico'
var outfit = 'leather jacket'
var text = <mark>upperExpr</mark>`hello ${name}, you look lovely today in that ${outfit}`
console.log(text)
// <- 'hello NICO, you look lovely today in that LEATHER JACKET'
```

There's obviously much more useful use cases for tagged templates than laughing at yourself. In fact, you could go crazy with tagged templates. A decidedly useful use case would be to sanitize user input in your templates automatically. Given a template where all expressions are considered user-input, we could use [`insane`][5] to sanitize them out of HTML tags we dislike.

```js
import insane from 'insane'
function sanitize (template, ...expressions) {
  return template.reduce((accumulator, part, i) => {
    return accumulator + <mark>insane</mark>(expressions[i - 1]) + part
  })
}
var comment = 'haha xss is so easy <mark><iframe src="http://evil.corp"></iframe></mark>'
var html = <mark>sanitize</mark>`<div>${comment}</div>`
console.log(html)
// <- '<div>haha xss is so easy </div>'
```

_Not so easy now!_

> I can definitely see a future where the only strings I use in JavaScript begin and finish with a backtick.

[1]: /articles/react-jsx-and-es6-the-weird-parts "React, JSX and ES6: The Weird Parts on Pony Foo"
[2]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
[3]: /articles/fun-with-native-arrays#computing-with-reduce-reduceright "Computing with .reduce, .reduceRight on Pony Foo"
[4]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce "Array.prototype.reduce() – MDN"
[5]: https://github.com/bevacqua/insane "bevacqua/insane on GitHub"
[6]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/rest_parameters "Rest parameters in ES6 – MDN"
