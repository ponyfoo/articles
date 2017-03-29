Iterators follow a similar pattern _[(you may read our primer on iterators here)][iterators]_. They enforce a contract that dictates we should return an object with a `next` method. That method should return sequence elements following a `{ value, done }` shape. The following example shows a `fibonacci` iterable that's a rough equivalent of the generator we were just looking at.

```js
const fibonacci = {
  [Symbol.iterator]() {
    let previous = 0
    let current = 1
    return {
      next() {
        const value = current
        const next = current + previous
        previous = current
        current = next
        return { value, done: false }
      }
    }
  }
}
const sequence = fibonacci[Symbol.iterator]()
console.log(sequence.next()) // <- { value: 1, done: false }
console.log(sequence.next()) // <- { value: 1, done: false }
console.log(sequence.next()) // <- { value: 2, done: false }
console.log(sequence.next()) // <- { value: 3, done: false }
console.log(sequence.next()) // <- { value: 5, done: false }
console.log(sequence.next()) // <- { value: 8, done: false }
console.log(sequence.next()) // <- { value: 13, done: false }
console.log(sequence.next()) // <- { value: 21, done: false }
```

Let's reiterate. An iterator should return an object with a `next` method: generator functions do just that. The `next` method should return objects with a `{ value, done }` shape: generator functions do that too. What happens if we change the `fibonacci` iterable to use a generator function for its `Symbol.iterator` property? As it turns out, it just works.

The following example shows the iterable `fibonacci` object using a generator function for its iterator. Note how that iterator has the exact same contents as the `fibonacci` generator function we saw earlier. We can use `yield`, `yield*`, and all of the semantics found in generator functions hold.

```js
const fibonacci = {
  * [Symbol.iterator]() {
    let previous = 0
    let current = 1
    while (true) {
      yield current
      const next = current + previous
      previous = current
      current = next
    }
  }
}
const g = fibonacci[Symbol.iterator]()
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 2, done: false }
console.log(g.next()) // <- { value: 3, done: false }
console.log(g.next()) // <- { value: 5, done: false }
console.log(g.next()) // <- { value: 8, done: false }
console.log(g.next()) // <- { value: 13, done: false }
console.log(g.next()) // <- { value: 21, done: false }
```

Meanwhile, the iterable protocol also holds up. To verify that you might use a construct like `for..of`, instead of manually creating the generator object. The following example uses `for..of` and introduces a circuit breaker to prevent an infinite loop from crashing the program.

```js
for (const value of fibonacci) {
  console.log(value)
  if (value > 20) {
    break
  }
}
// <- 1
// <- 1
// <- 2
// <- 3
// <- 5
// <- 8
// <- 13
// <- 21
```

This was a fun trick. What would you use it for in a real-world program?

# Further Reading

- [ES6 Iterators in Depth][iterators]
- [ES6 Generators in Depth][generators]
- [ES6 Symbols in Depth][symbols]
- [Understanding JavaScript's `async`/`await`][asyncawait] *(bonus track!) 🎶*

[iterators]: /articles/es6-iterators-in-depth "ES6 Iterators in Depth on Pony Foo"
[generators]: /articles/es6-generators-in-depth "ES6 Generators in Depth on Pony Foo"
[symbols]: /articles/es6-symbols-in-depth "ES6 Symbols in Depth on Pony Foo"
[asyncawait]: /articles/understanding-javascript-async-await "Understanding JavaScript's async await on Pony Foo"
