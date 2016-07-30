## Generator Functions and Generator Objects

Generators are a new feature in ES6. You declare a _generator function_ which returns generator objects `g` that can then be iterated using any of [`Array.from(g)`][3], [`[...g]`][2], or [`for value of g`][1] loops. Generator functions allow you to declare a special kind of _iterator_. These iterators can suspend execution while retaining their context. We already examined iterators in [the previous article][1] and how their `.next()` method is called once at a time to pull values from a sequence.

Here is an example generator function. Note the `*` after `function`. That's not a typo, that's how you mark a generator function as a _generator_.

```js
function* generator () {
  yield 'f'
  yield 'o'
  yield 'o'
}
```

Generator objects conform to both the _iterable_ protocol and the _iterator_ protocol. This means...

```js
var g = <mark>generator()</mark>
// a generator object g is built using the generator function
typeof g[Symbol.iterator] === 'function'
// it's an iterable because it has an @@iterator
typeof g.next === 'function'
// it's also an iterator because it has a .next method
g[Symbol.iterator]() === g
// the iterator for a generator object is the generator object itself
console.log(<mark>[...g]</mark>)
// <- ['f', 'o', 'o']
console.log(<mark>Array.from(g)</mark>)
// <- ['f', 'o', 'o']
```

<sub>_(This article is starting to sound an awful lot like a Math course...)_</sub>

When you create a generator object _(I'll just call them "generator" from here on out)_, you'll get an _iterator_ that uses the generator to produce its _sequence_. Whenever a `yield` expression is reached, that value is emitted by the iterator and **function execution is suspended**.

Let's use a different example, this time with some other statements mixed in between `yield` expressions. This is a simple generator but it behaves in an interesting enough way for our purposes here.

```js
function* generator () {
  yield 'p'
  console.log('o')
  yield 'n'
  console.log('y')
  yield 'f'
  console.log('o')
  yield 'o'
  console.log('!')
}
```

If we use a `for..of` loop, this will print `ponyfoo!` one character at a time, as expected.

```js
var foo = generator()
for (<mark>let pony of foo</mark>) {
  console.log(pony)
  // <- 'p'
  // <- 'o'
  // <- 'n'
  // <- 'y'
  // <- 'f'
  // <- 'o'
  // <- 'o'
  // <- '!'
}
```

What about using the spread `[...foo]` syntax? Things turn out a little different here. This might be a little unexpected, but that's how generators work, everything that's not yielded ends up becoming **a side effect**. As the sequence is being constructed, the `console.log` statements in between `yield` calls are executed, and they print characters to the console before `foo` is spread over an array. The previous example worked because we were printing characters as soon as they were pulled from the sequence, instead of waiting to construct a range for the entire sequence first.

```js
var foo = generator()
console.log(<mark>[...foo]</mark>)
// <- 'o'
// <- 'y'
// <- 'o'
// <- '!'
// <- ['p', 'n', 'f', 'o']
```

A neat aspect of generator functions is that you can also use `yield*` to delegate to another generator function. Want a very contrived way to split `'ponyfoo'` into individual characters? Since strings in ES6 adhere to the _iterable_ protocol, you could do the following.

```js
function* generator () {
  yield* 'ponyfoo'
}
console.log([...generator()])
// <- ['p', 'o', 'n', 'y', 'f', 'o', 'o']
```

Of course, in the real world you could just do `[...'ponyfoo']`, since spread supports iterables just fine. Just like you could `yield*` a string, you can `yield*` anything that adheres to the iterable protocol. That includes other generators, arrays, and come ES6 -- _just about anything._

```js
var foo = {
  [Symbol.iterator]: () => ({
    items: <mark>['p', 'o', 'n', 'y', 'f', 'o', 'o']</mark>,
    next: function next () {
      return {
        done: this.items.length === 0,
        value: this.items.shift()
      }
    }
  })
}
function* multiplier (value) {
  yield value * 2
  yield value * 3
  yield value * 4
  yield value * 5
}
function* trailmix () {
  yield 0
  yield* [1, 2]
  yield* <mark>[...multiplier(2)]</mark>
  yield* multiplier(3)
  yield* <mark>foo</mark>
}
console.log([...trailmix()])
// <- [0, 1, 2, <mark>4, 6, 8, 10</mark>, 6, 9, 12, 15, <mark>'p', 'o', 'n', 'y', 'f', 'o', 'o'</mark>]
```

You could also iterate the sequence by hand, calling `.next()`. This approach gives you the most control over the iteration, but it's also the most involved. There's a few features you can leverage here that give you even more control over the iteration.

## Iterating Over Generators by Hand

Besides iterating over `trailmix` as we've already covered, using `[...trailmix()]`, `for value of trailmix()`, and `Array.from(trailmix())`, we could use the generator returned by `trailmix()` directly, and iterate over that. But `trailmix` was an overcomplicated showcase of `yield*`, let's go back to the _side-effects_ `generator` for this one.

```js
function* generator () {
  yield 'p'
  console.log('o')
  yield 'n'
  console.log('y')
  yield 'f'
  console.log('o')
  yield 'o'
  console.log('!')
}
var g = generator()
while (true) {
  let item = <mark>g.next()</mark>
  if (item.done) {
    break
  }
  console.log(item.value)
}
```

Just like we [learned yesterday][1], any items returned by an iterator will have a `done` property that indicates whether the sequence has reached its end, and a `value` indicating the current value in the sequence.

> If you're confused as to **why the `'!'` is printed** even though there are no more `yield` expressions after it, that's because `g.next()` doesn't know that. The way it works is that each time its called, it executes the method until a `yield` expression is reached, emits its value and _suspends execution_. The next time `g.next()` is called, _execution is resumed _from where it left off _(the last `yield` expression)_, until the next `yield` expression is reached. When no `yield` expression is reached, the generator returns `{ done: true }`, signaling that the sequence has ended. At this point, the `console.log('!')` statement has been already executed, though.
>
> It's also worth noting that **context is preserved** across suspensions and resumptions. That means generators can be stateful. Generators are, in fact, the underlying implementation for `async`/`await` semantics coming in ES7.

Whenever `.next()` is called on a generator, there's four "events" that will suspend execution in the generator, returning an _`IteratorResult`_ to the caller of `.next()`.

- A `yield` expression returning the _next_ value in the sequence
- A `return` statement returning the _last_ value in the sequence
- A `throw` statement halts execution in the generator entirely
- Reaching the end of the generator function signals `{ done: true }`

Once the `g` generator ended iterating over a sequence, subsequent calls to `g.next()` will have no effect and just return `{ done: true }`.

```js
function* generator () {
  yield 'only'
}
var g = generator()
console.log(g.next())
// <- { done: false, value: 'only' }
console.log(g.next())
// <- { done: true }
console.log(g.next())
// <- { done: true }
```

## Generators: The <del>Weird</del> <ins>_Awesome_</ins> Parts

Generator objects come with a couple more methods besides [`.next`][7]. These are [`.return`][6] and [`.throw`][5]. We've already covered `.next` extensively, but not quite. You could also use `.next(value)` to send values _into the generator_.

Let's make **a magic 8-ball generator**. First off, you'll need some answers. Wikipedia obliges, yielding [20 possible answers][8] for our magic 8-ball.

```js
var answers = [
  `It is certain`, `It is decidedly so`, `Without a doubt`,
  `Yes definitely`, `You may rely on it`, `As I see it, yes`,
  `Most likely`, `Outlook good`, `Yes`, `Signs point to yes`,
  `Reply hazy try again`, `Ask again later`, `Better not tell you now`,
  `Cannot predict now`, `Concentrate and ask again`,
  `Don't count on it`, `My reply is no`, `My sources say no`,
  `Outlook not so good`, `Very doubtful`
]
function answer () {
  return answers[Math.floor(Math.random() * answers.length)]
}
```

The following generator function can act as a _"genie"_ that answers any questions you might have for them. Note how we discard the first result from `g.next()`. That's because the first call to `.next` enters the generator and there's no `yield` expression waiting to capture the `value` from `g.next(value)`.

```js
function* chat () {
  while (true) {
    let question = yield '[Genie] ' + answer()
    console.log(question)
  }
}
var g = chat()
<mark>g.next()</mark>
console.log(g.next('[Me] Will ES6 die a painful death?').value)
// <- '[Me] Will ES6 die a painful death?'
// <- '[Genie] My sources say no'
console.log(g.next('[Me] How youuu doing?').value)
// <- '[Me] How youuu doing?'
// <- '[Genie] Concentrate and ask again'
```

Randomly dropping `g.next()` feels like a very dirty coding practice, though. What else could we do? We could flip responsibilities around.

### Inversion of Control

We could have the Genie be in control, and have the generator ask the questions. How would that look like? At first, you might think that the code below is unconventional, but in fact, most libraries built around generators work by inverting responsibility.

```js
function* chat () {
  yield '[Me] Will ES6 die a painful death?'
  yield '[Me] How youuu doing?'
}
var g = chat()
while (true) {
  let question = g.next()
  if (question.done) {
    break
  }
  console.log(question.value)
  console.log('[Genie] ' + answer())
  // <- '[Me] Will ES6 die a painful death?'
  // <- '[Genie] Very doubtful'
  // <- '[Me] How youuu doing?'
  // <- '[Genie] My reply is no'
}
```

You would expect the **generator to do the heavy lifting** of an iteration, but in fact generators make it easy to iterate over things by suspending execution of themselves -- and deferring the heavy lifting. That's one of the most powerful aspects of generators. Suppose now that the iterator is a `genie` method in a library, like so:

```js
function genie (questions) {
  var g = questions()
  while (true) {
    let question = g.next()
    if (question.done) {
      break
    }
    console.log(question.value)
    console.log('[Genie] ' + answer())
  }
}
```

To use it, all you'd have to do is pass in a simple generator like the one we just made.

```js
genie(function* questions () {
  yield '[Me] Will ES6 die a painful death?'
  yield '[Me] How youuu doing?'
})
```

Compare that to the generator we had before, where questions were sent to the generator instead of the other way around. See how much more complicated the logic would have to be to achieve the same goal? Letting the library deal with the flow control means you can **just worry about the _thing_ you want to iterate** over, and you can **delegate _how_ to iterate over it**. But yes, it does mean your code now has an asterisk in it. _Weird._

### Dealing with asynchronous flows

Imagine now that the `genie` library gets its magic 8-ball answers from an API. How does that look then? Probably something like the snippet below. Assume the [`xhr`][9] pseudocode call always yields JSON responses like `{ answer: 'No' }`. Keep in mind this is a simple example that just processes each question in series. You could put together different and more complex flow control algorithms depending on what you're looking for.

This is just a demonstration of the sheer power of generators.

```js
function genie (questions) {
  var g = questions()
  pull()
  function pull () {
    let question = <mark>g.next()</mark>
    if (question.done) {
      return
    }
    ask(question.value, pull)
  }
  function ask (q, next) {
    <mark>xhr('https://computer.genie/?q=' + encodeURIComponent(q), got)</mark>
    function got (err, res, body) {
      if (err) {
        // todo
      }
      console.log(q)
      console.log('[Genie] ' + body.answer)
      next()
    }
  }
}
```

<sub>See [this link for a live demo][10] on the Babel REPL</sub>

Even though we've just made our `genie` method asynchronous and are now using an API to fetch responses to the user's questions, the way the consumer uses the `genie` library by passing a `questions` generator function *remains unchanged!* That's awesome.

We haven't handled the case for an `err` coming out of the API. That's inconvenient. What can we do about that one?

### Throwing _at_ a Generator

Now that we've figured out that the most important aspect of generators is _actually the control flow code_ that decides when to call `g.next()`, we can look at the other two methods and actually understand their purpose. Before shifting our thinking into _"the generator defines **what** to iterate over, not the **how**"_, we would've been hard pressed to find a user case for [`g.throw`][5]. Now however it seems immediately obvious. The flow control that leverages a generator needs to be able to tell the generator that's yielding the sequence to be iterated when something goes wrong processing an item in the sequence.

In the case of our `genie` flow, that is now using [`xhr`][9], we may experience network issues and be unable to continue processing items, or we may want to warn the user about unexpected errors. Here's how, we simply add `g.throw(error)` in our control flow code.

```js
function genie (questions) {
  var g = questions()
  pull()
  function pull () {
    let question = g.next()
    if (question.done) {
      return
    }
    ask(question.value, pull)
  }
  function ask (q, next) {
    xhr('https://computer.genie/?q=' + encodeURIComponent(q), got)
    function got (err, res, body) {
      if (err) {
        <mark>g.throw(err)</mark>
      }
      console.log(q)
      console.log('[Genie] ' + body.answer)
      next()
    }
  }
}
```

The _user code_ is still unchanged, though. In between `yield` statements it may throw errors now. You could use `try`/`catch` blocks to address those issues. If you do this, execution will be able to resume. The good thing is that this is up to the user, it's still perfectly sequential on their end, and they can leverage `try`/`catch` semantics just like in high-school.

```js
genie(function* questions () {
  try {
    yield '[Me] Will ES6 die a painful death?'
  } catch (e) {
    console.error('Error', e.message)
  }
  try {
    yield '[Me] How youuu doing?'
  } catch (e) {
    console.error('Error', e.message)
  }
})
```

### Returning on Behalf of a Generator

Usually not as interesting in asynchronous control flow mechanisms in general, the [`g.return()`][6] method allows you to resume execution inside a generator function, much like [`g.throw()`][5] did moments earlier. The key difference is that `g.return()` won't result in an exception at the generator level, although **it will end the sequence.**

```js
function* numbers () {
  yield 1
  yield 2
  yield 3
}
var g = numbers()
console.log(g.next())
// <- { done: false, value: 1 }
console.log(<mark>g.return()</mark>)
// <- { done: true }
console.log(g.next())
// <- { done: true }, <mark>as we know</mark>
```

You could also return a `value` using `g.return(value)`, and the resulting `IteratorResult` will contain said `value`. This is equivalent to having `return value` somewhere in the generator function. You should be careful there though -- as neither `for..of`, `[...generator()]`, nor `Array.from(generator())` include the `value` in the `IteratorResult` that signals `{ done: true }`.

```js
function* numbers () {
  yield 1
  yield 2
  <mark>return 3</mark>
  yield 4
}
console.log([...numbers()])
// <- <mark>[1, 2]</mark>
console.log(Array.from(numbers()))
// <- [1, 2]
for (let n of numbers()) {
  console.log(n)
  // <- 1
  // <- 2
}
var g = numbers()
console.log(g.next())
// <- { done: false, value: 1 }
console.log(g.next())
// <- { done: false, value: 2 }
console.log(g.next())
// <- { done: true, <mark>value: 3</mark> }
console.log(g.next())
// <- { done: true }
```

Using `g.return` is no different in this regard, think of it as the programmatic equivalent of what we just did.

```js
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}
var g = numbers()
console.log(g.next())
// <- { done: false, value: 1 }
console.log(g.return(5))
// <- { done: true, value: 5 }
console.log(g.next())
// <- { done: true }
```

You can avoid the impending sequence termination, [as Axel points out][4], if the code in the generator function when `g.return()` got called is wrapped in `try`/`finally`. Once the `yield` expressions in the `finally` block are over, the sequence _will_ end with the `value` passed to `g.return(value)`

```js
function* numbers () {
  yield 1
  try {
    yield 2
  } finally {
    yield 3
    yield 4
  }
  yield 5
}
var g = numbers()
console.log(g.next())
// <- { done: false, value: 1 }
console.log(g.next())
// <- { done: false, value: 2 }
console.log(<mark>g.return(6)</mark>)
// <- { done: false, value: 3 }
console.log(g.next())
// <- { done: false, value: 4 }
console.log(g.next())
// <- { done: true, <mark>value: 6</mark> }
```

That's all there is to know when it comes to generators _in terms of functionality._

## Use Cases for ES6 Generators

At this point in the article you should feel comfortable with the concepts of iterators, iterables, and generators in ES6. If you feel like reading more on the subject, I highly recommend you go over [Axel's article on generators][4], as he put together an amazing write-up on use cases for generators just _a few months ago_.

[1]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[2]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from "Array.from() on MDN"
[4]: http://www.2ality.com/2015/03/es6-generators.html "ES6 generators in depth by Dr. Axel Rauschmayer"
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/throw "Generator.prototype.throw() on MDN"
[6]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/return "Generator.prototype.return() on MDN"
[7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next "Generator.prototype.next() on MDN"
[8]: https://en.wikipedia.org/wiki/Magic_8-Ball#Possible_answers "Magic 8 Ball Possible Answers on Wikipedia"
[9]: https://github.com/Raynos/xhr "Raynos/xhr on GitHub"
[10]: http://buff.ly/1UimWsZ "Babel REPL of async generator for genie responses"
