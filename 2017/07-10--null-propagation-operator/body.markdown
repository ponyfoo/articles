There's a proposal in stage 1 for the *Null Propagation operator*. In this article we'll take a look at the proposal, which offers an alternative to null checks ad nauseum.

Very often, we want to grab a value deeply, such as in the following bit of code.

```js
const firstName = person.profile.name.firstName
```

The `profile` might not be an object. Or the `name` might not be an object. So we `null`-check all the things!

```js
const firstName = (
  person &&
  person.profile &&
  person.profile.name &&
  person.profile.name.firstName
)
```

Or we use a library like `lodash` to do the checking for us, at least there's less unwarranted complexity on our own bits of code.

```js
import { get } from 'lodash'
const firstName = get(person, ['profile', 'name', 'firstName'])
```

The Null Propagation operator is a native solution to the problem, allowing us to handle these cases by sprinkling our code with question marks. The operator is all of  `?.`, as we'll see in a bit.

```js
const firstName = person.profile<mark>?.</mark>name<mark>?.</mark>firstName
```

Note that the `?.` goes right before the property access. We can think of each `?.` operator as a short circuit where *"if the expression up until this point is `null` or `undefined`, then the whole expression evaluates to `undefined`"*.

```js
const read = person => person.profile<mark>?.</mark>name<mark>?.</mark>firstName
read() // <- Error, because `person` is undefined
read({}) // <- undefined, because of `profile?.`
read({ profile: {} }) // <- undefined, because of `name?.`
read({ profile: { name: {} } }) // <- undefined, because `firstName` is undefined
read({ profile: { name: { firstName: 'Bob' } } }) // <- 'Bob'
```

The operator can come after any expression, including function calls. In the following example we run a regular expression against a string, and if it matches we get back the matched group. Note that even though we're using the object property access expression notation, we have to use `?.[expression]` and can't just use `?[expression]`. This allows the compilers to disambiguate the grammar more easily.

```js
/(\d+)/.exec('abcdef')<mark>?.</mark>[1] // <- undefined
/(\d+)/.exec('abc1234def')<mark>?.</mark>[1] // <- '1234'
```

Using Null Propagation, we could also optionally call functions. In the following example, we have the `person` eat some foods, provided a `person.eat` method exists. Again, the operator remains `?.` to ease the burden on lexical analyzers.

```js
person.eat<mark>?.</mark>(carrot, pasta, apple)
```

If we go back to the earlier example of reading a person's name, and assuming that names are in the form `'First Last'`, we could do the following to get each part of their name, but only if they indeed have a name and only if the name property value may be sliced.

```js
const read = person => person.name<mark>?.</mark>slice<mark>?.</mark>(' ')
read({}) // <- undefined, because `name` doesn't exist
read({ name: 33 }) // <- undefined, because `33` doesn't have a `slice` method
read({ name: 'Uncle Bob' }) // <- ['Uncle', 'Bob']
```

Probably the least useful bit of the proposal is optional constructor invocation, shown in the next snippet. That said, it's a good idea to include this in the proposal as to avoid the drama that came with `new` + `apply` prior to the rest operator<sup>**<a name='new-apply-rest-ref' href='#new-apply-rest'>\*</a>**</sup>.

```js
new Carriage<mark>?.</mark>(horses)
```

The proposal also discusses _write context_, that is, using the null propagation operator `?.` while writing or deleting properties. These kinds of use cases rarely pop up in the wild, so the proposal probably will end up not covering them.

```js
person?.firstName = 'Bob' // only carried out if `person` is not null or undefined
delete person?.lastName // only carried out if `person` is not null or undefined
```

You can find the [proposal document on GitHub][proposal].

<sup>**<a name='new-apply-rest' href='#new-apply-rest-ref'>\*</a>** The rest operator introduced a clean `new Date(...[2017, 6, 17])` syntax. In the fun old days, doing `new` and `apply` on a constructor involved lot more fun stuff than that: `new (Date.bind.apply(Date, [null, 2017, 6, 17]))`.</sup>

[proposal]: https://github.com/claudepache/es-optional-chaining
