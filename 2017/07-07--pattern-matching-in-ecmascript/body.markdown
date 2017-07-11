There's a stage 0 proposal for pattern matching in JavaScript. In this article we'll take a look at what the proposal entails and also show how you might find it useful.

The [proposal document][proposal] has a few code examples, as usual. Here is one of them.

```js
let length = vector => match (vector) {
  { x, y, z }: Math.sqrt(x ** 2 + y ** 2 + z ** 2),
  { x, y }: Math.sqrt(x ** 2 + y ** 2),
  [...]: vector.length,
  else: {
    throw new Error(`Unknown vector type`)
  }
}
```

Trying to make sense of that bit of code might prove challenging, given all the unfamiliar code, paired with an arrow function, let assignment, and a boatload of exponentiation operators. Let's reduce it.

# The Basics of ECMAScript Pattern Matching

The following example is a match *expression* which receives a `point` parameter. When `point` has an `x` property and a `y` property, the expression evaluates to `[point.x, point.y]`.

```js
const point = { x: 5, y: 7 }
const result = match (<mark>point</mark>) {
  <mark>{ x, y }</mark>: <mark>[point.x, point.y]</mark>
}
console.log(result) // <- [5, 7]
```

For convenience, we might turn this into a function.

```js
function matchPoint(point) {
  return match (point) {
    { x, y }: [point.x, point.y]
  }
}
```

Or an arrow function. See how terse this is?

```js
const matchPoint = point => match (point) {
  { x, y }: [point.x, point.y]
}
```

We could make it more terse! In the `{ x, y }` pattern above, `x` and `y` are bound to the properties on `point` by the same name, meaning we could write code like the following. Note that `x` and `y` would only be bound in the "match leg" for the `{ x, y }` pattern, meaning that the only place where we can reference those bindings is in the case where that pattern is matched.

```js
const matchPoint = point => match (point) {
  { x, y }: [x, y]
}
```

We could take this a step further, and match on arrays as well. In this case we're matching an array with two elements, and binding them as `x` and `y`. Just for fun, we'll call this one `flipPoint`.

```js
const flipPoint = point => match (point) {
  { x, y }: [x, y],
  [x, y]: { x, y }
}
flipPoint([3, 7]) // { x: 3, y: 7 }
flipPoint({ x: 3, y: 7 }) // [3, 7]
```

Note that if `point` doesn't match any pattern, a runtime error will occur.

```js
matchPoint({ x: 3, z: 7 })
// <- Error
```

Alternatively, we can set up an `else` pattern. This will match when nothing else does.

```js
const matchPoint = point => match (point) {
  { x, y }: [x, y],
  <mark>else</mark>: [0, 0]
}

matchPoint({ x: 3, z: 7 })
// <- [0, 0]
```

Instead of a implicitly returning an expression for the match leg, you can use a block. This is akin to how it works for arrow functions.

```js
const matchPoint = point => match (point) {
  { x, y }: <mark>{</mark>
    return [x, y]
  <mark>}</mark>,
  else: <mark>{</mark>
    throw new Error(`That's not even a point!`)
  <mark>}</mark>
}
```

# More Patterns!

There are literal patterns. This means we can match things like `null`, `undefined`, `true`, `false`, in addition to numbers like `0`, and strings like `'two'`. These don't seem all that useful but they may come in handy depending on your use case for pattern matching.

Object patterns are **inclusive**: the `{ x, y }` pattern will match on an object shaped like `{ x, y ,z }`.

```js
const matchPoint = point => match (point) {
  { x, y }: [x, y]
}
matchPoint({ x: 2, y: 5, z: -1 })
// <- [2, 5]
```

If we still want to get all other properties like we would do while destructuring --- maybe we consider them options --- we can use a similar dot dot dot operator in pattern matching.

```js
const matchPoint = point => match (point) {
  { x, y, ...options }: { point: [x, y], options }
}
matchPoint({ x: 2, y: 5, radius: 50, width: 3 })
// <- { point: [2, 5], options: { radius: 50, width: 3 } }
```

Object patterns allow for nesting. We can match an object that literally has `{ x: 3, y: 4 }`, for example.

```js
const matchNullPoint = point => match (point) {
  { x: 0, y: 0 }: [x, y]
}
matchNullPoint({ x: 0, y: 0 })
// <- [0, 0]
```

The nested pattern could also contain other object matchers.

```js
const isUSD = point => match (item) {
  { options: { currency: 'USD' } }: true,
  else: false
}
isUSD({ value: 19.99, options: { currency: 'USD' } })
// <- true
isUSD({ value: 19.99, options: { currency: 'ARS' } })
// <- false
```

Array pattern matching is a little different in that it is **exclusive** by default: the `[]` pattern only matches empty array-like objects with a `length` property, unlike `{}` which would match any object.

Arrays can be made **inclusive** by adding the `...` pattern. Unlike in rest, spread, or object pattern matching, it's not necessary to name the rest parameter. We could simply do `[...]`, meaning match an array of any length. Or we could do `[first, ...]` to match an array of any length and place the first item on a binding. Doing `[...rest]` places every element in a binding, and so on.

Arrays also support nested pattern matching just like objects did. The following examples matches an array, with a single element (because array patterns are **exclusive** unless we make them **inclusive** by adding `...` to them), that is an object, that has `x` and `y` properties (and maybe some other properties because object patterns are **inclusive**).

```js
const matchPoint = point => match (point) {
  [{ x, y }]: [x, y]
}
matchPoint([{ x: 1, y: 2 }]) // <- [1, 2]
```

# Identifiers and `Symbol.matches`

We can also match with a regular expression. Note that we can only pass the identifier as a valid match pattern --- `numbers` --- and not the regular expression or any expression literal directly. This makes the syntax less complicated while keeping `match` powerful.

```js
const numbers = <mark>/^-?\d+,\s*-?\d+$/</mark>
const matchPoint = point => match (point) {
  { x, y }: [x, y],
  [x, y]: [x, y],
  <mark>numbers</mark>: point.split(/,\s*/).map(n => parseInt(n))
}

matchPoint(`{ x: 7, y: -3 }) // <- [7, -3]
matchPoint([7, -3]) // <- [7, -3]
matchPoint(`7, -3`) // <- [7, -3]
```

Regular expressions as a pattern matcher are made possible thanks to symbols. The proposal includes `Symbol.matches`, which can be used to determine whether the host object matches the received value.

```js
const threeDigitNumber = {
  [Symbol.matches](value) {
    return value >= 100 && value < 1000
  }
}
```

Now we can match using the identifier for `threeDigitNumber` in our match patterns.

```js
const matchPoint = point => match (point) {
  threeDigitNumber: point.toString.split(``).map(n => parseInt(n))
}
matchPoint(735) // <- [7, 3, 5]
```

The proposal is in active development, and a few useful `Symbol.matches` extensions and built-in implementations are being considered at this time.

If something like basic type pattern matching `Symbol.matches` are offered natively, we'd be able to do something akin to type checking in native JavaScript, at least at runtime. This opens up the specification for interesting static type checking possibilities using a similar syntax, though, so the potential is there! 😘

```js
const matchPoint = point => match (point) {
  { x: Number, y: Number }: [x, y]
}
matchPoint({ x: 1, y: 2 }) // <- [1, 2]
matchPoint({ x: 1, z: 2 }) // <- Error
matchPoint({ x: 1, y: 'two' }) // <- Error
```

As always, remember this proposal is at stage 0 and thus highly likely to change or fail to materialize as an official JavaScript language feature. 😅

[proposal]: https://github.com/tc39/proposal-pattern-matching "tc39/proposal-pattern-matching on GitHub"
