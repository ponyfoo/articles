Granted, there's more rules to a `const` declaration than to a `var` declaration: block-scoped, TDZ, assign at declaration, no reassignment. Whereas `var` statements only signal function scoping. Rule-counting, however, doesn't offer a lot of insight. It is better to weigh these rules in terms of complexity: does the rule add or subtract complexity? In the case of `const`, block scoping means a narrower scope than function scoping, TDZ means that we don't need to scan the scope backwards from the declaration in order to spot usage before declaration, and assignment rules mean that the binding will always preserve the same reference.

The more constrained statements are, the simpler a piece of code becomes. As we add constraints to what a statement might mean, code becomes less unpredictable. This is one of the biggest reasons why statically typed programs are generally easier to read than dynamically typed ones. Static typing places a big constraint on the program writer, but it also places a big constraint on how the program can be interpreted, making its code easier to understand.

With these arguments in mind, it is recommended that you use `const` where possible, as it's the statement that gives us the least possibilities to think about.

```js
if (condition) {
  // can't access `isReady` before declaration is reached
  const isReady = true
  // `isReady` binding can't be reassigned
}
// can't access `isReady` outside of its containing block scope
```

When `const` isn't an option, because the variable needs to be reassigned later, we may resort to a `let` statement. Using `let` carries all the benefits of `const`, except that the variable can be reassigned. This may be necessary in order to increment a counter, flip a boolean flag, or to defer initialization.

Consider the following example, where we take a number of megabytes and return a string such as `1.2 GB`. We're using `let`, as the values need to change if a condition is met.

```js
function prettySize (input) {
  let value = input
  let unit = `MB`
  if (value >= 1024) {
    value /= 1024
    unit = `GB`
  }
  if (value >= 1024) {
    value /= 1024
    unit = `TB`
  }
  return `${ value.toFixed(1) } ${ unit }`
}
```

Adding support for petabytes would involve a new `if` branch before the `return` statement.

```js
if (value >= 1024) {
  value /= 1024
  unit = `PB`
}
```

If we were looking to make `prettySize` easier to extend with new units, we could consider implementing a `toLargestUnit` function that computes the `unit` and `value` for any given `input` and its current unit. We could then consume `toLargestUnit` in `prettySize` to return the formatted string.

The following code snippet implements such a function. It relies on a list of supported `units` instead of using a new branch for each unit. When the input `value` is at least `1024` and there's larger units, we divide the input by `1024` and move to the next unit. Then we call `toLargestUnit` with the updated values, which will continue recursively reducing the `value` until it's small enough or we reach the largest unit.

```js
function toLargestUnit (value, unit = `MB`) {
  const units = [`MB`, `GB`, `TB`]
  const i = units.indexOf(unit)
  const nextUnit = units[i + 1]
  if (value >= 1024 && nextUnit) {
    return toLargestUnit(value / 1024, nextUnit)
  }
  return { value, unit }
}
```

Introducing petabyte support used to involve a new `if` branch and repeating logic, but now it's only a matter of adding the `PB` string at the end of the `units` array.

The `prettySize` function becomes concerned only with how to display the string, as it can offload its calculations to the `toLargestUnit` function. This separation of concerns is also instrumental in producing more readable code.

```js
function prettySize (input) {
  const { value, unit } = toLargestUnit(input)
  return `${ value.toFixed(1) } ${ unit }`
}
```

Whenever a piece of code has variables that need to be reassigned, we should spend a few minutes thinking about whether there's a better pattern that could resolve the same problem without reassignment. This is not always possible, but it can be accomplished most of the time.

Once you've arrived at a different solution, compare it to what you used to have. Make sure that code readability has actually improved and that the implementation is still correct. Unit tests can be instrumental in this regard, as they'll ensure you don't run into the same shortcomings twice. If the refactored piece of code seems worse in terms of readability or extensibility, carefully consider going back to the previous solution.

Consider the following contrived example, where we use array concatenation to generate the `result` array. Here, too, we could change from `let` to `const` by making a simple adjustment.

```js
function makeCollection (size) {
  let result = []
  if (size > 0) {
    result = result.concat([1, 2])
  }
  if (size > 1) {
    result = result.concat([3, 4])
  }
  if (size > 2) {
    result = result.concat([5, 6])
  }
  return result
}
makeCollection(0) // <- []
makeCollection(1) // <- [1, 2]
makeCollection(2) // <- [1, 2, 3, 4]
makeCollection(3) // <- [1, 2, 3, 4, 5, 6]
```

We can replace the reassignment operations with `Array#push`, which accepts multiple values. If we had a dynamic list, we could use the spread operator to push as many `...items` as necessary.

```js
function makeCollection (size) {
  const result = []
  if (size > 0) {
    result.push(1, 2)
  }
  if (size > 1) {
    result.push(3, 4)
  }
  if (size > 2) {
    result.push(5, 6)
  }
  return result
}
makeCollection(0) // <- []
makeCollection(1) // <- [1, 2]
makeCollection(2) // <- [1, 2, 3, 4]
makeCollection(3) // <- [1, 2, 3, 4, 5, 6]
```

When you do need to use `Array#concat`, you should probably use `[...result, 1, 2]` instead, to keep it simpler.

The last case we'll cover is one of refactoring. Sometimes, we write code like the next snippet, usually in the context of a larger function.

```js
let completionText = `in progress`
if (completionPercent >= 85) {
  completionText = `almost done`
} else if (completionPercent >= 70) {
  completionText = `reticulating splines`
}
```

In these cases, it makes sense to extract the logic into a pure function. This way we avoid the initialization complexity near the top of the larger function, while clustering all the logic about computing the completion text in one place.

The following piece of code shows how we could extract the completion text logic into its own function. We can then move `getCompletionText` out of the way, making the code more linear in terms of readability.

```js
const completionText = getCompletionText(completionPercent)
// ...
function getCompletionText(progress) {
  if (progress >= 85) {
    return `almost done`
  }
  if (progress >= 70) {
    return `reticulating splines`
  }
  return `in progress`
}
```

What's your stance in `const` vs. `let` vs. `var`?

> _This article was extracted from [Practical ES6][pes6], a book I'm writing. It's openly available online under [HTML][html] format, and on GitHub as [AsciiDoc][asc]. It recently **raised over $12,000 💰** in funding [on Indiegogo][raise] and is available as an [Early Release][er] by the publisher, O'Reilly Media._

[pes6]: /books/practical-es6/chapters#toc "Check out its table of contents!"
[html]: /books/practical-es6/chapters/9#read "Read chapter 9 on Pony Foo"
[asc]: https://github.com/mjavascript/practical-es6 "mjavascript/practical-es6 on GitHub"
[raise]: https://www.indiegogo.com/projects/modular-javascript-a-pragmatic-js-book-series#/ "Modular JavaScript on Indiegogo"
[er]: http://shop.oreilly.com/product/0636920047124.do "Practical ES6 Early Release"
