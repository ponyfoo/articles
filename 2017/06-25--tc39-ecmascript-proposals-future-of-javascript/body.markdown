TC39 follows a process to develop language features that's based on maturity stages. Once a proposal is mature enough, TC39 updates the specification with the changes presented in the proposal. Up until recently, TC39 relied on an older flow based on Microsoft Word. But after ES3 came out, they spent **ten years with virtually no changes** making their way to the specification. After that, it took them *another four years* for ES6 to materialize.

> It became evident that their process had to improve.

Since ES6 came out, they streamlined the proposal revisioning process to meet modern expectations. The new process uses a superset of HTML to format the proposals. They use [GitHub pull requests][gh], which helped boost participation from the community and the number of proposals being made also increased. The specification is now more of a [living standard][ls], meaning that proposals see adoption faster, and we don't spend years waiting for a new edition of the specification to come out.

The new process involves four different maturity stages. The more mature a proposal is, the more likely it is to eventually make it into the specification.

# Stage 0

Any discussion, idea, change, or addition which has not yet been submitted as a formal proposal is considered to be a "strawman" proposal at Stage 0. Only members of TC39 can create these proposals, and there's over a dozen active strawman proposals today.

Proposals currently in Stage 0 include [cancellation tokens][cancel-token] for asynchronous operations, [Zones][zones] as the ones originally hailed by the Angular team, along with many proposals that never made it into Stage 1.

Later in this article, we'll take a closer look at individual proposals.

# Stage 1

At Stage 1 a proposal is formalized and expected to address cross-cutting concerns, interactions with other proposals, and implementation concerns. Proposals in this stage identify a discrete problem and offer a concrete solution to that problem.

A Stage 1 proposal often includes a high level API description, usage examples and a discussion of internal semantics and algorithms. These proposals are _likely to change significantly_ as they make their way through the process.

Examples of proposals currently in Stage 1 include: [Observable][obs], [`do`][do-expr] expressions, generator arrow functions, and [`Promise.try`][promise-try].

# Stage 2

Proposals in Stage 2 should offer an initial draft of the specification.

At this point, it's reasonable for implementers to begin experimenting with actual implementations in runtimes. The implementation could come in the form of a polyfill, user code that mangles the runtime into adhering to the proposal; an engine implementation, which natively provides support for the proposal; or it could be support by a build-time compiler like Babel.

In Stage 2 we currently have [public class fields][public-fields], [private class fields][private-fields], [decorators][decorators], and [`Promise#finally`][promise-finally], to name a few.

# Stage 3

Proposals in Stage 3 are candidate recommendations. At this advanced stage, the specification editor and designated reviewers must have signed off on the final specification. A Stage 3 proposal is unlikely to change beyond fixes to issues identified in the wild.

Implementors should have expressed interest in the proposal as well --- a proposal without support from implementors is dead in the water. In practice, proposals move to this level with at least one browser implementation, a high-fidelity polyfill or when supported by a build-time transpiler like Babel.

Stage 3 has exciting features like [object rest and spread][object-rest], [asynchronous iteration][async-iter], the [`import()`][dynamic-import] method, and [better Unicode support][regex] for regular expressions.

# Stage 4

Finally, proposals get to Stage 4 when there are at least two independent implementations that pass acceptance tests.

Proposals that make their way through to stage four will be included in the next revision of ECMAScript.

[Async functions][async-await], [`Array#includes`][array-includes], and the [exponentiation operator][exp] are some examples that made it to stage 4 since the revision process was overhauled.

# Staying Up To Date

I made a website that shows a list of currently active proposals. It describes what stage they're in and links to each proposal so that you can learn more about them.

It lives at [prop-tc39.now.sh][pt].

New formal specification releases are now expected every year, but the streamlined process also means formal releases are becoming less relevant. The focus is now on proposal stages, and we can expect references to specific revisions of the standard to become uncommon after ES6.

# Proposals

Let's look at some of the most interesting proposals that are currently in development.

## `Array#includes` (Stage 4)

Before `Array#includes` was introduced, we had to rely on `Array#indexOf` and checking whether the index was out of bounds to figure out whether an element belonged to an array.

With `Array#includes` now in stage 4, we can use that instead. It complements `Array#find` and `Array#findIndex`, which were introduced in ES6.

```js
[1, 2].indexOf(2) !== -1 // true
[1, 2].indexOf(3) !== -1 // false
[1, 2].includes(2) // true
[1, 2].includes(3) // false
```

## Async Functions (Stage 4)

When working with promises, we often think in terms of execution threads where we have an async task, like `fetch`, and other tasks which depend on the response, but are blocked until that data is received.

In the following example we're fetching a list of products from our API, which returns a `Promise`. This promise will resolve with the response to our request. We then read the response stream as JSON and update a view with data from the response. If any errors happened during this process, we could log them to the console, to understand what's going on.

```js
fetch('/api/products')
  .then(response => response.json())
  .then(data => {
    updateView(data)
  })
  .catch(err => {
    console.log('Update failed', err)
  })
```

Async functions are sugar that can be used to improve how we write Promise-based code. Let's start transforming our promise-based code line-by-line. We can prefix any expression using the `await` keyword. When we `await` on a promise, we get an expression that evaluates to that promise's fulfillment value.

Promises gave a meaning to our code that was like "I want to run this operation, and _then_ I want to use its result within this other operation".

Meanwhile, `await` effectively inverts that meaning, making it more like "I want to get back the result of this operation", which I like, because it sounds simpler.

In our example, the response object is what we're after, so we'll flip things over and assign the result of `await fetch` to the `response` variable, instead of using a promise reaction.

```diff
+ const response = await fetch('/api/products')
- fetch('/api/products')
    .then(response => response.json())
    .then(data => {
      updateView(data)
    })
    .catch(err => {
      console.log('Update failed', err)
    })
```

We give `response.json()` the same treatment. We `await` on its promise and assign that to the `data` variable.

```diff
  const response = await fetch('/api/products')
+ const data = await response.json()
-   .then(response => response.json())
    .then(data => {
      updateView(data)
    })
    .catch(err => {
      console.log('Update failed', err)
    })
```

Now that the reactions are gone, `updateView` is its own statement, since we don't need to await on any other promises, given we've reached the end of our old promise chain.

```diff
  const response = await fetch('/api/products')
  const data = await response.json()
+ updateView(data)
-   .then(data => {
-     updateView(data)
-   })
    .catch(err => {
      console.log('Update failed', err)
    })
```

We can now just use `try`/`catch` blocks instead of the `.catch` reaction we used in the promise-based code, leading us to more semantic code.

```diff
+ try {
    const response = await fetch('/api/products')
    const data = await response.json()
    updateView(data)
+ } catch(err) {
- .catch(err => {
    console.log('Update failed', err)
+ }
- )}
```

One limitation is that `await` only works inside async functions.

```diff
+ async function run() {
    try {
      const response = await fetch('/api/products')
      const data = await response.json()
      updateView(data)
    } catch(err) {
      console.log('Update failed', err)
    }
+ }
```

We could, however, turn our async function into a self-invoking function expression. If we wrap our top-level code in an expression like this, we can use `await` expressions anywhere in our codebase.

Some of the community wants native top level await, while some others think it would have a negative effect in user-land, making it all too easy for libraries to block on asynchronous operations while loading, considerably slowing down the load time of our applications.

```diff
+ (async () => {
- async function run() {
    try {
      const response = await fetch('/api/products')
      const data = await response.json()
      updateView(data)
    } catch(err) {
      console.log('Update failed', err)
    }
+ })()
- }
```

Personally, I think there's more than enough room already to do silly things in the JavaScript performance space, and libraries that block initialization with `await` will never thrive and become popular.

Note that you could also `await` on non-promise values, even writing code like `await (2 + 3)`. In this case, the result of the `(2 + 3)` expression is boxed in a promise, and that promise's fulfillment value, which is `5`, becomes the result of the `await` expression.

Note that `await` plus any JavaScript expression is also an expression. This means we aren't limited to awaiting for values that get assigned to variables, but that we can also, for instance, `await` on a function call as part of a template literal interpolation.

```js
`Price: ${ await getPrice() }`
```

Or as part of another function call…

```js
renderView(await getPrice())
```

Or even as part of a mathematical equation.

```js
2 * (await getPrice())
```

Finally, regardless of their contents, async functions **always return a promise**. This means we can add `.then` or `.catch` reactions to an async function, and it also means we can `await` on its result.

```js
const sleep = delay => new Promise(resolve =>
  setTimeout(resolve, delay)
)
const slowLog = async (...terms) => {
  await sleep(2000)
  console.log(...terms)
}
slowLog('Well that was underwhelming')
  .then(() => console.log('Nailed it!'))
  .catch(reason => console.error('Failed', reason))
```

As you would expect, the returned promise settles with the value returned from the async function, or is rejected with any uncaught exceptions that arised inside the async function.

## Async Iteration (Stage 3)

Moving onto Stage 3, we have async iteration. Before diving into it, let's talk briefly about [iterables][iterable], which were introduced in ES6. An iterable can be any object which adheres to the iterator protocol.

To make an object iterable, we define a `Symbol.iterator` method. The iterator method should return an object that has a `next` method. This object describes the sequence for our iterable. When an object is being iterated, the `next` method will be called each time we need to read the next element in the sequence. Each `value` from the returned objects is used to construct the sequence. When the returned object is marked as `done`, the sequence ends.

```js
const list = {
  <mark>[Symbol.iterator]</mark>() {
    let i = 0
    return {
      next: () => ({
        value: i++,
        done: i > 5
      })
    }
  }
}
[...list]
// <- [0, 1, 2, 3, 4]
Array.from(list)
// <- [0, 1, 2, 3, 4]
for (const i of list) {
  // <- 0, 1, 2, 3, 4
}
```

Iterables can be consumed all at once with `Array.from` or by using the spread operator. They can also be consumed element by element using a `for..of` loop.

Async iterators are only a little bit different. Under this proposal, an object can use `Symbol.asyncIterator` to advertise that they are iterable asynchronously. The contract for an async iterator is only slightly different from that for a regular iterator: the `next` method needs to return a `Promise` for a `{ value, done }` pair instead of returning `{ value, done }` directly.

```js
const list = {
  [Symbol.asyncIterator]() {
    let i = 0
    return {
      next: () => Promise.resolve({
        value: i++,
        done: i > 5
      })
    }
  }
}
```

This simple change is sufficient and elegant, since promises can easily represent the eventual elements of the sequence.

An async iterable can't be consumed with the array spread operator, nor with `Array.from`, nor with `for..of`, since all three were exclusively purpose-built for synchronous iteration.

This proposal introduces a new `for await..of` construct as well. It can be used to semantically iterate over an asynchronously iterable sequence.

```js
for await (const i of items) {
  // <- 0, 1, 2, 3, 4
}
```

Note that the `for await..of` construct can only be used inside async functions. Otherwise we'll get syntax errors. Just like with any other async functions, we could also use `try`/`catch` blocks around or inside our `for await..of` loops.

```js
async function readItems() {
  for await (const i of items) {
    // <- 0, 1, 2, 3, 4
  }
}
```

The rabbit hole goes deeper of course. There's also async generator functions. These are somewhat similar to plain generator functions, except async generator functions support `async` `await` semantics, allowing `await` statements as well as `for await..of`.

```js
async function* getProducts(categoryUrl) {
  const listReq = await fetch(categoryUrl)
  const list = await listReq.json()
  for (const product of list) {
    const productReq = await fetch(product.url)
    const product = await productReq.json()
    yield product
  }
}
```

Inside async generator functions, we can use `yield*` with other async generators and with plain generators as well. When invoked, async generator functions return async generator objects, whose methods return promises for `{ value, done }` pairs, instead of the plain `{ value, done }` pairs returned by plain generators.

Finally, an async generator object can be consumed with `for await..of`, just like an async iterable. This is because async generator objects are async iterables, just like plain generator objects are plain iterables.

```js
async function readProducts() {
  const g = getProducts(category)
  for await (const product of g) {
    // use product details
  }
}
```

## Object Rest and Spread (Stage 3)

Starting with ES6, we can use `Object.assign` to copy the properties from one or more source objects onto one target object. In the next example we're copying a few properties onto an empty object, and getting that same object back.

```js
Object.assign(
 {},
 { a: 'a' },
 { b: 'b' },
 { a: 'c' }
)
```

The object spread proposal allows us to write equivalent code using plain syntax. We start with an empty object where `Object.assign` is implicitly buried in the syntax. Every object we spread onto an object literal acts as a source for assigning its own properties to that object literal.

```js
{
 ...{ a: 'a' },
 ...{ b: 'b' },
 ...{ a: 'c' }
}
// <- { a: 'c', b: 'b' }
```

There's also a rest counterpart to object spread, just like with spread in arrays and rest parameters. When destructuring an object, we can use the object spread operator to destructure any own properties not explicitly named in the pattern into another object.

In the following example, the `id` is explicitly named and will not be included in the rest object. Object rest can be read literally as _"everything else goes to an object named rest"_, and of course, the variable name is for you to choose.

```js
const item = {
 id: '4fe09c27',
 name: 'Banana',
 amount: 3
}
const { id, ...rest } = item
// <- { name: 'Banana', amount: 3 }
```

When destructuring an object in a function's parameter list, we can use object rest as well.

```js
function print({ id, ...rest }) {
  console.log(rest)
}
print({ id: '4fe09c27', name: 'Banana' })
// <- { name: 'Banana' }
```

## Dynamic `import()` (Stage 3)

ES6 introduced native JavaScript modules. Unlike CommonJS and similar, JavaScript modules opted for static statements. Tooling has an easier time analyzing and building dependency trees out of static source code, which makes it a great default.

```
import markdown from './markdown'
// …
export default compile
```

However, as developers we don't always know the modules we need to import ahead of time. For these cases, such as when we depend on localization to load a module with strings under the user's language, the dynamic `import()` proposal in Stage 3 comes into play.

Dynamic `import()` loads modules at runtime. It returns a promise for the module's namespace object, which resolves after fetching, parsing, and executing the requested module and all of its dependencies. If the module fails to load, the promise will be rejected.

```js
import(`./i18n.${ navigator.language }.js`)
  .then(module => console.log(module.messages))
  .catch(reason => console.error(reason))
```

## Named Captures (Stage 3)

Regular expressions are not that hard to write, but they're many times harder to read. The expression on the next bit of code can be used to capture parts of a URL. I took the liberty of adding non-normative spaces and line breaks so that the expression isn't as daunting to read, feel free to remove these if you want to try out the regular expression.

```js
const urlRegExp = /
  ^
  (?:(http[s]?|ftp):\/)?
  \/?
  ([^:\/\s]+)
  ((?:\/\w+)*\/)
  ([\w\-\.]+[^#?\s]+)
  ([^#]*)?
  (#[\w\-]+)?
  $
/
```

Which parts are captured? Well, you'll probably have to try a few matches to figure that out. Or maybe you can put the regular expression through [a tool that tells you what it does][regexper].

The named captures proposal allows us to name capture groups so that expressions become a little bit easier to read and use.

In the following piece, the highlighted parts are the names I gave to each interesting capture group in the expression. Note I had to turn on the Unicode flag `/u` in order to use named captures.

```js
const urlRegExp = /
  ^
  (?:(<mark>?<protocol></mark>http[s]?|ftp):\/)?
  \/?
  (<mark>?<host></mark>[^:\/\s]+)
  (<mark>?<path></mark>(?:\/\w+)*\/)
  (<mark>?<file></mark>[\w\-\.]+[^#?\s]+)
  (<mark>?<query></mark>[^#]*)?
  (<mark>?<hash></mark>#[\w\-]+)?
  $
/u
```

Reading the expression is a little better now that the important groups have names.

When matching against a regular expression, the resulting array will now also contain a `groups` property with key/value pairs matching each of the groups named in the regular expression.

```js
const url = 'https://commits.com/8b48e3/diff?w=1#readme'
const { groups } = urlRegExp.exec(url)
```

The snippet shown above produces the following object:

```js
{
  protocol: 'https',
  host: 'commits.com',
  path: '/8b48e3/',
  file: 'diff',
  query: '?w=1',
  hash: '#readme'
}
```

When doing `String#replace`, we can use named capture groups instead of numbered capture groups, making our code easier to follow than if we used indices. Our code is now less brittle too, because new capturing groups might change the indices of the captures we care about, they won't affect their names.

```js
const url = 'https://commits.com/8b48e3/diff?w=1#readme'
const pattern = '$<protocol>://github.com/$<file>'
const replaced = url.replace(urlRegExp, pattern)
console.log(replaced)
// <- 'https://github.com/diff'
```

Named captures can be reused to capture the same pattern later in the same regular expression, just like we could do with numbered backreferences.

```js
const duplicateRegExp = /^(.*)=\1$/
const duplicateRegExp = /^(?<thing>.*)=\k<thing>$/u
duplicateRegExp.test('a=b') // <- false
duplicateRegExp.test('a=a') // <- true
duplicateRegExp.test('aa=a') // <- false
duplicateRegExp.test('bbb=bbb') // <- true
```

## Unicode Escapes (Stage 3)

The Unicode escapes proposal adds a pattern to test whether the input has a certain Unicode property value. It can be used to match certain Unicode properties of a symbol, such as the Script that symbol belongs to.

The examples in the next piece of code show how you can use the lowercase `\p` escape to test whether `π` is a greek symbol. The uppercase `\P` escape negates the condition: in the example, it matches everything except for greek symbols.

```js
/^\p{Script=Greek}$/u.test('π')
// <- true
/^\P{Script=Greek}$/u.test('π')
// <- false
```

## Lookbehind Assertions (Stage 3)

Lookbehind assertions test whether a pattern is matched to the left of the current position. They look like the highlighted patterns in the code snippet, which test against the Yuan symbol.

```js
/\d+/.test('¥1245') // <- true
/<mark>(?<=¥)</mark>\d+/.test('¥1245') // <- true
/<mark>(?<=¥)</mark>\d+/.test('$1245') // <- false
/<mark>(?<!¥)</mark>\d+/.test('¥1245') // <- false
/<mark>(?<!¥)</mark>\d+/.test('$1245') // <- true
```

The less than means this is a lookbehind expression and not a lookahead.
The equals sign means this is a _positive_ lookbehind assertion. If the pattern is matched then the regular expression will match.
In the first example the Yuan is matched in the input, so the positive lookbehind assertion succeeds. In the second, the Yuan is not matched and the assertion fails.

When the equals sign is an exclamation point instead _-- like in the third and fourth highlighted examples --_ then the assertion would be a _negative_ lookbehind. It will match when the pattern is not matched.

## Class Decorators (Stage 2)

Decorators are in Stage 2. They can be applied to classes or to statically defined properties of classes.

```js
@pure
@decorators.elastic()
class View {
  @throttle(200)
  reconcile() {
  }
}
```

Decorators are implemented as a function and can be used to make a property readonly or wrap a method with error-logging.

Here we have a `readonly` decorator which simply transforms the descriptor to be nonwritable. When we decorate a property as `@readonly` it will become non-writable.

```js
function readonly({ descriptor, ...rest }) {
  return {
    ...rest,
    descriptor: {
      ...descriptor,
      writable: false
    }
  }
}
```

## `Promise#finally` (Stage 2)

Finally, the last proposal we'll discuss is `Promise#finally`. This proposal is very simple. It helps us avoid repetition when we want to run a callback after a promise settles, regardless of whether it resolved or was rejected.

We can think of `Promise#finally(fn)` as the equivalent of `Promise#then(fn, fn)`, except `Promise#finally` does not receive any arguments.

```diff
  showSpinner()
  fetch(productUrl)
    .then(renderProduct)
+   .finally(
-   .then(
-     hideSpinner,
      hideSpinner
    )
 ```

# Future of JavaScript

TC39 is currently working on over 30 active proposals. What else does the future hold in store?

These days we download our packages from npm. We use webpack to manage the complexity of our applications. We use Babel to get the latest language features. We use tools like `uglifyjs` and `rollup` to optimize the size of our payloads. We use `eslint` and `prettier` to uphold code quality and a consistent coding style. We use Node.js and Electron to run our JavaScript code everywhere.

## Transpilation and the ECMAScript Standard

All of ES6 will soon be available in the majority of runtime engines, but thanks to Babel we've been using ES6 features for a long time already. Babel has long started evolving away from being just an ES6 compiler: today, you can play around with most late stage proposals thanks to Babel plugins. As browser support for ES6 becomes more prominent, we can expect Babel to stop transpiling ES6 features. By that time, we'll already be using newer features like async functions or class decorators, that will still need transpiling.

In this sense, we can think of transpilation as a moving window where we transpile only the absolute necessary in order to maximize browser support for production. A key aspect of modern web development is evergreen browsers, which auto-update. Auto-updating browsers keep Babel thin. As browsers rush to implement the latest features, there's less code for Babel to transpile. However, Babel still plays a key role as well, by providing easy access to proposals while they're in development.

This simplifies the feedback loop between practising web developers and implementors, preventing proposals from being developed in a vacuum.

## Linting and Code Quality

In the past we had linters like JSLint and JSHint, which were a little too concerned with enforcing a coding style. ESLint arose as a solution which is entirely configurable, allowing us to control exactly which aspects of our codebase we want linted in addition to syntax errors.

In the future, we can expect more innovative tools like Prettier, which automatically formats our code to follow a certain coding style, making it consistent throughout the codebase.

## Bundling, Bloat, and complexity

By bringing true CommonJS modules to the browser, Browserify reshaped front-end development. Webpack won over browserify by offering a wealth of features like automated code splitting and the ability to manage CSS or images, becoming not just a JavaScript bundler but the bundler for all front-end assets.

This centralization is interesting because webpack makes it easy to build bloated apps but is also uniquely positioned to combat the web of bloat.
In the future, I wish webpack becomes as simple as some other tools we've discussed, or is replaced by a simpler tool.

## Experimentation

Last, we have [Prepack][prepack], a new tool from Facebook. It is quite unique in that it's a full-blown JavaScript interpreter specifically aimed to reducing the amount of indirection in our code.

Its goal is to reduce initialization costs by precomputing code away during the build step. For example, we might originally have the following piece of code:

```js
(function () {
  function fibonacci(x) {
    return x <= 1 ? x : fibonacci(x - 1) + fibonacci(x - 2)
  }
  global.x = fibonacci(23)
})()
```

Prepack would interpret our code using a full-blown interpreter during our build, and produce code like the following:

```js
(function () {
  x = 28657
})()
```

Prepack might eventually become the gold standard, but for now it's just an experiment.

# Resources

If you'd like to learn more about the things I've discussed in this article, here are some links!

- [ES6 Overview in 350 Bullet Points][es6]
- [Understanding JavaScript's `async`  `await`][async-await]
- [Regular Expressions in a post-ES6 World][regex]
- [The JavaScript Standard][standard]
- [JavaScript Asynchronous Iteration Proposal][async-iter]
- [Practical Modern JavaScript][pmj]
- [`prop-tc39`][pt]
- [`tc39/proposals`][gh]
- [Prepack][prepack]
- [@nzgb][tw]

Thanks for tuning in!

[ls]: https://tc39.github.io/ecma262 "The Living Standard is at tc39.github.io/ecma262"
[gh]: https://github.com/tc39/proposals "tc39/proposals on GitHub"
[tw]: https://twitter.com/nzgb
[es6]: /articles/es6
[async-await]: /articles/understanding-javascript-async-await
[async-iter]: /articles/javascript-asynchronous-iteration-proposal
[regex]: /articles/regular-expressions-post-es6
[pmj]: /books/practical-modern-javascript/chapters
[pt]: https://prop-tc39.now.sh
[standard]: /articles/standard
[cancel-token]: https://github.com/tc39/proposal-cancellation
[zones]: https://github.com/domenic/zones
[obs]: /articles/observables-coming-to-ecmascript "Observables Proposal for ECMAScript! on Pony Foo"
[do-expr]: /articles/proposal-statements-as-expressions-using-do "Proposal: “Statements as Expressions” using do on Pony Foo"
[promise-try]: https://github.com/tc39/proposal-promise-try
[public-fields]: https://github.com/tc39/proposal-class-fields
[private-fields]: https://medium.com/the-thinkmill/javascripts-new-private-class-fields-93106e37647a
[decorators]: /articles/javascript-decorators-proposal "ECMAScript Proposal for JavaScript Decorators on Pony Foo"
[promise-finally]: https://github.com/tc39/proposal-promise-finally
[object-rest]: https://github.com/tc39/proposal-object-rest-spread
[dynamic-import]: https://github.com/tc39/proposal-dynamic-import
[array-includes]: https://github.com/tc39/Array.prototype.includes/
[exp]: https://github.com/rwaldron/exponentiation-operator
[iterable]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[regexper]: http://bit.ly/2rPIotw "Regexper is my favorite tool to explore regular expressions"
[prepack]: https://prepack.io/
