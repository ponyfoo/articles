# `Array.prototype.flatten`

The `.flatten` proposal will take an array and return a new array where the old array was flattened recursively. The following bits of code represent the `Array.prototype.flatten` API.

```js
[1, 2, 3, 4].flatten() // <- [1, 2, 3, 4]
[1, [2, 3], 4].flatten() // <- [1, 2, 3, 4]
[1, [2, [3]], 4].flatten() // <- [1, 2, 3, 4]
```

One could implement a polyfill for `.flatten` thus far like below. I separated the implementation of `flatten` from the polyfill so that you don't necessarily have to use it as a polyfill if you just want to use the method without changing `Array.prototype`.

```js
Array.prototype.flatten = function () {
  return flatten(this)
}
function flatten (list) {
  return list.reduce((a, b) => (Array.isArray(b) ? a.push(...flatten(b)) : a.push(b), a), [])
}
```

Keep in mind that the code above might not be the most efficient approach to array flattening, but it accomplishes recursive array flattening in a few lines of code. Here's how it works.

- A consumer calls `x.flatten()`
- The `x` list is reduced using `.reduce` into a new array `[]` named `a`
- Each item `b` in `x` is evaluated through `Array.isArray`
- Items that aren't an array are pushed to `a`
- Items that are an array are flattened into a new array
  - Those items are [spread over][1] a `.push` call for `a`
- This eventually results in a flat array

The proposal also comes with an optional `depth` parameter _-- that defaults to `Infinity` --_ which can be used to determine how deep the flattening should go.

```js
[1, [2, [3]], 4].flatten() // <- [1, 2, 3, 4]
[1, [2, [3]], 4].flatten(2) // <- [1, 2, 3, 4]
[1, [2, [3]], 4].flatten(1) // <- [1, 2, [3], 4]
[1, [2, [3]], 4].flatten(0) // <- [1, [2, [3]], 4]
```

Adding the `depth` option to our polyfill wouldn't be that hard, we pass it down to recursive `flatten` calls and ensure that, _when the bottom is reached_, we stop flattening and recursion.


```js
Array.prototype.flatten = function (<mark>depth=Infinity</mark>) {
  return flatten(this<mark>, depth</mark>)
}
function flatten (list<mark>, depth</mark>) {
  if (depth === 0) {
    return list
  }
  return list.reduce((accumulator, item) => {
    if (Array.isArray(item)) {
      accumulator.push(...flatten(item, <mark>depth - 1</mark>))
    } else {
      accumulator.push(item)
    }
    return accumulator
  }, [])
}
```

Alternatively _-- for Internet points --_ we could fit the whole of `flatten` in a single expression.

```js
function flatten (list, depth) {
  return depth === 0 ? list : list.reduce((a, b) => (Array.isArray(b) ?
    a.push(...flatten(b, depth - 1)) :
    a.push(b), a), [])
}
```

Then there's `.flatMap`.

# `Array.prototype.flatMap`

This method is convenient because of how often use cases come up where it might be appropriate, and at the same time it provides a small boost in performance, as we'll note next.

Taking into account the polyfill we created earlier for flattening through `Array.prototype.flatten`, the `.flatMap` method can be represented in code like below. Note how you can provide a mapping function `fn` and its `ctx` context as usual, but the flattening is fixed at a depth of `1`.

```js
Array.prototype.flatMap = function (fn, ctx) {
  return this.map(fn, ctx).flatten(1)
}
```

Typically, the code shown above is how you would implement `.flatMap` in user code, but the native `.flatMap` **trades a bit of readability for performance**, by introducing the ability to map items directly in the internal flatten procedure, avoiding the two-pass that's necessary if we first `.map` and then `.flatten` an `Array`.

A possible example of using `.flatMap` can be found below.

```js
[{ x: 1, y: 2 }, { x: 3, y: 4 }, { x: 5, y: 6 }].flatMap(c => [c.x, c.y])
// <- [1, 2, 3, 4, 5, 6]
```

The above is syntactic sugar for doing `.map(c => [c.x, c.y]).flatten()` while providing a small performance boost by avoiding the aforementioned two-pass when first mapping and then flattening.

Note that our previous polyfill **doesn't cover the performance boost**, let's fix that by changing our own internal `flatten` function and adjust `Array.prototype.flatMap` accordingly. We've added a couple more parameters to `flatten`, where we allow the item to be mapped into a different value right before flattening, and avoiding the extra loop over the array.

```js
Array.prototype.flatMap = function (fn, ctx) {
  return <mark>flatten(this, 1, fn, ctx)</mark>
}
function flatten (list, depth<mark>, mapperFn, mapperCtx</mark>) {
  if (depth === 0) {
    return list
  }
  return list.reduce((accumulator, item<mark>, i</mark>) => {
    if (mapperFn) {
      <mark>item = mapperFn.call(mapperCtx || list, item, i, list)</mark>
    }
    if (Array.isArray(item)) {
      accumulator.push(...flatten(item, depth - 1))
    } else {
      accumulator.push(item)
    }
    return accumulator
  }, [])
}
```

> Since the `mapperFn` and `mapperCtx` parameters of `flatten` are entirely optional, we could still use this same internal `flatten` function to polyfill both `.flatten` and `.flatMap`.

[1]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth"
