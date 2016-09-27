For context, let's start with what we know. You may recall how iterators work using `Symbol.iterator` as an interface to define how an object is to be iterated.

```js
const ponyfoo = {
  [Symbol.iterator]: () => {
    const items = [`p`, `o`, `n`, `y`, `f`, `o`, `o`];
    return {
      next: () => ({
        done: items.length === 0,
        value: items.shift()
      })
    }
  }
}
```

And that the `ponyfoo` object can be iterated in a number of different ways: such as using the spread operator, `Array.from`, or `for..of`, among others.

```js
[...ponyfoo]
// <- [`p`, `o`, `n`, `y`, `f`, `o`, `o`]
Array.from(ponyfoo)
// <- [`p`, `o`, `n`, `y`, `f`, `o`, `o`]

for (const item of ponyfoo) {
  console.log(item)
  // <- `p`
  // <- `o`
  // <- `n`
  // <- `y`
  // <- `f`
  // <- `o`
  // <- `o`
}
```

The contract to an iterator mandates that the `next` method of `Symbol.iterator` instances returns an object with `value` and `done` properties. The `value` property indicates the current value in the sequence, while `done` is a boolean indicating whether the sequence has ended.

In *async iterators*, the contract changes a little bit: `next` is supposed to return a `Promise` that resolves to an object containing `value` and `done` properties. Instead of reusing the same `Symbol`, a new `Symbol.asyncIterator` is introduced to declare asynchronous iterators.

For the purposes of our demonstration, the `ponyfoo` iterable could be made iterable asynchronously with two small changes, we ditch `Symbol.iterator` in favor of `Symbol.asyncIterator`, and we wrap the return value for the `next` method in `Promise.resolve`, returning a `Promise`.

```js
const ponyfoo = {
  <mark>[Symbol.asyncIterator]</mark>: () => {
    const items = [`p`, `o`, `n`, `y`, `f`, `o`, `o`];
    return {
      next: () => <mark>Promise.resolve</mark>({
        done: items.length === 0,
        value: items.shift()
      })
    }
  }
}
```

Naturally, that was quite a contrived example. Another contrived example could be a utility function that fetches a series of HTTP resources sequentially.

```js
const getResources = endpoints => ({
  [Symbol.asyncIterator]: () => ({
    next: () => {
      if (endpoints.length === 0) {
        return Promise.resolve({ done: true })
      }
      return fetch(endpoints.shift())
        .then(response => response.json())
        .then(value => ({ value, done: false }))
    }
  })
})
```

In order to consume an async iterator, we can leverage the `for await..of` syntax that would also be introduced by this proposal. This way, there's yet another way of writing code that looks synchronous *yet behaves asynchronously.*

```js
const resources = [
  `/api/users`,
  `/api/testers`,
  `/api/hackers`,
  `/api/nsa-backdoor`
];

for await (const data of getResources(resources)) {
  console.log(data);
}
```

There's also async generator functions in this proposal. An async generator function is just like a generator function, but also supports `await` and `for await..of` declarations.

```js
async function* getResources(endpoints) {
  while (endpoints.length > 0) {
    const endpoint = endpoints.shift()
    const response = await fetch(endpoint)
    yield await response.json()
  }
}
```

When called, async generators return an `{ next, return, throw }` object whose methods return promises for `{ next, done }`, instead of returning `{ next, done }` directly.

You can consume the `getResources` async generator in exactly the same way you could consume the object-oriented async iterator.
