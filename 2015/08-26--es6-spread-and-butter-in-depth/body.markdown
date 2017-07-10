## Rest parameters

You know how sometimes there's a ton of arguments and you end up having to use the `arguments` magic variable to work with them? Consider the following method that joins any arguments passed to it as a string.

```js
function concat () {
  return Array.prototype.slice.call(arguments).join(' ')
}
var result = concat('this', 'was', 'no', 'fun')
console.log(result)
// <- 'this was no fun'
```

The rest parameters syntax enables you to pull a real `Array` out of the `function`'s arguments by adding a parameter name prefixed by `...`. Definitely simpler, the fact that it's a real `Array` is also very convenient, and I for one am glad not to have to resort to `arguments` anymore.

```js
function concat (...words) {
  return words.join(' ')
}
var result = concat('this', 'is', 'okay')
console.log(result)
// <- 'this is okay'
```

When you have more parameters in your `function` it works slightly different. Whenever I declare a method that has a rest parameter, I like to think of its behavior as follows.

- Rest parameter gets all the `arguments` passed to the function call
- Each time a parameter is added on the left, it's as if its value is assigned by calling `rest.shift()`
- Note that you can't actually place parameters to the right: rest parameters can only be the last argument

It's easier to visualize how that would behave than try to put it into words, so let's do that. The method below computes the `sum` for all `arguments` except the first one, which is then used as a `multiplier` for the `sum`. In case you don't recall, `.shift()` returns the first value in an array, and also removes it from the collection, which makes it a useful mnemonic device in my opinion.

```js
function sum () {
  var numbers = Array.prototype.slice.call(arguments) // numbers gets all arguments
  <mark>var multiplier = numbers.shift()</mark>
  <mark>var base = numbers.shift()</mark>
  var sum = numbers.reduce((accumulator, num) => accumulator + num, base)
  return multiplier * sum
}
var total = sum(2, 6, 10, 8, 9)
console.log(total)
// <- 66
```

Here's how that method would look if we were to use the rest parameter to pluck the numbers. Note how we don't need to use `arguments` nor do any shifting anymore. This is great because it vastly reduces the complexity in our method -- which now can focus on its functionality itself and not so much on rebalancing `arguments`.

```js
function sum (multiplier, base, <mark>...numbers</mark>) {
  var sum = numbers.reduce((accumulator, num) => accumulator + num, base)
  return multiplier * sum
}
var total = sum(2, 6, 10, 8, 9)
console.log(total)
// <- 66
```

## Spread Operator

Typically you invoke a function by passing arguments into it.

```js
console.log(1, 2, 3)
// <- '1 2 3'
```

Sometimes however you have those arguments in a list and just don't want to access every index just for a method call _-- or you just can't because the array is formed dynamically --_ so you use `.apply`. This feels kind of awkward because `.apply` also takes a context for `this`, which feels out of place when it's not relevant and you have to reiterate the host object _(or use `null`)_.

```js
console.log.apply(console, [1, 2, 3])
// <- '1 2 3'
```

The spread operator can be used as _a butter knife_ alternative over using `.apply`. There is no need for a context either. You just append three dots `...` to the array, just like with the rest parameter.

```js
console.log(<mark>...</mark>[1, 2, 3])
// <- '1 2 3'
```

As we'll investigate more in-depth next monday, in the article about iterators in ES6, a nice perk of the spread operator is that it can be used on anything that's an _iterable_. This encompasses even things like the results of `document.querySelectorAll('div')`.

```js
[...document.querySelectorAll('div')]
// <- [<div>, <div>, <div>]
```

Another nice aspect of the _butter knife operator_ is that you can **mix and match** regular arguments with it, and they'll be spread over the function call exactly how you'd expect them to. This, too, can be _very very useful_ when you have a lot of argument rebalancing going on in your ES5 code.

```js
console.log(1, ...[2, 3, 4], 5) // becomes `console.log(1, 2, 3, 4, 5)`
// <- '1 2 3 4 5'
```

Time for a real-world example. I sometimes use the method below in Express applications to allow [`morgan`][1] _(the request logger in Express)_ stream its messages through [`winston`][2], a general purpose multi-transport logger. I remove the trailing line breaks from the `message` because `winston` already takes care of those. I also place some metadata about the currently executing process like the host and the process `pid` into the arguments list, and then I `.apply` everything on the `winston` logging mechanism. If you take a close look at the code, the only line of code that's actually doing anything is the one I've highlighted in yellow, the rest is just playing around with `arguments`.

```js
function createWriteStream (level) {
  return {
    write: function () {
      var bits = Array.prototype.slice.call(arguments)
      var message = bits.shift().replace(/\n+$/, '') // remove trailing breaks
      bits.unshift(message)
      bits.push({ hostname: os.hostname(), pid: process.pid })
      <mark>winston[level].apply(winston, bits)</mark>
    }
  }
}
app.use(morgan(':status :method :url', {
  stream: createWriteStream('debug')
}))
```

We can thoroughly simplify the solution with ES6. First, we can use the rest parameter instead of relying on `arguments`. The rest parameter already gives us a true array, so there's no casting involved either. We can grab the `message` directly as the first parameter, and we can then apply everything on `winston[level]` directly by combining normal arguments with the rest of the `...bits` and pieces. The code below is **in much better shape**, as now every piece of it is actually relevant to what we're trying to accomplish, which is call `winston[level]` with a few _modified arguments_. The piece of code we had earlier, in contrast, spent most time manipulating the arguments, and the focus quickly dissipated into **a battle of wits against JavaScript itself** -- _the method stopped being about the code we were trying to write._

```js
function createWriteStream (level) {
  return {
    write: function (message, <mark>...bits</mark>) {
      winston[level](message.replace(/\n+$/, ''), <mark>...bits</mark>, {
        hostname: os.hostname(), pid: process.pid
      })
    }
  }
}
```

We could further _simplify the method by pulling_ the process metadata out, since that won't change for the lifespan of the process. We could've done that in the ES5 code too, though.

```js
var proc = { hostname: os.hostname(), pid: process.pid }
function createWriteStream (level) {
  return {
    write: function (message, ...bits) {
      winston[level](message.replace(/\n+$/, ''), ...bits, proc)
    }
  }
}
```

Another thing we could do to shorten that piece of code might be to use [an arrow function][3]. In this case however, it **would only complicate matters**. You'd have to shorten `message` to `msg` so that it fits in a single line, and the call to `winston[level]` with the rest and spread operators in there makes it **an incredibly complicated sight** to anyone who *hasn't* spent the last 15 minutes thinking about the method _-- be it a team mate or yourself the week after you wrote this function._

```js
var proc = { hostname: os.hostname(), pid: process.pid }
function createWriteStream (level) {
  return {
    write: (msg, ...bits) => winston[level](msg.replace(/\n+$/, ''), ...bits, proc)
  }
}
```

It would be wiser to just keep our earlier version. While it's _quite self-evident_ in this case that an arrow function only **piles onto the complexity**, in other cases it might not be so. It's up to you to decide, and you need to be able to distinguish between using ES6 features because they genuinely improve your codebase and its maintainability, or **whether you're actually decreasing maintainability** by translating things into ES6 just for the sake of doing so.

Some other useful uses are detailed below. You can obviously use the spread operator when creating a new array, but you can also use [while destructuring][5], in which case it works sort of like `...rest` did, and a use case that's not going to come up often but is still worth mentioning is that you can use spread to pseudo-`.apply` when using the `new` operator as well.

Use Case | ES5                                                          | ES6
---|----------------------------------------------|-----------------------------------------
Concatenation | `[1, 2].concat(more)`                                  | `[1, 2, ...more]`
Push onto list | `list.push.apply(list, [3, 4])`                       | `list.push(...[3, 4])`
Destructuring | `a = list[0], rest = list.slice(1)` | `[a, ...rest] = list`
`new` + `apply` | [`new (Date.bind.apply(Date, [null,2015,8,1]))`][4] | `new Date(...[2015,8,1])`

## Default Operator

The default operator is something we've covered in [the destructuring article][5], but only tangentially. Just like you can use default values during destructuring, you can define a default value for any parameter in a function, as shown below.

```js
function sum (left=1, right=2) {
  return left + right
}
console.log(sum())
// <- 3
console.log(sum(2))
// <- 4
console.log(sum(1, 0))
// <- 1
```

Consider the code that initializes options in [`dragula`][6].

```js
function dragula (options) {
  var o = options || {};
  if (o.moves === void 0) { o.moves = always; }
  if (o.accepts === void 0) { o.accepts = always; }
  if (o.invalid === void 0) { o.invalid = invalidTarget; }
  if (o.containers === void 0) { o.containers = initialContainers || []; }
  if (o.isContainer === void 0) { o.isContainer = never; }
  if (o.copy === void 0) { o.copy = false; }
  if (o.revertOnSpill === void 0) { o.revertOnSpill = false; }
  if (o.removeOnSpill === void 0) { o.removeOnSpill = false; }
  if (o.direction === void 0) { o.direction = 'vertical'; }
  if (o.mirrorContainer === void 0) { o.mirrorContainer = body; }
}
```

> Do you think it would be useful to switch to default parameters under ES6 syntax? How would you do that?

[1]: https://github.com/expressjs/morgan "expressjs/morgan on GitHub"
[2]: https://github.com/winstonjs/winston "winstonjs/winston on GitHub"
[3]: /articles/es6-arrow-functions-in-depth "ES6 Arrow Functions in Depth on Pony Foo"
[4]: http://stackoverflow.com/a/8843181/389745 "This is insanely unreadable. Avoid."
[5]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
[6]: https://github.com/bevacqua/dragula/blob/f5f4c569780b0db160269e978eaf69dc36e421bb/dragula.js#L27-L37 "Option initialization in dragula on GitHub"
